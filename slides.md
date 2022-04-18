---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# Ansible 介绍

BBFE-武汉 李聪



---

# Ansible 是什么?

Ansible 是一个 python 语言编写的 IT 自动化工具.
-  **Agentless** - Ansible 管理目标机器，并不需要 agent 部署再目标机器上
-  **Python** - Ansible 管理服务器，只需要目标机器上安装 python 2.4+ 
-  **SSH** - Ansible 和目标机器之间使用 SSH 协议通讯
-  **Fast** - Ansible 可以同时管理成百上千台服务器
-  **Scalable** - Ansible 是插件化的，编写插件也非常简单
<br>
<br>

Read more about [Ansible?](https://www.ansible.com/)


---

# 安装
Ansible 是 Python 编写的一个包，我们可以使用 pip 安装 Ansible.
> Ansible 能管理 windows , 但是 Ansible本身 不能运行再 Windows 上，Ansible 能运行再 Windows 的 WSL 里面。
```
pip install ansible
```
```
vagrant@archlinux ~ $ ansible --version
ansible [core 2.12.4]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.10/site-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.4 (main, Mar 23 2022, 23:05:40) [GCC 11.2.0]
  jinja version = 3.0.3
  libyaml = True
```
安装成功以后，系统中会加入一系列 ansible 开头的命令 ansible-* 。
---

# Ansible 命令基本用法
ansible <主机>   <模块>   -a   <模块参数>
```
vagrant@archlinux ~ $ ansible localhost -m shell -a "date"
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED | rc=0 >>
Mon Apr 18 08:19:47 AM UTC 2022
vagrant@archlinux ~ $ ansible localhost -m shell -a "pwd"
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED | rc=0 >>
/home/vagrant
```
Ansible 的默认模块是 command 模块
```
vagrant@archlinux ~ $ ansible localhost -a "pwd"
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED | rc=0 >>
/home/vagrant
```

---

# 名词解释
Ansible 不仅 ansible 命令那么简单，要使用好 Ansible , 还需要了解一些概念。
|     |     |
| --- | --- |
| Inventory | 记录被管理主机的文件 |
| Taks      | Ansible 执行的任务  |
| Playbooks | Ansible 脚本，Yaml 格式或者 JSON 格式,包含一系列的 Task |
| Roles | 一系列 playbook 的组合 |
| Ansible-galaxy | Ansible Role 的管理工具，可以共享，下载 Roles |

---

# Inventory
Inventory 是一个类似 ini 格式的文件，它可以对服务器进行分组
```
mail.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```
更加完整的文件如下, 跟多参数可以参考 [Ansible Doc](https://docs.ansible.com/ansible/2.3/intro_inventory.html#list-of-behavioral-inventory-parameters)
```
mail.example.com ansible_ssh_port=22 ansible_ssh_host=mail.example.com ansible_ssh_user=root ansible_ssh_private_key_file=password

[dbservers]
one.example.com ansible_ssh_port=22 ansible_ssh_host=mail.example.com ansible_ssh_user=root ansible_ssh_private_key_file=password
two.example.com ansible_ssh_port=22 ansible_ssh_host=mail.example.com ansible_ssh_user=root ansible_ssh_private_key_file=password
three.example.com ansible_ssh_port=22 ansible_ssh_host=mail.example.com ansible_ssh_user=root ansible_ssh_private_key_file=password
```

> 如果需要使用账号密码进行 SSH 认证，需要控制机器上安装 sshpass 命令。

---

# Playbook

Ansible 的 playbook 是一个 Yaml 或者 JSON 语言编写的脚本。大致如下格式:
```yaml
---
- hosts: webservers
  vars:
    http_port: 80
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: name=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running (and enable it at boot)
    service: name=httpd state=started enabled=yes
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```
执行
```
ansible-playbook -i inventory palybook.yaml
```
---

# 常用参数

1. `-vvv` 打印 debug 级别的日志，便于排查问题
```
ansible-playbook -i inventory -playbook.yml -vvv
```
2. `--tags` 仅仅执行符合 tag 的部分任务
```
ansible-playbook -i inventory -playbook.yml -tags file
```
3. `--check` 仅仅测试脚本语法和网络，不对目标执行任何变更
```
ansible-playbook -i inventory -playbook.yml --check
```
---

# 常用模块

|     |     |
| --- | --- |
| copy | 复制文件(夹)到目标机器 |
| file | 操作目标机器的文件(夹)创建，删除,权限设置 |
| shell | 在目标机器上执行命令 |
| service | 管理目标机器上的服务，启动，重启，关闭 |
| user | 管理目标机器上的用，创建，删除，组管理 |
| yum | 通过 yum 管理目标机器上包, 更新，安装，卸载 |
| template | 渲染模板文件到目标机器,长用于配置文件管理 |
| lineinfile | 控制目标机器的文本内容，类似 awk ,sed |

更多配置模块，参见[Ansible.Builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)
---

# Ansible Role

---

# 演示点啥

1. 上传文件到服务器
2. 修改配置文件
3. 安装软件
4. 关闭服务

---

# 插件开发

---

# Learn More

[Documentations](https://docs.ansible.com/ansible/latest/index.html) · [GitHub](https://github.com/ansible/ansible) · [Slides](https://github.com/hellojukay/ansible-slides)
