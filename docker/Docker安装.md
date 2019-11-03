### Docker安装

docker官方说至少3.8以上，建议3.10以上（ubuntu下要linux内核3.8以上， RHEL/Centos 的内核修补过， centos6.5的版本就可以。

1. root账户登录，查看内核版本如下

   ```shell
   [root@localhost ~]# uname -a
   Linux localhost.localdomain 3.10.0-862.el7.x86_64 #1 SMP Fri Apr 20 16:44:24 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
   ```

2. 把yum包更新到最新

   ```shell
   [root@localhost ~]# yum update
   ```

3. 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

   ```shell
   [root@localhost ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

4. 设置yum源

   ```shell
   [root@localhost ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```

5. 可以查看所有仓库中所有docker版本，并选择特定版本安装

   ```shell
   [root@localhost ~]# yum list docker-ce --showduplicates | sort -r
   已加载插件：fastestmirror, langpacks
   可安装的软件包
    * updates: centos.ustc.edu.cn
   Loading mirror speeds from cached hostfile
    * extras: mirrors.aliyun.com
   docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
   docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
   docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
   docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
   docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
   docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
   ...
   ```

6. 安装Docker，命令：yum install docker-ce-版本号，我选的是17.12.1.ce，如下

   ```shell
   [root@localhost ~]# yum install docker-ce-17.12.1.ce
   或
   [root@localhost ~]# yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
   ```

7. 启动Docker，命令：systemctl start docker，然后加入开机启动，如下

   ```shell
   [root@localhost ~]# systemctl start docker
   [root@localhost ~]# systemctl enable docker
   Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
   ```

8. 验证安装是否成功

   ```shell
   [root@localhost ~]# docker version
   Client:
    Version:           18.09.6
    API version:       1.39
    Go version:        go1.10.8
    Git commit:        481bc77156
    Built:             Sat May  4 02:34:58 2019
    OS/Arch:           linux/amd64
    Experimental:      false
   
   Server: Docker Engine - Community
    Engine:
     Version:          18.09.6
     API version:      1.39 (minimum version 1.12)
     Go version:       go1.10.8
     Git commit:       481bc77
     Built:            Sat May  4 02:02:43 2019
     OS/Arch:          linux/amd64
     Experimental:     false
   ```

