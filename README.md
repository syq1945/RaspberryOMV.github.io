
# 树莓派换源
更换清华源： 

## 来自清华源官网的介绍
**Raspbian 简介**<br>
Raspbian 是专门用于 ARM 卡片式计算机 Raspberry Pi® “树莓派”的操作系统， 其基于 Debian 开发，针对 Raspberry Pi 硬件优化。

Raspbian 并非由树莓派的开发与维护机构 The Raspberry Pi Foundation“树莓派基金会”官方支持。其维护者是一群 Raspberry Pi 硬件和 Debian 项目的爱好者。

注：Raspbian 系统由于从诞生开始就基于（为了 armhf，也必须基于）当时还是 testing 版本的 7.0/wheezy，所以 Raspbian 不倾向于使用 stable/testing 表示版本。

**使用说明**<br>
首先通过 `sudo uname -m` 确定你使用的系统的架构。

编辑镜像站后，请使用sudo apt-get update命令，更新软件源列表，同时检查您的编辑是否正确。

**armv7l**<br>
选择你的 Raspbian 对应的 Debian 版本
Debian 11 (bullseye)
启用源码源
启用 multi-arch aarch64
``` Linux
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ bullseye main non-free contrib rpi
# deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ bullseye main non-free contrib rpi

# deb [arch=arm64] https://mirrors.tuna.tsinghua.edu.cn/raspbian/multiarch/ bullseye main
```
注意：网址末尾的 raspbian 重复两次是必须的。因为 Raspbian 的仓库中除了 APT 软件源还包含其他代码。APT 软件源不在仓库的根目录，而在raspbian/子目录下。

**aarch64**<br>
[aarch64 用户可直接参考 Debian 帮助](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)

**raspberry 镜像**<br>
对于两个架构，编辑 `sudo nano /etc/apt/sources.list.d/raspi.list `文件，[这需要查看 Raspberrypi 帮助。](https://mirrors.tuna.tsinghua.edu.cn/help/raspberrypi/)


## 适合我的树莓派的设置

[清华大学开源软件镜像站_Debian 软件源](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)

一般情况下，将`sudo nano /etc/apt/sources.list` 文件中 Debian 默认的源地址 http://deb.debian.org/ 替换为镜像地址即可。
Debian Buster 以上版本默认支持 HTTPS 源。如果遇到无法拉取 HTTPS 源的情况，请先使用 HTTP 源并安装：
`sudo apt install apt-transport-https ca-certificates`
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E77FC0EC34276B4B

``` linux   
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
```
[Raspberrypi 软件仓库](https://mirrors.tuna.tsinghua.edu.cn/help/raspberrypi/)<br>
编辑 `sudo nano /etc/apt/sources.list.d/raspi.list` 文件。<br>
选择你的 Raspbian 对应的 Debian 版本  Debian 12 (bookworm)<br>
`deb https://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ bookworm main`

**如果 sudo apt-get update 报错：**<br>
W: GPG error: https://mirrors.tuna.tsinghua.edu.cn/debian bookworm InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY **0E98404D386FA1D9** NO_PUBKEY 6ED0E7B82643E131 NO_PUBKEY F8D2585B8783D481<br>

解决方案为： （把代码中keys 后面的数字变更成上面错误的PUBKEY）<br>
`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9 `

更新： 
``` linux
#更新
sudo apt-get update
sudo apt-get upgrade -y
sudo rm -f /etc/systemd/network/99-default.link
sudo reboot
```
**安装OMV**
``` linux
wget  https://cdn.jsdelivr.net/gh/OpenMediaVault-Plugin-Developers/installScript@master/install

chmod +x install

sudo ./install -n
```

**配置OMV**
浏览器输入树莓派IP地址就可以进入NAS系统了。
`用户名默认为admin，密码为openmediavault`


# 树莓派搭建Samba 文件共享服务器

一、树莓派挂载U盘

1. 创建挂载目录
```
// 创建目录
sudo mkdir -p /home/pi/udisk
​
// 为目录设置读写权限
sudo chmod 777 /home/pi/udisk
```

2. 查看U盘信息
```
插入U盘到树莓派上后，输入 sudo fdisk -l 查看硬盘位置和硬盘格式

Device     Boot Start       End   Sectors  Size Id Type
/dev/sda1          32 120176639 120176608 57.3G  7 HPFS/NTFS/exFAT
如图所见，硬盘位置为 /dev/sda1 ,硬盘格式是NTFS格式

U盘最好先做好格式化，在windows上面格式化为NTFS格式，否则可能会出现挂载中文文件名乱码等奇怪问题
```
3. 挂载U盘到树莓派
```
将硬盘位置/dev/sda1 挂载到我们创建的udisk挂载目录

sudo mount /dev/sda1 /home/pi/udisk


挂载报错：Mount is denied because the NTFS volume is already exclusively opened.
The volume may be already mounted, or another software may use it which
could be identified for example by the help of the ‘fuser’ command.

出现错误的原因：已经被挂载或者有应用程序正在使用它。

执行命令$fuser  /dev/sda1 查看正在使用它的应用程序
root@pi:/home/pi// fuser /dev/sda1
/dev/sda1:            1918

root@pi:/home/pi// kill 1918
root@pi:/home/pi// sudo mount /dev/sda1 /home/pi/udisk

执行命令：$kill 1918 杀死它。
再次挂载：sudo mount /dev/sda1 /home/pi/udisk

成功！
```
4. 测试挂载效果
```
创建文件查看是否挂载成功，我们可以使用vim创建中文名文件看中文格式是否正常

sudo vim 测试.txt

vim是Linux下最常用的文本编译器，在终端输入vim时可能会出现

Connand 'vim' not found, but can be installed with:

这是因为默认的文本编译器是vi, 而没有安装vim的缘故

输入 sudo apt-get install vim 安装vim


sudo nano /etc/apt/sources.list
```

二、设置开机自动挂载
要实现开机自动挂载U盘，我们需要将U盘的设备信息写入到/etc/fstab文件中

1. 查看硬盘UUID信息
```
blkid

我们记录下信息准备后面使用：
 /dev/sda1:  UUID="0CDC5408DC53EA8A"  PARTUUID="69016b36-01"
```
2. 挂载信息写入配置文件
```
查看下cat /etc/fstab文件的内容

将以下信息，添加到/etc/fstab文件末尾
使用nano编辑/etc/fstab文件，追加下面的内容
sudo nano  /etc/fstab

UUID=0CDC5408DC53EA8A  /home/pi/udisk  ntfs defaults 0 0
```

3. 使配置生效
```
sudo mount -a
```

4. 查看挂载情况
```
df -h
```
5.重启服务查看自动挂载效果
```
//重启
sudo reboot
```

三、树莓派部署Samba服务
1. 安装samba服务
```
sudo apt-get install samba
```

2. 创建账户与设置密码
```
这里把pi为用samba的登录用户，并设置密码

// 创建samba配置的密码文件
sudo touch /etc/samba/smbpasswd
​
// 添加smb账户
sudo smbpasswd -a pi
执行smbpasswd命令后会提示输入samba的账户密码，这个密码后面访问smb服务会用到，
我这里使用pi这个默认root用户所以不用新建，用户需要在系统中存在，没有则先用useradd创建。
```

3. 设置samba的配置文件
```
需要在smaba配置中指定相关的smb共享文件夹

sudo nano /etc/samba/smb.conf
将如下配置添加到smb.conf最后面

//树莓派本磁盘配置，挂载的是test目录
[local]
    comment = local
    path = /home/pi/test
    browseable=yes
    writable=yes
    //public=yes
    //available= yes

    //这一行是特别重要的，设置可以被共享访问！！
    guest ok=yes

    create mask = 0775
    directory mask = 0775

    //将新文件的所有者和组属性设置为属于root用户
    forceuser = pi
    forcegroup = pi
​
// u盘挂载的配置，目录是挂载目录
[udisk]
    comment = udisk
    path = /home/pi/udisk
    browseable=yes
    writable=yes
    //public=yes
    //available= yes

    //这一行是特别重要的，设置可以被共享访问！！
    guest ok=yes

    create mask = 0775
    directory mask = 0775

    //将新文件的所有者和组属性设置为属于root用户
    forceuser = pi
    forcegroup = pi

上面我添加了两个共享目录配置，
一个是local共享的是树莓派本地的文件夹/home/ubuntu/test，
一个是udisk共享的是U盘挂载的/home/ubuntu/udisk文件夹，
大家可以根据自己情况设置一个或是多个

各个参数具体含义如下：

[udisk]：分享名称
comment：备注描述
path：共享文件夹目录
writable：是否可写入，不能写入就不能创建文件夹
browseable：是否可以访问浏览
```

4. 重启服务使配置生效
```
使用下面的命令效果相同

// 重启服务
sudo service smbd restart
​
// 查看服务状态
sudo service smbd status
```

5. 设置开机自动启动
```
// 开机自启动
sudo systemctl enable smbd


sudo chmod 777 /home/pi/udisk
sudo chmod 777 /home/pi/test

sudo service smb restart 
```
