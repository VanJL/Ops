# Import Resources





## YUM 源服务器配置

在无法连接公网的环境中，导入资源的前提是需要在 YUM 源服务器上配置好内网的离线 YUM 源，将内网 YUM 源作为 Foreman 的上游来同步资源到 Foreman 服务器中。



配置基础YUM源





配置 errata









## 导入 YUM 源







## 导入 errata

通过使用命令生成 updateinfo

```bash

```



创建空的 repodate 后导入 updateinfo

```bash

```





## 附录

### 附录1 获取官方errata

**CentOS Errata**

可以参考这个网站提供的（这个网站应该是个个人开源项目，具体的运作方式后面我再仔细了解一下）：https://cefs.steve-meier.de/ 

最新的 centos errata 下载地址：https://cefs.steve-meier.de/errata.latest.xml.bz2



**RHEL Errata**

红帽官方的可以在官方提供的资源中获取：https://www.redhat.com/security/data/oval/v2/

> 不同的版本所需的errata，可以在官网中自行选择。



## References

- Importing CentOS Errata with Pulp3：https://community.theforeman.org/t/importing-centos-errata-with-pulp3/23269
- generate_updateinfo：https://github.com/vmfarms/generate_updateinfo
- Errata import into Katello for CentOS 7：https://community.theforeman.org/t/errata-import-into-katello-for-centos-7/21658