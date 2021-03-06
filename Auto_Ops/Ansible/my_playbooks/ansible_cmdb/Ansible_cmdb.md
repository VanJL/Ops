# Ansible CMDB



## Author

```tex
Name: Shinefire
Blog: https://github.com/shine-fire/Ops_Notes
E-mail: shine_fire@outlook.com
```



## Introduction

### Features

(Not all features are supported by all templates)

- **Multiple formats / templates:**
  - Fancy HTML (`--template html_fancy`), as seen in the screenshots above.
  - Fancy HTML Split (`--template html_fancy_split`), with each host's details in a separate file (for large number of hosts).
  - CSV (`--template csv`), the trustworthy and flexible comma-separated format.
  - JSON (`--template json`), a dump of all facts in JSON format.
  - Markdown (`--template markdown`), useful for copy-pasting into Wiki's and such.
  - Markdown Split (`--template markdown_split`), with each host's details in a seperate file (for large number of hosts).
  - SQL (`--template sql`), for importing host facts into a (My)SQL database.
  - Plain Text table (`--template txt_table`), for the console gurus.
  - and of course, any custom template you're willing to make.
- Host overview and detailed host information.
- Host and group variables.
- Gathered host facts and manual custom facts.
- Adding and extending facts of existing hosts and manually adding entirely new hosts.
- Custom columns







## Install ansible_cmdb

使用pip install可以直接安装

```bash
# pip install ansible-cmdb
```

## Use ansible_cmdb

1. 创建playbook目录

   ```bash
   # mkdir -p /playbook/ansible_cmdb
   ```

2. 编写inventory

   ```bash
   # cat /playbook/ansible_cmdb/inventory 
   [test]
   local ansible_connection=local
   ```

3. 编写ansible.cfg

   ```bash
   # cat /playbook/ansible_cmdb/ansible.cfg 
   [defaults]
   fact_caching = jsonfile
   fact_caching_connection = /tmp/ansible-cmdb
   fact_caching_timeout = 600
   forks = 50
   host_key_checking = false
   ```

4. 编写site.yml

   ```yaml
   # cat /playbook/ansible_cmdb/site.yml 
   ---
   - hosts: all
     gather_facts: true
     
     tasks: 
     - name: Generate the cmdb html for all group 
       local_action:
         module: shell
         _raw_params: 'ansible-cmdb -t html_fancy_split -f /tmp/ansible-cmdb/'
         chdir: /var/www/html/
       run_once: yes
   ```

5. 执行playbook

   ```bash
   # cd /playbook/ansible_cmdb
   # ansible-playbook -i inventory site.yml 
   ```

6. 浏览器进入web端查看

   http://IP/cmdb/

7. 总结

   ansible_cmdb的主要作用是通过ansible先对目标机器进行系统数据信息收集，再通过转化格式的形式来展示数据，gather_facts: true这一步即是完成数据收集，后续再通过ansible-cmdb命令来完成形式的转化。

   一般采用`html_fancy_split`模板，可以直接在web端展示很多服务器信息，其他人只要在自己的浏览器中输入URL也可以方便的查看CMDB信息。

## Other

官方文档：https://ansible-cmdb.readthedocs.io/en/latest/usage/

github：https://github.com/fboender/ansible-cmdb