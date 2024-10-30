---
title: Linux日常
description: linux, rsync
hide:
  - navigation
---

## 一、复制大文件显示进度

``` shell
rsync -av --progress t01/demo.zip t02/
```

## 二、系统u盘制作

``` shell
sudo dd bs=4M if=/path/to/arcolinux...iso of=/dev/sdX status=progress && sync
```

## 三、查看Manjaro安装的软件

``` shell
cat /var/log/pacman.log | grep -E installed > installed
```

## 四、文件备份

``` shell
[shafish@shafish-laptop shafish]# sudo su
[root@shafish-laptop shafish]# cd /
[root@shafish-laptop shafish]# tar cvpzf system_backup_root.tar.gz \
--exclude=/proc \
--exclude=/lost+found \
--exclude=/dev \
--exclude=/lib \
--exclude=/mnt \
--exclude=/sys \
--exclude=/home \
--exclude=/run/media \
--one-file-system \
--exclude=/system_backup_root.tar.gz \
/
[root@shafish-laptop shafish]# cd /home
[root@shafish-laptop shafish]# tar cvpzf system_backup_home.tar.gz \
--exclude=/home/shafish/node_modules \
--exclude=/home/shafish/.m2 \
--exclude=/home/shafish/snap \
--exclude=/home/system_backup_home.tar.gz \
/home
```

## 五、git代理
``` shell
git config --global http.proxy 192.168.2.1:7890
git config --global https.proxy 192.168.2.1:7890
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 六、硬盘挂载
``` shell
# sudo
# 1. 插上硬盘
# 2. 查看对应设备名 (/dev/sda)
lsblk
# 3. 创建分区
fdisk /dev/sda
# 3.1 输入 n 创建一个新的分区
n
# 3.2 选择默认 p 选择主分区
p
# 3.3 输入分区号，默认从1开始，默认回车
# 3.4 first sector 起始扇区 (2048-4294967295, 默认 2048)：默认回车
# 3.5 last sector 结束扇区，默认使用剩余全部（或者自己需要多少，慢慢算）：回车
# 4. 保存设置
w
# 5.格式化分区
mkfs.ext4 /dev/sda1
# 6. 创建挂载点
mkdir /data
# 7. 挂载
mount /dev/sda1 /data
# 8. 设置开机自动挂载
# 8.1 查看新分区uuid (6ee49c14-xxxx-4bdd-xxxx-4e5afcd0e2fa)
blkid
# 8.2 备份默认配置
cp /etc/fstab /etc/fstab.bak
# 8.3 添加到fstab配置中 (要挂载的分区设备号	挂载点	文件系统类型	挂载选项	是否备份	是否检测)
echo "UUID=6ee49c14-xxxx-4bdd-xxxx-4e5afcd0e2fa /data ext4 defaults 0 0" >> /etc/fstab
# 9. 挂载配置 mount -a
# 10. 关机重启逝世
```

## 七、pve smb
``` shell
# sudo
# 1.安装smb
apt update && apt install samba -y
# 2. 验证安装 systemctl start nmbd
systemctl status nmbd
# 3. 创建共享文件夹
mkdir /data/smb/movie
# 4. 添加smb系统用户
useradd sha
smbpasswd -a sha
# 5.设置目录权限
chown -R sha /data/smb/movie
# 6. 备份默认配置
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
# 7. 开始配置
nano /etc/samba/smb.conf
```
``` conf
# 文件末尾添加
# 共享名称
[samba-movie]
# 描述
comment = for movie
# 共享路径
path = /data/smb/movie
# 要登录
guest ok = no
# 该共享设置为只读
read-only = no
# 可浏览
browsable = no
# smb用户
write list = sha
```
``` shell
# 8. 重启smb
systemctl restart smbd.service
```

``` shell
[wd]                                          # 自定义共享名
      comment = Home Directories                 # 描述符，是给系统管理员看的
      path = /home/wd                         # 共享的路径
      public = yes                               # 是否公开，也就是是否能在网上邻居看到该共享
      browseable = yes                           # 共享的目录是否让所有人可见
      writable = yes                             # 是否可写
      guest ok = no                              # 是否拒绝匿名访问，仅当安全级别为 share 时才生效
      workgroup = WORKGROUP                      # 工作组，要设置成跟 Windows 的工作组一致
      server string = Samba Server Version %v    # 其他 Linux 主机查看共享时的提示符
      netbios name = MYSERVER                    # 用于在 Windows 网上邻居上显示的主机名
      hosts allow = 127. 192.168.12. 192.168.13. EXCEPT 192.168.13.13       # 指定允许访问 samba 服务器的主机   
      security = share                           # 定义安全级别
      log file = /var/log/samba/log.%m           # 定义日志文件，每个访问的主机会产生独立的日志文件，%m 是客户端主机名
      max log size = 50                          # 定义单个日志的最大容量（KB）
      passdb backend = tdbsam                    # Samba 用户的存储方式，smbpasswd 表示明文存储，tdbsam 表示密文存储
      deadtime = 10                              # 客户端在10分钟内没有打开任何 Samba 资源，服务器将自动关闭会话，在大量的并发访问环境中，这样的设置可以提高服务器性能
      display charset = UTF8                     # 设置显示的字符集
      max connections = 0                        # 设置最大连接数，0表示无限制，如果超过最大连接数则拒绝连接
      guest account = nobody                     # 设置匿名账户为nobody
      load printers = yes                        # 是否在启动 Samba 时就共享打印机   
      cups options = raw                         # 设置打印机使用的方式
      valid users = user1 user2    user3         # 指定哪些用户可以访问，如果不指定则所有用户都可访问
      invalid users = user1 user2                # 指定哪些用户不可以访问
      create mask = 0775                         # 客户端上传文件的默认权限
      directory mask = 0775                      # 客户端创建目录的默认权限
      write list = user1 user2 user3             # 设置可对文件进行写操作的用户
      admin users = user1                        # 设置共享目录的管理员，具有完全权限
```

## 八、挂载远程smb
``` shell
# 1. 安装挂载smb相关组件
apt install cifs-utils -y
# 2. 本地挂载目录
mkdir /data/movie
# 3. 挂载smb mount -t cifs -o username=sha,password=xxx //192.168.2.100/samba-movie /data/movie
nano /etc/fstab
//192.168.2.100/samba-movie /data/movie cifs username=sha,password=xxxx 0 0
# mount -t cifs //xxx-crf23.eu-west-1.nas.aliyuncs.com/myshare /mnt -o vers=2.0,guest,uid=0,gid=0,dir_mode=0755,file_mode=0755,mfsymlinks,cache=strict,rsize=1048576,wsize=1048576
```

## 九、scp远程传输文件
``` shell
# 本地 to 远程
scp /run/media/shafish/movie/xxxfile root@192.168.2.100:/data/smb/movie
scp -r /run/media/shafish/movie/xxxdir root@192.168.2.100:/data/smb/movie
```

## 十、mysql
``` shell
# 安装
apt install mysql-server
# 检查
systemctl status mysql
# 配置
mysql_secure_installation
# 授权
GRANT ALL PRIVILEGES ON *.* TO 'administrator'@'localhost' IDENTIFIED BY 'very_strong_password888';
```

## 十、计算文件md5、hash值
``` shell
md5sum xxxfile
sha1sum xxxfile
sha256sum xxxfile
```

## 十一、bbr
``` shell
bash <(curl -Lso- https://git.io/kernel.sh)
```


## 十二、向日葵
``` shell
pacman -S yay
yay -S sunloginclient
systemctl start runsunloginclient.service
systemctl enable runsunloginclient.service
```

## 十三、aliyundrive-fuse
``` shell
apt-get -y install python3-pip
pip install aliyundrive-fuse
mkdir -p /data/smb/aliyun/data
vim /usr/lib/systemd/system/aliyun.service
```
``` bash
[Unit]
Description=AliyunDrive FUSE
After=network.target

[Service]
Type=simple
PermissionsStartOnly=true
ExecStart=/usr/local/bin/aliyundrive-fuse --allow-other -r xxxtoken -w /var/run/aliyundrive-fuse /data/smb/aliyun/data
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

``` shell
systemctl daemon-reload
systemctl start aliyun.service
systemctl status aliyun.service
systemctl enable aliyun.service
```

## 十四、bspwm
``` shell
sudo pacman -Syu bspwm sxhkd
echo "sxhkd &
exec bspwm" >> ~/.xinitrc
install -Dm755 /usr/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm/bspwmrc
install -Dm644 /usr/share/doc/bspwm/examples/sxhkdrc ~/.config/sxhkd/sxhkdrc
bspc monitor -d I II III IV V VI VII VIII IX X
```

https://wiki.archlinux.org/title/Bspwm


## 十五、特定目录下查找文件内容

``` shell
grep -nri [搜索词] 目录 | cat --number
```

## 十六、unzip中文解压乱码
``` shell
unzip -O CP936/GBK/GB18030 xxxx.zip
```

## 十七、vim中文乱码
vim ~/.vimrc

``` shell
set termencoding=utf-8
set encoding=utf8
set fileencodings=utf8,ucs-bom,gbk,cp936,gb2312,gb18030
```

## 十八、netstat
使用netstat报错command not found
`apt install net-tools`

netstat -aptn

## 十九、centos换源

``` shell
cd /etc/yum.repos.d/
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```