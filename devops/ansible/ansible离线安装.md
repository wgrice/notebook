系统：centos7
 服务器：一台能联网的、一台离线的。系统版本相同即可
 工具：yumdownloader

## 在能联网的服务器上

#### 1、安装yumdownloader

yumdownloader是什么：
 yumdownloader is a program for downloading RPMs from Yum repositories

安装：
 `yum install yum-utils -y`

#### 2、获取ansible安装包及依赖

```
mkdir /tmp/ansible
yumdownloader --resolve --destdir /tmp/ansible ansible
yumdownloader --resolve --destdir /tmp/ansible createrepo
tar zcf ansible.tar.gz /tmp/ansible
tar -czvPf ansible.tar.gz /tmp/ansible
```

注意/tmp/ansible ansible之间是有空格的哦

#### 3、上传

将ansible.tar.gz上传到离线服务器上/tmp目录下

## 在离线服务器上

#### 1、解压压缩包

```
tar zxf /tmp/ansible.tar.gz
```

#### 2、制作离线源

```
cd /tmp/ansible
rpm -ivh deltarpm-3.6-3.el7.x86_64.rpm
rpm -ivh python-deltarpm-3.6-3.el7.x86_64.rpm
rpm -ivh createrepo-0.9.9-28.el7.noarch.rpm
cd /tmp
createrepo ansible
```

#### 3、编辑yum文件

```
vim /etc/yum.repos.d/ansible.repo
[ansible]
name=ansible
baseurl=file:///tmp/ansible
gpgcheck=0
enabled=1
```

#### 4、安装ansible

```shell
yum install ansible -y
```

#### 5、配置ssh

服务端:192.168.0.64 客户端:192.168.0.65
一键生成非交互式秘钥对

```shell
ssh-keygen -t rsa -f /root/.ssh/id_rsa
```
然后把公钥(id_rsa.pub)拷贝到客户端上：
```shell
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.0.65
```
本机也要拷贝：
```shell
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys　　　　　　# 必须是600, 否则用ansible连接本机报错
```
在服务端测试ssh是否可以登录

#### 6、配置主机组

如果没有ansible目录创建即可

```shell
mkdir -p /etc/ansible/
touch /etc/ansible/hosts
cat > /etc/ansible/hosts << EOF
[k8s]
192.168.0.91
192.168.0.92
192.168.0.93
192.168.0.94
[test1]
192.168.0.91
[test2]
192.168.0.92
[test3]
192.168.0.93
[test4]
192.168.0.94
EOF
```

#### 7、创建、配置ansible配置文件

```shell
touch /etc/ansible/ansible.cfg

cat > /etc/ansible/ansible.cfg << EOF 
[defaults]
inventory = /etc/ansible/hosts
sudo_user=root
remote_port=22
host_key_checking=False
remote_user=root
log_path=/var/log/ansible.log
module_name=command
private_key_file=/root/.ssh/id_rsa

#关闭报错信息显示
deprecation_warnings=False

pipelining = True

#不收集系统变量
gather_facts: no

#开启时间显示
callback_whitelist = profile_tasks

#关闭秘钥检测
host_key_cheking=False
EOF
```

