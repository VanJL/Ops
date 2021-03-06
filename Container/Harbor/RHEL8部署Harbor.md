# RHEL8中部署Harbor

> 本章为在RHEL8操作系统上部署harbor的操作文档



## Introduction

### online vs. offline

offline是离线版本，下载的时候就会下载很多镜像，更适合不能联网的环境使用。



## Installation Prerequisites

### Hardware

The following table lists the minimum and recommended hardware configurations for deploying Harbor.

| Resource | Minimum | Recommended |
| :------- | :------ | :---------- |
| CPU      | 2 CPU   | 4 CPU       |
| Mem      | 4 GB    | 8 GB        |
| Disk     | 40 GB   | 160 GB      |



### Software

The following table lists the software versions that must be installed on the target host.

| Software       | Version                       | Description                                                  |
| :------------- | :---------------------------- | :----------------------------------------------------------- |
| Docker engine  | Version 17.06.0-ce+ or higher | For installation instructions, see [Docker Engine documentation](https://docs.docker.com/engine/installation/) |
| Docker Compose | Version 1.18.0 or higher      | For installation instructions, see [Docker Compose documentation](https://docs.docker.com/compose/install/) |
| Openssl        | Latest is preferred           | Used to generate certificate and keys for Harbor             |



### Network ports

Harbor requires that the following ports be open on the target host.

| Port | Protocol | Description                                                  |
| :--- | :------- | :----------------------------------------------------------- |
| 443  | HTTPS    | Harbor portal and core API accept HTTPS requests on this port. You can change this port in the configuration file. |
| 4443 | HTTPS    | Connections to the Docker Content Trust service for Harbor. Only required if Notary is enabled. You can change this port in the configuration file. |
| 80   | HTTP     | Harbor portal and core API accept HTTP requests on this port. You can change this port in the configuration file. |



## Environment

### Software

| Name           | Version |
| -------------- | ------- |
| OS             | RHEL8.3 |
| Docker-CE      | 20.10.8 |
| Docker Compose | 1.29.2  |
| Harbor         | 2.3.2   |



### Hardware

| Resource | Vars |
| -------- | ---- |
| CPU      | 2    |
| Mem      | 2G   |
| Storage  | 100G |





## Deploy Harbor

### 系统环境配置

#### 补丁更新

新安装的系统，先把所有的补丁打一下

```bash
~]# yum update -y
```



#### Hostname配置

修改基础节点主机名：

```bash
~]# hostnamectl set-hostname registry.ocp4.shinefire.com
```



### 安装Docker-ce

下面为使用阿里云提供的yum源安装Docker-ce的方案

```bash
yum install -y yum-utils
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache
yum -y install docker-ce
systemctl enable docker --now
```



### 安装docker-compose

直接在GitHub上获取最新版本的compose即可

```bash
~]# curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
~]# chmod +x /usr/local/bin/docker-compose
```

说明：可以直接在 `https://github.com/docker/compose/releases` 界面查看当前最新的稳定版本，网络环境有限的话可以先离线下载好文件再导入到服务器中。



### 创建自签名证书

证书创建部分都是使用`example.com`来进行的，在不同环境的不同需求中需要替换掉这个域名。

证书均在harbor服务中来进行操作，自己创建CA证书，自己给自己签发证书供harbor使用。



#### Generate a Certificate Authority Certificate

创建一个临时目录用于保存创建证书过程中生成的文件

```bash
~]# mkdir /tmp/certs/
~]# cd /tmp/certs/
```

生成CA证书私钥

```bash
~]# openssl genrsa -out myrootCA.key 4096
```

生成CA certificate

```bash
~]# openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=GD/L=GZ/O=example/OU=Personal/CN=*.ocp4shinefire.com" \
 -key myrootCA.key \
 -out myrootCA.crt
```

#### Generate a Server Certificate

创建给`harbor`签发的证书

生成私钥

```bash
~]# openssl genrsa -out registry.ocp4.shinefire.com.key 4096
```

Generate a certificate signing request (CSR)，生成证书签名请求

```bash
~]# openssl req -sha512 -new \
 -subj "/C=CN/ST=GD/L=GZ/O=example/OU=Personal/CN=registry.ocp4.shinefire.com" \
 -key registry.ocp4.shinefire.com.key \
 -out registry.ocp4.shinefire.com.csr
```

Generate an x509 v3 extension file

```bash
~]# cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=registry.ocp4.shinefire.com
DNS.2=registry
DNS.3=harbor-server2.shinefire.com
DNS.4=harbor-server2
DNS.5=harbor-server.shinefire.com
DNS.6=harbor-server
EOF
```

Use the `v3.ext` file to generate a certificate for your Harbor host.

Replace the `registry.ocp4.shinefire.com` in the CRS and CRT file names with the Harbor host name.

```bash
~]# openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA myrootCA.crt -CAkey myrootCA.key -CAcreateserial \
    -in registry.ocp4.shinefire.com.csr \
    -out registry.ocp4.shinefire.com.crt
```

#### Provide the Certificates to Harbor and Docker

After generating the `ca.crt`, `registry.ocp4.shinefire.com.crt`, and `registry.ocp4.shinefire.com.key` files, you must provide them to Harbor and to Docker, and reconfigure Harbor to use them.

Copy the server certificate and key into the certficates folder

```bash
~]# mkdir -p /data/cert
~]# cp registry.ocp4.shinefire.com.crt /data/cert
~]# cp registry.ocp4.shinefire.com.key /data/cert
```

Convert `registry.ocp4.shinefire.com.crt` to `registry.ocp4.shinefire.com.cert`, for use by Docker.

The Docker daemon interprets `.crt` files as CA certificates and `.cert` files as client certificates.

```bash
~]# openssl x509 -inform PEM -in registry.ocp4.shinefire.com.crt -out registry.ocp4.shinefire.com.cert
```

Copy the server certificate, key and CA files into the Docker certificates folder on the Harbor host. You must create the appropriate folders first.

```bash
~]# mkdir -p /etc/docker/certs.d/registry.ocp4.shinefire.com/
~]# cp registry.ocp4.shinefire.com.cert /etc/docker/certs.d/registry.ocp4.shinefire.com/
~]# cp registry.ocp4.shinefire.com.key /etc/docker/certs.d/registry.ocp4.shinefire.com/
~]# cp myrootCA.crt /etc/docker/certs.d/registry.ocp4.shinefire.com/
```

放在ca_download目录下

```bash
~]# mkdir /data/ca_download
~]# cp myrootCA.crt /data/ca_download/
```

#### trust the certificate

证书创建完毕后，将CA证书放到系统的指定路径下，做证书信任

```bash
~]# cp myrootCA.crt /etc/pki/ca-trust/source/anchors/myrootCA.crt
~]# update-ca-trust
```

#### Restart Docker Engine

```bash
~]# systemctl restart docker
```



### 安装harbor

#### 下载离线安装包

下载 Harbor 离线安装包并解压

```bash
~]# wget https://github.com/goharbor/harbor/releases/download/v2.3.2/harbor-offline-installer-v2.3.2.tgz
~]# tar -xvf harbor-offline-installer-v2.3.2.tgz -C /opt/
```

> 确认当前版本是否有更新的，可以去官网看看，然后把url和文件名称替换一下就行了

#### 修改配置文件

从模板复制一份配置文件

```bash
~]# cd /opt/harbor/
~]# cp harbor.yml.tmpl harbor.yml
```

打开 `/opt/harbor/harbor.yml` 文件，修改 hostname 域名、https 证书等配置信息，具体如下：

```yaml
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: registry.ocp4.shinefire.com
external_url: https://registry.ocp4.shinefire.com
  
# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /etc/docker/certs.d/registry.ocp4.shinefire.com/registry.ocp4.shinefire.com.cert
  private_key: /etc/docker/certs.d/registry.ocp4.shinefire.com/registry.ocp4.shinefire.com.key

...(略)
```

> 我这里把所有需要的基础服务都部署在同一个节点上，后面安装 OCP 时需要用到负载均衡器的 `443` 和 `80` 端口，所以这里 Harbor 使用了非标准端口。如果你资源充足，可以将 Harbor 部署在不同的节点上。



接着执行下面的命令进行安装：

```bash
[root@registry harbor]# ./install.sh --with-notary --with-trivy --with-chartmuseum
```

检查安装完后的结果，刚开始的时候可能会是starting启动的状态，等待都成功healthy状态

```bash
]# docker-compose ps
      Name                     Command                  State                                                               Ports
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
chartmuseum         ./docker-entrypoint.sh           Up (healthy)
harbor-core         /harbor/entrypoint.sh            Up (healthy)
harbor-db           /docker-entrypoint.sh 96 13      Up (healthy)
harbor-jobservice   /harbor/entrypoint.sh            Up (healthy)
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up (healthy)   127.0.0.1:1514->10514/tcp
harbor-portal       nginx -g daemon off;             Up (healthy)
nginx               nginx -g daemon off;             Up (healthy)   0.0.0.0:4443->4443/tcp,:::4443->4443/tcp, 0.0.0.0:80->8080/tcp,:::80->8080/tcp, 0.0.0.0:443->8443/tcp,:::443->8443/tcp
notary-server       /bin/sh -c migrate-patch - ...   Up
notary-signer       /bin/sh -c migrate-patch - ...   Up
redis               redis-server /etc/redis.conf     Up (healthy)
registry            /home/harbor/entrypoint.sh       Up (healthy)
registryctl         /home/harbor/start.sh            Up (healthy)
trivy-adapter       /home/scanner/entrypoint.sh      Up (healthy)
```



### 登录测试

登录测试由另外的一台client机器来做

将ca证书传给 client 机器后，在 client 中执行命令：

```bash
~]# mv myrootCA.crt /etc/pki/ca-trust/source/anchors/
~]# update-ca-trust
```

现在可以通过 `podman login` 命令来测试仓库的连通性，看到如下字样即表示登录成功

```bash
~]# podman login -u admin https://registry.ocp4.shinefire.com
Password:
Login Succeeded!
```

推送测试，直接将当前的 image 推送一个到仓库中测试是否可以正常推送

```bash
~]# podman images
REPOSITORY                       TAG     IMAGE ID      CREATED      SIZE
localhost/goharbor/nginx-photon  v2.3.2  83bd32904c30  6 weeks ago  46 MB
~]# podman tag localhost/goharbor/nginx-photon:v2.3.2 bastion.shinefire.com/library/goharbor/nginx-photon
~]# podman push registry.ocp4.shinefire.com/library/goharbor/nginx-photon
Getting image source signatures
Copying blob f6e68d4c9b22 done
Copying blob 7301dee185fe done
Copying config 83bd32904c done
Writing manifest to image destination
Storing signatures
```



## References

- [Harbor Installation and Configuration](https://goharbor.io/docs/2.3.0/install-config/)



## Doc Changelogs

- 2021.10.05 增加了额外的机器来测试Harbor仓库登录，之前的登录测试是在Harbor自己的机器上测试的

- 2021.09.25 初版

  

