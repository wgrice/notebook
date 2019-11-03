### 一、使用VMWare创建虚拟机

1. 选择CentOS的镜像文件
2. 配置虚拟机的参数（选择桥接模式、勾上复制物理网络连接状态）

### 二、配置CentOS

1. 软件选择（SOFTWARE SELECTION）

   要指定软件包将被安装，选择软件时选择安装摘要屏幕。包组分为基础环境。这些环境是预先定义的一组具有特定用途的软件包；例如，在虚拟化主机环境中包含的一组所需的系统上运行的虚拟机软件程序包。只有一个软件环境可以在安装时选择。

   对于每一个环境，也有附加组件的形式提供额外的软件包。附加组件列于屏幕的右侧部分，当选择一个新的环境，它们的列表被刷新。您可以选择多个附加组件的安装环境。选择“带GUI的服务器（Server with GUI）”，然后按“完成”键。

   **最小安装（Minimal Install）**

   这个选项只提供运行CentOS 的基本软件包。最小安装为单一目的服务器提供基本需要，并可在这样的安装中最大化性能和安全性。

   **基础设施服务器（Infrastructure Server）**

   这个选项提供在服务器中使用的CentOS 基本安装，不包含桌面。

   **文件及打印服务器（File and Print Server）**

   用于企业的文件、打印及存储服务器。

   **基本网页服务器（Basic Web Server）**

   基本系统平台，加上PHP，Web server，还有MySQL和PostgreSQL数据库的客户端，无桌面。

   **虚拟化主机（Virtualization Host）**

   这个选项提供 KVM 和 Virtual Machine Manager 工具以创建用于虚拟机器的主机。

   **带GUI的服务器（Server with GUI）**

   带有用于操作网络基础设施服务GUI的服务器。

   **GNOME桌面（GNOME Desktop）**

   GNOME是一个非常直观且用户友好的桌面环境。

   **KDE Pasma Workspaces（KDE桌面）**

   一个高度可配置图形用户界面，其中包括面板、桌面、系统图标以及桌面向导和很多功能强大的KDE应用程序。

2. 安装位置（INSTALLTION DESTINATION）

   分区前先规划好

   swap #交换分区

   / #剩余所有空间

   备注：生产服务器建议单独再划分一个data分区存放数据

   默认创建：swap[交换分区] /[根分区] /home[数据分区]

### 三、配置网络

1. 进入网络配置文件目录

   `cd  /etc/sysconfig/network-scripts/ ` 并 `ls`

   ![1560671614782](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1560671614782.png)

2. 查看当前网卡名称

   `ifconfig` 或 `ip addr`

   ![1560671658667](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1560671658667.png)

3. 编辑对应网卡的配置文件

   `vi ifcfg-ens33`

   ![1560671725010](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1560671725010.png)

4. 重启网络

   `service network restart `

5. 测试网络连接是否正常

   `ping www.baidu.com `

### 三、网卡名称修改为`ifcfg-eth0`

1. 进入目录 `cd /etc/sysconfig/network-scripts/`
2. 修改文件名称 `mv ifcfg-ens33 ifcfg-eth0`
3. 编辑文件 `vi ifcfg-eth0`
4. 修改文件中的配置项 `NAME=eth0 ` `DEVICE=eth0`

6. 编辑grub `vi /etc/sysconfig/grub`

7. 在 `GRUB_CMDLINE_LINUX` 变量中添加一句 `net.ifnames=0 biosdevname=0`

8. 重新生成grub配置并更新内核参数 `grub2-mkconfig -o /boot/grub2/grub.cfg`

9. 添加udev的规则，在 `/etc/udev/rules.d` 目录中创建一个网卡规则 `70-persistent-net.rules` ，并写入下面的语句:

   ```
   SUBSYSTEM=="net",ACTION=="add",DRIVERS=="?*",ATTR{address}=="00:0c:29:dc:dd:ad",ATTR｛type｝=="1" ,KERNEL=="eth*",NAME="eth0"
   #ATTR{address}=="00:0c:29:dc:dd:ad"是网卡的MAC地址
   ```

10. 重启系统 `shutdown -r now`

### 四、配置完了虚拟机还是无法联网 - 配置VMWare

1. 菜单：编辑---->虚拟网络编辑器

   设置VMnet0的桥接模式的外部链接为如图所示

   ![1560675129625](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1560675129625.png)

