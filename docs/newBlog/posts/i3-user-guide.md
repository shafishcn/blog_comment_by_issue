---
title: Archlinx主力机配置记录
authors:
    - shafish
date:
    created: 2023-10-31
    updated: 2024-01-20
categories:
    - archlinx
    - 装机
---

[ :fishing_pole_and_fish: ](../../index.md)

## 硬件环境
- CPU:AMD5900X
- 主板:华擎x570太极雷蛇
- 显卡:3060Ti 8G
- 台式机！！！

## 一、archlinx系统安装
视频参考：[https://www.bilibili.com/video/BV1J34y1f74E/](https://www.bilibili.com/video/BV1J34y1f74E/){target=_blank}

### 1.1 archlinux USB引导安装

???+ "把arch镜像写入U盘就行"

    - 插入U盘
    - 下载写盘工具[BalenaEtcher](https://etcher.balena.io/){target=_blank} 或者 [Rufus](https://rufus.ie/){target=_blank}
    - 下载[Archlinux镜像](https://archlinux.org/download/){target=_blank}

### 1.2 设置U盘启动，进入live环境

???+ "F2 bro"

    - 根据主板进入bios（一般是F2或者F12），禁用安全启动，并选择U盘启动

    启动后出现引导加载程序菜单，选择Arch Linux install medium 确认就行

<!-- more -->

### 1.3 硬盘分区配置(整个磁盘，非双系统)

???+ "zhitai yyds"

    内存>=16G建议就别创建交换空间了，如果确实程序需要高内存，建议直接加内存。

    - 查看硬盘：`lsblk`（磁盘对应映射文件为/dev/sdx或者/dev/nvmexxx）
    - 创建分区：`cfdisk /dev/nvme0n1`
        - efi引导分区：500M (nvme0n1p1)
        - /根分区：剩余所有容量 (nvme0n1p2)
    - 分区格式化
        - 根分区格式化：`mkfs.ext4 /dev/nvme0n1p2`
        - efi分区格式化：`mkfs.fat -F 32 /dev/nvme0n1p1`
    - 分区挂载（挂载分区一定要遵循顺序，先挂载根分区，再挂载引导分区）
        - 挂在根分区：`mount /dev/nvme0n1p2 /mnt`
        - 挂在引导分区：`mount --mkdir /dev/nvme0n1p1 /mnt/boot`

    - 如果原硬盘已存在分区，备份好其中资料，删除分区后，再继续上述操作
        - `sudo fdisk /dev/nvme0n1`
        - 输入 `p` 查看分区
        - 输入 `d` 删除分区
        - 输入分区编号 1、2 即可删除对应分区
        - 输入 `w` 写入操作

### 1.4 安装系统包

???+ "linux-zen是华点"

    这里选择安装[linux-zen内核](https://wiki.archlinuxcn.org/wiki/%E5%86%85%E6%A0%B8){target=_blank}，需要特别留意待会显卡驱动的安装。amd-ucode是amd的微码包。

    - `pacstrap -K /mnt base linux-zen linux-firmware base-devel amd-ucode`

### 1.5 挂载系统目录
- `genfstab -U /mnt >> /mnt/etc/fstab`

### 1.6 进入挂载系统
- `arch-chroot /mnt`

#### 1.6.1 时区配置
- `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
- `hwclock --systohc`

#### 1.6.2 本地化配置
[https://wiki.archlinuxcn.org/wiki/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E6%9C%AC%E5%9C%B0%E5%8C%96](https://wiki.archlinuxcn.org/wiki/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E6%9C%AC%E5%9C%B0%E5%8C%96){target=_blank}

???+ "先选择en_US"

    - 安装编辑器：`pacman -S vim`
    - 修改配置文件：`vim /etc/locale.gen`（取消掉 en_US.UTF-8 UTF-8前的注释符#）
    - 生成locale信息：`locale-gen`
    - 修改配置文件：`vim /etc/locale.conf`（添加LANG=en_US.UTF-8）

#### 1.6.3 网络配置

???+ "NetworkManager不错"

    [https://wiki.archlinuxcn.org/wiki/NetworkManager](https://wiki.archlinuxcn.org/wiki/NetworkManager){target=_blank}

    - 配置主机名：`vim /etc/hostname`（添加主机名）
    - `pacman -S networkmanager nm-connection-editor network-manager-applet`
    - `systemctl enable NetworkManager.service`

#### 1.6.4 root用户配置

???+ "以后要把自定义用户加到wheel组"

    - `passwd`
    - `pacman -S sudo`
    - `visudo`（去掉%wheel ALL=(ALL:ALL) ALL 注释）

#### 1.6.5 开机引导配置

???+ "efi+GRUB的组合"

    [https://wiki.archlinuxcn.org/wiki/GRUB](https://wiki.archlinuxcn.org/wiki/GRUB){target=_blank}

    - `pacman -S grub efibootmgr`
    - `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`
    - `grub-mkconfig -o /boot/grub/grub.cfg`

### 1.7 系统重启
- `exit`
- `reboot`

## 二、系统配置

### 2.1 安装yay、archlinuxcn

???+ "yay+archlinuxcn无敌"

    - 修改pacman配置文件：`vim /etc/pacman.conf`
        ```
        [multilib]
        Include = /etc/pacman.d/mirrorlist
        [archlinuxcn]
        Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
        ```
    - 更新下系统：`pacman -Syu`
    - 密钥更新：`pacman -S archlinuxcn-keyring`
    - yay安装：`pacman -S yay`

### 2.2 添加普通用户

???+ "shafish来了"

    - 创建用户：`useradd -m shafish`
    - 将该用户加到wheel组中：`usermod -a -G wheel shafish`
    - 设置用户密码：`passwd shafish`
    - 创建默认目录：比如Download、Document等目录
        - 切换到新建用户：`su - shafish`
        - `sudo pacman -S xdg-user-dirs`
        - `LC_ALL=C xdg-user-dirs-update --force`

### 2.3 窗口管理器-I3WM
i3-gaps已经合并到i3wm 4.23版本，别安装i3-gaps了。
[https://wiki.archlinuxcn.org/wiki/I3](https://wiki.archlinuxcn.org/wiki/I3){target=_blank}
[https://i3wm.org/docs/userguide.html](https://i3wm.org/docs/userguide.html){target=_blank}

???+ "xorg是显示服务器，i3wm是窗口管理器。xorg是基础框架，i3wm及其系列组件则提供桌面环境支持"

    - `pacman -S xorg-server xorg-apps i3-wm i3status`

???+ "vim ~/.config/i3/config"

    ``` shell
    # 窗口隐藏
    bindsym $mod+minus move scratchpad
    bindsym $mod+plus scratchpad show
    # 边距
    gaps inner 3px
    gaps outer 3px
    gaps top 1px
    smart_gaps inverse_outer
    # 去掉窗口title
    for_window [class="^.*"] border pixel 0
    # 工作区切换
    workspace_auto_back_and_forth yes
    ```

### 2.4 显示管理器-SDDM
[https://wiki.archlinuxcn.org/wiki/SDDM](https://wiki.archlinuxcn.org/wiki/SDDM){target=_blank}

???+ "别用xorg-xinit的startx进入桌面环境了"

    - 安装sddm包：`sudo pacman -S --needed sddm qt5-graphicaleffects qt5-quickcontrols2 qt5-svg`
    - 开机启动sddm服务：`sudo systemctl enable sddm`
    - 启动服务：`sudo systemctl start sddm`
    - 创建配置目录：`sudo mkdir /etc/sddm.conf.d`
    - 生成默认配置文件：`sudo sddm --example-config > /etc/sddm.conf.d/sddm.conf`
    - 使用sddm-sugar-candy主题：
        - `cd /usr/share/sddm/themes/`
        - `sudo git clone https://framagit.org/MarianArlt/sddm-sugar-candy.git`
        - `sudo vim /etc/sddm.conf.d/sddm.conf` 中配置 Current=sddm-sugar-candy

### 2.5 终端模拟器
- `pacman -S alacritty`

- 装完终端后再装shell（用惯了zsh）
    - 安装：`sudo pacman -S zsh`
    - 切换：`chsh -s /bin/zsh`
    - 检查（注销后重新登录）：`echo $SHELL`
    - 配置代理
    ``` shell title="~/.zshrc"
    # where proxy
    proxy () {
        export http_proxy="http://127.0.0.1:8087"
        export https_proxy=$http_proxy
        echo "HTTP Proxy on"
    }

    # where noproxy
    noproxy () {
        unset http_proxy
        unset https_proxy
        echo "HTTP Proxy off"
    }

    ckproxy () {
        curl ipinfo.io
    }

    . /etc/zsh_command_not_found
    ```
    ``` shell title="/etc/zsh_command_not_found"
    # 找不到命令时，提示命令对应需安装的软件
    # (c) Zygmunt Krynicki 2007,
    # Licensed under GPL, see COPYING for the whole text
    #
    # This script will look-up command in the database and suggest
    # installation of packages available from the repository

    if [[ -x /usr/lib/command-not-found ]] ; then
            if (( ! ${+functions[command_not_found_handler]} )) ; then
                    function command_not_found_handler {
                            [[ -x /usr/lib/command-not-found ]] || return 1
                            /usr/lib/command-not-found -- ${1+"$1"} && :
                    }
            fi
    fi
    ```
    - 安装oh-zsh：`sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

[https://zhuanlan.zhihu.com/p/393405979](https://zhuanlan.zhihu.com/p/393405979){target=_blank}

[https://huangno1.github.io/archlinux_install_part4_on_my_zsh/](https://huangno1.github.io/archlinux_install_part4_on_my_zsh/){target=_blank}

### 2.6 蓝牙
[https://wiki.archlinuxcn.org/wiki/%E8%93%9D%E7%89%99](https://wiki.archlinuxcn.org/wiki/%E8%93%9D%E7%89%99){target=_blank}

- `sudo pacman -S bluez bluez-utils`
- `sudo systemctl enable bluetooth.service`
- `sudo systemctl start bluetooth.service`

### 2.7 声音
[https://wiki.archlinuxcn.org/wiki/PulseAudio](https://wiki.archlinuxcn.org/wiki/PulseAudio){target=_blank}

- `sudo pacman -S pulseaudio`
- `systemctl --user enable pulseaudio --now`
- `sudo pacman -S alsa-utils alsa-plugins volumeicon`
- `sudo pacman -S pulseaudio-bluetooth blueman`

切换HDMI外接音频设备输出(视频采集卡用)：`yay -S indicator-sound-switcher` 选择 hdmi 输出即可

### 2.8 字体
[https://wiki.archlinuxcn.org/wiki/%E5%AD%97%E4%BD%93](https://wiki.archlinuxcn.org/wiki/%E5%AD%97%E4%BD%93){target=_blank}

- 中文字体
    - `sudo pacman -S wqy-microhei`
- 字体大小
    [https://wiki.archlinuxcn.org/wiki/Xrandr](https://wiki.archlinuxcn.org/wiki/Xrandr){target=_blank}

    - `vim ~/.xprofile`
    - `xrandr --dpi 192 &`

- 字体管理：`pacman -S font-manager`

### 2.9 系统主题
- `pacman -S arc-icon-theme adwaita-icon-theme capitaine-cursors papirus-icon-theme arc-gtk-theme lxappearance`

用lxappearance选择心水的图标、主题

### 2.10 硬盘挂载

- 查看硬盘：`lsblk`
- 查看挂载磁盘的id：`sudo blkid /dev/sda` (/dev/sdb: UUID="a1c1e224-ee15-4732-bd39-af79869b84ae" BLOCK_SIZE="4096" TYPE="ext4")
- 创建硬盘挂载点：`sudo mkdir /mnt/wd`
- 设置开机自动挂载：`echo "UUID=a1c1e224-ee15-4732-bd39-af79869b84ae /mnt/wd ext4 defaults 0 0" > /etc/fstab`

### 2.11 ssh
- 安装服务：`sudo pacman -S openssh`
- 启用：`sudo systemctl start sshd`
- 开机启用：`sudo systemctl enable sshd`
- 用户登陆：`echo "PermitRootLogin yes" >> /etc/ssh/sshd_config`
- 删除重复ip：`ssh-keygen -f "/root/.ssh/known_hosts" -R "ipxxxx"`

- 权限（权限不对时使用不了）：
``` 
.ssh 目录配置700权限
id_rsa、authorized_keys等 配置600权限
id_rsa.pub、known_hosts等 配置644权限
```

### 2.12 镜像加速

- 输出国内访问速度前5的镜像地址：`curl -s "https://archlinux.org/mirrorlist/?country=CN&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 5 -`
- 复制到`/etc/pacman.d/mirrorlist` 文件开头即可
- 强制更新镜像（所有软件包列表）：`sudo pacman -Syyu`

### 2.13 时间不准

xx mouth later

``` shell
sudo timedatectl set-ntp true
```

## 三、软件配置

???+ "vim ~/.config/i3/config"

    ``` shell
    # 网络托盘，对应network-manager-applet包
    exec --no-startup-id nm-applet
    # 终端透明，对应picom包
    exec --no-startup-id picom -b
    # 设置壁纸，对应feh包
    exec --no-startup-id feh --randomize --bg-fill /home/shafish/Pictures/wallpaper
    # 启动fcitx5，输入法用
    exec --no-startup-id fcitx5 -d
    # 蓝牙托盘
    exec --no-startup-id blueman-applet
    # 启动polybar
    exec --no-startup-id $HOME/.config/polybar/launch.sh
    # 设置开机色温 redshift
    # exec --no-startup-id redshift -P -O 4000
    # 设置启动器快捷键 rofi
    bindsym $mod+d exec "rofi -combi-modi window,drun,ssh -show combi"
    # 截图保存到指定目录
    bindsym Print exec --no-startup-id maim "/home/$USER/Pictures/$(date)"
    bindsym $mod+Print exec --no-startup-id maim --window $(xdotool getactivewindow) "/home/$USER/Pictures/$(date)"
    bindsym Shift+Print exec --no-startup-id maim --select "/home/$USER/Pictures/$(date)"
    # 截图保存到粘贴板
    bindsym Ctrl+Print exec --no-startup-id maim | xclip -selection clipboard -t image/png
    bindsym Ctrl+$mod+Print exec --no-startup-id maim --window $(xdotool getactivewindow) | xclip -selection clipboard -t image/png
    bindsym Ctrl+Shift+Print exec --no-startup-id maim --select | xclip -selection clipboard -t image/png
    ```

### 3.1 启动器（window系统的window键）
- `sudo pacman -S rofi`

### 3.2 壁纸、终端透明、装B
- `sudo pacman -S feh picom neofetch`

### 3.3 显卡驱动
[https://wiki.archlinuxcn.org/wiki/NVIDIA](https://wiki.archlinuxcn.org/wiki/NVIDIA){target=_blank}

???+ "之前安装的是linux-zen内核，注意这里是安装nvidia-dkms"

    - `sudo pacman -S linux-zen-headers nvidia-dkms nvidia-settings`
    - `vim /etc/mkinitcpio.conf`
        - 删去HOOKS那一项中的kms，阻止内核启动时加载nouveau
    - `sudo mkinitcpio -P`
    - `reboot`
    - `nvidia-smi`

### 3.4 输入法
[https://wiki.archlinuxcn.org/wiki/Fcitx5](https://wiki.archlinuxcn.org/wiki/Fcitx5){target=_blank}

???+ "安装fcitx5-input-support，就别手动配/etc/environment了"

    - `sudo pacman -S fcitx5-im fcitx5-rime fcitx5-chinese-addons rime-luna-pinyin rime-emoji`
    - `yay -S fcitx5-input-support`

### 3.5 polybar 3.7.0+
[https://github.com/polybar/polybar-scripts](https://github.com/polybar/polybar-scripts){target=_blank}

`sudo pacman -S polybar speedtest-cli redshift jq udisks2`

#### redshift
[https://github.com/VineshReddy/polybar-redshift](https://github.com/VineshReddy/polybar-redshift){target=_blank}

#### usb
[https://github.com/polybar/polybar-scripts/tree/master/polybar-scripts/system-usb-udev](https://github.com/polybar/polybar-scripts/tree/master/polybar-scripts/system-usb-udev){target=_blank}

???- "polybar相关配置"

    ???- "launch.sh"

        ``` shell title="~/.config/polybar/launch.sh"
        #!/bin/bash

        killall -q polybar
        polybar mybar 2>&1 | tee -a /tmp/polybar.log & disown
        echo "Polybar launched..."
        ```

    ???- "config.ini"

        ``` shell title="~/.config/polybar/config.ini"
        ;==========================================================
        ;
        ;
        ;   ██████╗  ██████╗ ██╗  ██╗   ██╗██████╗  █████╗ ██████╗
        ;   ██╔══██╗██╔═══██╗██║  ╚██╗ ██╔╝██╔══██╗██╔══██╗██╔══██╗
        ;   ██████╔╝██║   ██║██║   ╚████╔╝ ██████╔╝███████║██████╔╝
        ;   ██╔═══╝ ██║   ██║██║    ╚██╔╝  ██╔══██╗██╔══██║██╔══██╗
        ;   ██║     ╚██████╔╝███████╗██║   ██████╔╝██║  ██║██║  ██║
        ;   ╚═╝      ╚═════╝ ╚══════╝╚═╝   ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝
        ;
        ;
        ;   To learn more about how to configure Polybar
        ;   go to https://github.com/polybar/polybar
        ;
        ;   The README contains a lot of information
        ;
        ;==========================================================

        [colors]
        background = #282A2E
        background-alt = #373B41
        foreground = #C5C8C6
        primary = #F0C674
        secondary = #8ABEB7
        alert = #A54242
        disabled = #707880

        [bar/mybar]
        width = 100%
        height = 24pt
        radius = 6
        monitor = DP-4
        ; dpi = 96

        background = ${colors.background}
        foreground = ${colors.foreground}

        line-size = 3pt

        border-size = 4pt
        border-color = #00000000

        padding-left = 0
        padding-right = 1

        module-margin = 1

        separator = |
        separator-foreground = ${colors.disabled}

        font-0 = monospace;2
        ;wlan
        ;font-0 = HarmonyOS Sans;3
        modules-left = xworkspaces
        modules-right = speedtest system-usb-udev filesystem redshift pulseaudio xkeyboard memory cpu temperature gputemp eth date
        modules-center = tray

        cursor-click = pointer
        cursor-scroll = ns-resize

        enable-ipc = true

        ; tray-position = center
        ; tray-maxsize = 20

        ; wm-restack = generic
        ; wm-restack = bspwm
        ; wm-restack = i3
        ; override-redirect = true

        [module/tray]
        type = internal/tray

        format-margin = 8px
        tray-spacing = 8px

        [module/xworkspaces]
        type = internal/xworkspaces

        label-active = %name%
        label-active-background = ${colors.background-alt}
        label-active-underline= ${colors.primary}
        label-active-padding = 1

        label-occupied = %name%
        label-occupied-padding = 1

        label-urgent = %name%
        label-urgent-background = ${colors.alert}
        label-urgent-padding = 1

        label-empty = %name%
        label-empty-foreground = ${colors.disabled}
        label-empty-padding = 1

        [module/xwindow]
        type = internal/xwindow
        label = %title:0:60:...%

        [module/filesystem]
        type = internal/fs
        interval = 25

        mount-0 = /

        label-mounted = %{F#F0C674}%mountpoint%%{F-} %percentage_used%%

        label-unmounted = %mountpoint% not mounted
        label-unmounted-foreground = ${colors.disabled}

        [module/pulseaudio]
        type = internal/pulseaudio

        format-volume-prefix = "VOL "
        format-volume-prefix-foreground = ${colors.primary}
        format-volume = <label-volume>

        label-volume = %percentage%%

        label-muted = muted
        label-muted-foreground = ${colors.disabled}

        [module/xkeyboard]
        type = internal/xkeyboard
        blacklist-0 = num lock

        label-layout = %layout%
        label-layout-foreground = ${colors.primary}

        label-indicator-padding = 2
        label-indicator-margin = 1
        label-indicator-foreground = ${colors.background}
        label-indicator-background = ${colors.secondary}

        [module/memory]
        type = internal/memory
        interval = 2
        format-prefix = "RAM "
        format-prefix-foreground = ${colors.primary}
        label = %percentage_used:2%%

        [module/cpu]
        type = internal/cpu
        interval = 2
        format-prefix = "CPU "
        format-prefix-foreground = ${colors.primary}
        label = %percentage:2%%

        [network-base]
        type = internal/network
        interval = 5
        format-connected = <label-connected>
        format-disconnected = <label-disconnected>
        label-disconnected = %{F#F0C674}%ifname%%{F#707880} disconnected

        [module/wlan]
        inherit = network-base
        interface-type = wireless
        label-connected = %{F#F0C674}%ifname%%{F-} %essid% %local_ip%

        ; [module/eth]
        ; inherit = network-base
        ; interface-type = wired
        ; label-connected = %{F#F0C674}%ifname%%{F-} %local_ip%

        [module/eth]
        type = internal/network
        ;请将interface设为自己的网卡名称;
        interface = enp6s0
        interval = 2.0

        ; format-connected-underline = #55aa55
        ; format-connected-prefix = "enp6s0 "
        ;format-connected-prefix-foreground = ${colors.foreground-alt}
        ;label-connected = %local_ip%

        ; Seconds to sleep between updates
        ; Default: 1
        ;interval = 3.0

        ; Test connectivity every Nth update
        ; A value of 0 disables the feature
        ; NOTE: Experimental (needs more testing)
        ; Default: 0
        ;ping-interval = 3

        ; @deprecated: Define min width using token specifiers (%downspeed:min% and %upspeed:min%)
        ; Minimum output width of upload/download rate
        ; Default: 3
        ;udspeed-minwidth = 5

        ; Accumulate values from all interfaces
        ; when querying for up/downspeed rate
        ; Default: false
        accumulate-stats = true

        ; Consider an `UNKNOWN` interface state as up.
        ; Some devices like USB network adapters have 
        ; an unknown state, even when they're running
        ; Default: false
        unknown-as-up = true



        ; Available tags:
        ;   <label-connected> (default)
        ;   <ramp-signal>
        format-connected =  <label-connected>

        ; Available tags:
        ;   <label-disconnected> (default)
        format-disconnected = <label-disconnected>

        ; Available tags:
        ;   <label-connected> (default)
        ;   <label-packetloss>
        ;   <animation-packetloss>
        format-packetloss = <animation-packetloss> <label-connected>

        ; All labels support the following tokens:
        ;   %ifname%    [wireless+wired]
        ;   %local_ip%  [wireless+wired]
        ;   %local_ip6% [wireless+wired]
        ;   %essid%     [wireless]
        ;   %signal%    [wireless]
        ;   %upspeed%   [wireless+wired]
        ;   %downspeed% [wireless+wired]
        ;   %linkspeed% [wired]

        ; Default: %ifname% %local_ip%
        label-connected = %local_ip% %downspeed:9%

        ; Default: (none)
        ;label-packetloss = %essid%
        ;label-packetloss-foreground = #eefafafa

        ; Only applies if <ramp-signal> is used
        ramp-signal-0 = 😱
        ramp-signal-1 = 😠
        ramp-signal-2 = 😒
        ramp-signal-3 = 😊
        ramp-signal-4 = 😃
        ramp-signal-5 = 😈

        ; Only applies if <animation-packetloss> is used
        animation-packetloss-0 = ⚠
        animation-packetloss-0-foreground = #ffa64c
        animation-packetloss-1 = 📶
        animation-packetloss-1-foreground = #000000
        ; Framerate in milliseconds
        animation-packetloss-framerate = 500


        ;format-disconnected =
        ;format-disconnected = <label-disconnected>
        ;format-disconnected-underline = ${self.format-connected-underline}
        ;label-disconnected = %ifname% disconnected
        ;label-disconnected-foreground = ${colors.foreground-alt}

        [module/date]
        type = internal/date
        interval = 1

        date = %H:%M
        date-alt = %Y-%m-%d %H:%M:%S

        label = %date%
        label-foreground = ${colors.primary}

        [settings]
        screenchange-reload = true
        pseudo-transparency = true

        ; vim:ft=dosini

        [module/bright]
        type = custom/script
        exec = $HOME/.config/polybar/script/bright.sh
        scroll-up = "$HOME/.config/polybar/script/bright.sh +"
        scroll-down = "$HOME/.config/polybar/script/bright.sh -"
        interval = 2
        format-prefix = Bri
        format-prefix-foreground = #00cc00
        format-underline = #00cc00
        format-foreground = #00cc00

        [module/speedtest]  
        type = custom/script  
        exec-if = hash speedtest
        exec = "$HOME/.config/polybar/script/polybar-speedtest"  
        interval = 90

        [module/redshift]
        type = custom/script
        format-prefix = "CT "  
        exec = source ~/.config/polybar/script/redshiftenv.sh && ~/.config/polybar/script/redshift.sh temperature 
        click-left = source ~/.config/polybar/script/redshiftenv.sh && ~/.config/polybar/script/redshift.sh toggle 
        scroll-up = source ~/.config/polybar/script/redshiftenv.sh && ~/.config/polybar/script/redshift.sh increase
        scroll-down = source ~/.config/polybar/script/redshiftenv.sh && ~/.config/polybar/script/redshift.sh decrease
        interval=0.5


        [module/temperature]
        type = internal/temperature

        ; Seconds to sleep between updates
        ; Default: 1
        interval = 2

        ; Thermal zone to use
        ; To list all the zone types, run 
        ; $ for i in /sys/class/thermal/thermal_zone*; do echo "$i: $(<$i/type)"; done
        ; Default: 0
        thermal-zone = 0

        ; Full path of temperature sysfs path
        ; Use `sensors` to find preferred temperature source, then run
        ; $ for i in /sys/class/hwmon/hwmon*/temp*_input; do echo "$(<$(dirname $i)/name): $(cat ${i%_*}_label 2>/dev/null || echo $(basename ${i%_*})) $(readlink -f $i)"; done
        ; to find path to desired file
        ; Default reverts to thermal zone setting
        hwmon-path = /sys/class/hwmon/hwmon2/temp1_input

        ; Base temperature for where to start the ramp (in degrees celsius)
        ; Default: 0
        base-temperature = 20

        ; Threshold temperature to display warning label (in degrees celsius)
        ; Default: 80
        warn-temperature = 60
        label = C:%temperature-c%
        ; format-prefix = "C:"

        [module/gputemp]
        type = custom/script
        exec = nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader,nounits
        interval = 5

        label = G:%output%°C

        [module/system-usb-udev]
        type = custom/script
        exec = ~/.config/polybar/script/system-usb-udev.sh
        tail = true
        click-left = ~/.config/polybar/script/system-usb-udev.sh --mount &
        click-right = ~/.config/polybar/script/system-usb-udev.sh --unmount &
        ```


    ???- "script/bright.sh"

        ``` shell title="~/.config/polybar/script/bright.sh"
        #!/bin/bash                                                                                                                                                    
        current=$(cat /sys/class/backlight/intel_backlight/brightness)                                                                                                 
        max=$(cat /sys/class/backlight/intel_backlight/max_brightness)                                                                                                 
        per=$((current*100/max))                                                                                                                                       
        if [ "$1" = "+" ];then                                                                                                                                         
            new=$((per+5))                                                                                                                                             
            if [ $new -gt 100 ];then                                                                                                                                   
            ┆   new=$max                                                                                                                                               
            fi                                                                                                                                                         
            echo $((new*max/100)) |sudo tee /sys/class/backlight/intel_backlight/brightness                                                                            
        elif [ "$1" = "-" ];then                                                                                                                                       
            new=$((per-5))                                                                                                                                             
            if [ $new -lt 0 ];then                                                                                                                                     
            ┆   new=0                                                                                                                                                  
            fi                                                                                                                                                         
            echo $((new*max/100)) |sudo tee /sys/class/backlight/intel_backlight/brightness                                                                            
        else                                                                                                                                                           
            if [ $per -eq 100 ];then                                                                                                                                   
            ┆   echo "$per%"                                                                                                                                           
            elif [ $per -gt 75 ];then                                                                                                                                  
            ┆   echo "$per%"                                                                                                                                           
            elif [ $per -gt 50 ];then                                                                                                                                  
            ┆   echo "$per%"                                                                                                                                           
            elif [ $per -gt 25 ];then                                                                                                                                  
            ┆   echo "$per%"                                                                                                                                           
            else                                                                                                                                                       
            ┆   echo "$per%"                                                                                                                                           
            fi                                                                                                                                                         
        fi
        ```

    ???- "script/redshift.sh"

        ``` shell title="~/.config/polybar/script/redshift.sh"
        #!/bin/sh

        envFile=~/.config/polybar/script/redshiftenv.sh
        changeValue=300

        changeMode() {
        sed -i "s/REDSHIFT=$1/REDSHIFT=$2/g" $envFile 
        REDSHIFT=$2
        echo $REDSHIFT
        }

        changeTemp() {
        if [ "$2" -gt 1000 ] && [ "$2" -lt 25000 ]
        then
            sed -i "s/REDSHIFT_TEMP=$1/REDSHIFT_TEMP=$2/g" $envFile 
            redshift -P -O $((REDSHIFT_TEMP+changeValue))
        fi
        }

        case $1 in 
        toggle) 
            if [ "$REDSHIFT" = on ];
            then
            changeMode "$REDSHIFT" off
            redshift -x
            else
            changeMode "$REDSHIFT" on
            redshift -O "$REDSHIFT_TEMP"
            fi
            ;;
        increase)
            changeTemp $((REDSHIFT_TEMP)) $((REDSHIFT_TEMP+changeValue))
            ;;
        decrease)
            changeTemp $((REDSHIFT_TEMP)) $((REDSHIFT_TEMP-changeValue));
            ;;
        temperature)
            case $REDSHIFT in
            on)
                printf "%dK" "$REDSHIFT_TEMP"
                ;;
            off)
                printf "off"
                ;;
            esac
            ;;
        esac
        ```

    ???- "script/redshiftenv.sh"

        ``` shell title="~/.config/polybar/script/redshiftenv.sh"
        export REDSHIFT=on
        export REDSHIFT_TEMP=4000
        ```

    ???- "script/system-usb-udev.sh"

        ``` shell title="~/.config/polybar/script/system-usb-udev.sh"
        #!/bin/sh

        usb_print() {
            devices=$(lsblk -Jplno NAME,TYPE,RM,SIZE,MOUNTPOINT,VENDOR)
            output=""
            counter=0

            for unmounted in $(echo "$devices" | jq -r '.blockdevices[] | select(.type == "part") | select(.rm == true) | select(.mountpoint == null) | .name'); do
                unmounted=$(echo "$unmounted" | tr -d "[:digit:]")
                unmounted=$(echo "$devices" | jq -r '.blockdevices[] | select(.name == "'"$unmounted"'") | .vendor')
                unmounted=$(echo "$unmounted" | tr -d ' ')

                if [ $counter -eq 0 ]; then
                    space=""
                else
                    space="   "
                fi
                counter=$((counter + 1))

                output="$output$space#1 $unmounted"
            done

            for mounted in $(echo "$devices" | jq -r '.blockdevices[] | select(.type == "part") | select(.rm == true) | select(.mountpoint != null) | .size'); do
                if [ $counter -eq 0 ]; then
                    space=""
                else
                    space="   "
                fi
                counter=$((counter + 1))

                output="$output$space#2 $mounted"
            done

            echo "$output"
        }

        usb_update() {
            pid=$(cat "$path_pid")

            if [ "$pid" != "" ]; then
                kill -10 "$pid"
            fi
        }

        path_pid="/tmp/polybar-system-usb-udev.pid"

        case "$1" in
            --update)
                usb_update
                ;;
            --mount)
                devices=$(lsblk -Jplno NAME,TYPE,RM,MOUNTPOINT)

                for mount in $(echo "$devices" | jq -r '.blockdevices[] | select(.type == "part") | select(.rm == true) | select(.mountpoint == null) | .name'); do
                    udisksctl mount --no-user-interaction -b "$mount"

                    # mountpoint=$(udisksctl mount --no-user-interaction -b $mount)
                    # mountpoint=$(echo $mountpoint | cut -d " " -f 4- | tr -d ".")
                    # terminal -e "bash -lc 'filemanager $mountpoint'"
                done

                usb_update
                ;;
            --unmount)
                devices=$(lsblk -Jplno NAME,TYPE,RM,MOUNTPOINT)

                for unmount in $(echo "$devices" | jq -r '.blockdevices[] | select(.type == "part") | select(.rm == true) | select(.mountpoint != null) | .name'); do
                    udisksctl unmount --no-user-interaction -b "$unmount"
                    udisksctl power-off --no-user-interaction -b "$unmount"
                done

                usb_update
                ;;
            *)
                echo $$ > $path_pid

                trap exit INT
                trap "echo" USR1

                while true; do
                    usb_print

                    sleep 60 &
                    wait
                done
                ;;
        esac
        ```

    ???- "script/polybar-speedtest"

        ``` shell title="~/.config/polybar/script/polybar-speedtest"
        #!/usr/bin/env python3
        import os
        import argparse
        import speedtest

        def get_formatted_speed(s,bytes=False):
            unit = ""
            if s > 1024**3:
                s = s / 1024**3
                unit = "G"
            elif s > 1024**2:
                s = s / 1024**2
                unit = "M"
            elif s > 1024:
                s = s / 1024
                unit = "K"
            if bytes:
                return f"{(s/8):.2f} {unit}iB/s"
            return f"{s:.2f} {unit}ib/s"
            


        parser = argparse.ArgumentParser()
        parser.add_argument('--upload', action="store_true")
        parser.add_argument('--bytes', action="store_true")
        args= parser.parse_args()

        try:
            s = speedtest.Speedtest()
        except:
            exit(0)

        if args.upload:
            s.upload(pre_allocate=False)
            print("▲ " + get_formatted_speed(s.results.upload,args.bytes))
        else:
            s.download()
            print("▼ " + get_formatted_speed(s.results.download,args.bytes))
        ```

### 3.6 qq
`yay -S linuxqq`

???+ "每次qq接收消息都会闪退问题"

    - 环境：archlinux+i3wm
    - 问题：每次qq接收消息都会闪退
    - 解决：
        - 用终端输入linuxqq运行，闪退时查看运行log（鼠标滚轮滚了几十秒才看到在开头报了error）
        - 报错信息：[29188:1027/223745.914909:ERROR:libnotify_notification.cc(49)] notify_notification_show: domain=2117 code=2 message="GDBus.Error:org.freedesktop.DBus.Error.ServiceUnknown: The name org.freedesktop.Notifications was not provided by any .service files"
        - 谷歌搜这个报错，找到[https://wiki.archlinux.org/title/Desktop_notifications#Standalone](https://wiki.archlinux.org/title/Desktop_notifications#Standalone){target=_blank}，完美解决

    ``` shell
    # notification-daemon一个original的通知组件
    sudo pacman -S notification-daemon
    vim /usr/share/dbus-1/services/org.freedesktop.Notifications.service
    ```
    ``` shell
    [D-BUS Service]
    Name=org.freedesktop.Notifications
    Exec=/usr/lib/notification-daemon-1.0/notification-daemon
    ```

### 3.7 浏览器
- `sudo pacman -S brave firefox firefox-extension-arch-search`

### 3.8 vscode
- `yay -S visual-studio-code-bin`

???+ "settings.json"

    ``` json
    {
        "workbench.colorTheme": "Default Dark Modern",
        "editor.fontSize": 24,
        "window.zoomLevel": 1,
        "files.autoSave": "onFocusChange"
    }
    ```

### 3.9 深度图片查看器、截图、文件管理器
- `sudo pacman -S deepin-image-viewer deepin-screenshot deepin-file-manager ranger flameshot`

还是thunar好用

- `sudo pacman -S thunar thunar-archive-plugin thunar-media-tags-plugin thunar-volman`
- `yay -S thunar-shares-plugin`

### 3.10 docker
- `sudo pacman -S docker docker-compose`
- `sudo systemctl enable docker`
- `sudo systemctl start docker`
- `usermod -aG docker shafish`
- `sudo systemctl restart docker`

### 3.11 stable-diffusion
- [https://github.com/pyenv/pyenv#set-up-your-shell-environment-for-pyenv](https://github.com/pyenv/pyenv#set-up-your-shell-environment-for-pyenv){target=_blank}
- [https://github.com/pyenv/pyenv-virtualenv#installing-as-a-pyenv-plugin](https://github.com/pyenv/pyenv-virtualenv#installing-as-a-pyenv-plugin){target=_blank}
- [https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki){target=_blank}

???+ "当然是拿来搞stable-diffusion"

    - `sudo pacman -S pyenv` `yay -S pyenv-virtualenv`
        - 配置fish环境：`vim ~/.config/fish/config.fish`
        ``` shell
        pyenv init - | source
        status --is-interactive; and pyenv virtualenv-init - | source
        ```
        - 配置zsh环境：
        ``` shell
        echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
        echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
        echo 'eval "$(pyenv init -)"' >> ~/.zshrc
        echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
        ```
    - `pyenv install -v 3.10.6`
    - `pyenv virtualenv 3.10.6 sdwebui`
    - `cd ~/Project/Python`
    - `git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git`
    - `cd stable-diffusion-webui`
    - `pyenv local sdwebui`
    - `python -V`
    - `./webui.sh --xformers` # 自行配置代理

    ``` zsh
    proxysd () {
    export no_proxy="localhost, 127.0.0.1, ::1"
    export http_proxy="http://127.0.0.1:10001"
    export https_proxy=$http_proxy
    echo "http proxy 10 on"
    }

    noproxysd () {
    unset http_proxy
    unset https_proxy
    unset no_proxy
    }
    ```

### 3.12 轻量截图工具

- `sudo pacman -S maim xclip`

### 3.13 idea

- 官网下载

### 3.14 java

- 官网下载

### 3.15 数据库管理
`sudo pacman -S dbeaver`

### 3.15 手机投屏
手机启动无线调试

``` shell
sudo pacman -S scrcpy adb
adb tcpip 5555
adb connect 192.168.0.5:5555
```

### 3.16 git客户端
- git config --global user.email "xxxx"
- git config --global user.name "xxxx"

``` shell
sudo pacman -S lazygit
```

### 3.17 goldendict
[https://shafish.cn/english/#%E4%B8%80goldendict%E9%85%8D%E7%BD%AE](https://shafish.cn/english/#%E4%B8%80goldendict%E9%85%8D%E7%BD%AE){target=_blank}


``` shell
sudo pacman -S dictd
yay -S goldendict-ng dict-gcide aspeak 
sudo systemctl enable dictd.service
sudo systemctl start dictd.service
```

### 3.18 图片压缩
`sudo pacman -S pngquant zopfli`

``` shell
#!/bin/sh

# Use pngquant and zopflipng to compress png images
# The final result is similar to tinypng.com

file_seed="$(date +%s)"
pngquant --speed 1 --strip --verbose $1 -o temp_$file_seed.png
echo "========= zopflipng compressing ========= "
zopflipng -m --lossy_transparent temp_$file_seed.png compressed_$file_seed.png
echo "Deleting temp file..."
rm temp_$file_seed.png
echo "Done."
```

### 3.19 samba挂载
- `sudo pacman -S cifs-utils`

???+ "配置"
    - 手动加载模块：`modprobe cifs`
    - 挂载硬盘：`sudo mount.cifs //192.168.2.143/data /mnt/smb_dsm_data -o user=user,pass=passwd,uid=1000,gid=1000,sec=ntlmssp --verbose`

    https://www.scarletdrop.cn/archives/gGhbQxpO
    https://www.cnblogs.com/yudai/p/16326964.html

### 3.20 vlc
- `sudo pacman -S vlc`

???+ "配置"
    - 中文配置
    ``` shell title="/usr/share/applications/vlc.desktop"
    将文件中 `Exec=/usr/bin/vlc --started-from-file %U` 改为 `Exec=env LANGUAGE=zh_CN /usr/bin/vlc --started-from-file %U`
    ```
    - vlc web
        - 添加vlc用户：`useradd -c "VLC daemon" -d / -G audio -M -p \! -r -s /usr/bin/nologin -U vlcd`
        - 配置服务
        ``` shell title="/etc/systemd/system/vlc.service"
        [Unit]
        Description=VideoOnLAN Service
        After=network.target

        [Service]
        Type=forking
        User=vlcd
        ExecStart=/usr/bin/vlc --daemon --syslog -I http --http-port 8090 --http-password password 
        Restart=on-abort

        [Install]
        WantedBy=multi-user.target
        ```
        - 浏览器打开`http://127.0.0.1:8090`，输入密码即可
    
### 3.21 whisper

- `sudo pacman -S pyenv`
- `yay -S pyenv-virtualenv`

???+ "配置"
    - 配置fish环境：`vim ~/.config/fish/config.fish`
    ``` shell
    pyenv init - | source
    status --is-interactive; and pyenv virtualenv-init - | source
    ```
    - 配置zsh环境：
    ``` shell
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
    echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
    echo 'eval "$(pyenv init -)"' >> ~/.zshrc
    echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
    ```
    - `pyenv install -v 3.10.6`
    - `pyenv virtualenv 3.10.6 whisper`
    - `cd ~/Project/Python`
    - `mkdir whisper && cd whisper`
    - `pyenv local whisper`
    - `python -V`
    - `pip install git+https://github.com/openai/whisper.git`
    - 提取音频：`ffmpeg -i xxx.mp4 -vn -acodec pcm_s16le -ar 44100 -ac 2 xxx.wav`
        - -i input_file.mp4：指定输入文件的路径。
        - -vn：禁用视频输出。
        - -acodec pcm_s16le：指定音频编码格式为 PCM 16-bit 线性。
        - -ar 44100：指定采样率为 44.1 kHz。
        - -ac 2：指定声道数为 2。

    - whisper使用：`whisper xxx.wav --language Japanese --model medium`

### 3.22 远程桌面

> 主要远程协议是 `vnc` 和 `rdp` ，对应实现有 `x11vnc` 和 `xrdp`。rdp的话window使用的多（本次没安装使用），x11vnc内网连接切换页面都感觉卡卡的，配置 `-ncache 10` 还提示控制端屏幕分辨率要一致。

> 最后还是选了 `noMachine`，然后用 `frp` 穿透本地的4000端口。（不想折腾可以一步到位直接安装向日葵）

???- "x11vnc"

    ???+ "被控端server配置"

        - 安装：`sudo pacman -S x11vnc`
        - 设置密码：`x11vnc -storepasswd` `sudo mv ~/.vnc/passwd /etc/x11vnc.pwd `
        - 创建服务（sddm）：
        ``` shell title="/etc/systemd/system/x11vnc.service"
        [Unit]
        Description=Remote desktop service (VNC)
        Requires=display-manager.service
        After=display-manager.service

        [Service]
        ExecStart=
        ExecStart=/bin/bash -c "/usr/bin/x11vnc -auth /var/run/sddm/* -display :0 -forever -loop -noxdamage -repeat -rfbauth /etc/x11vnc.pwd -shared"

        [Install]
        WantedBy=graphical.target
        ```
        - 刷新服务：`sudo systemctl daemon-reload`
        - 服务启动：`sudo systemctl restart x11vnc`
        - 服务状态：`sudo systemctl status x11vnc`
        - 服务自启动：`sudo systemctl enable x11vnc`

        ref: [man x11vnc](https://man.archlinux.org/man/x11vnc.1){target=_blank}  [wiki x11vnc](https://wiki.archlinux.org/title/x11vnc#){target=_blank}

    ???+ "vnc客户端（三选一）"
        - [realvnc](https://www.realvnc.com/en/connect/download/viewer/){target=_blank}
        - `sudo pacman -S tigervnc`
        - `sudo pacman -S remmina`

    ???+ "远程桌面"
        - `frp`穿透5900端口即可

    ???+ "命令参数"
        - `rfbauth`：配置vnc访问密码
        - `rfbport`：VNC 端口
        - `noipv6`：禁ipv6
        - `shared`：屏幕共享
        - `forever`：客户端断开连接后，继续监听连接
        - `loop`：当 x11vnc 进程终止时，重新启动


???- "noMachine（推荐）"

    - `yay -s nomachine`
    - `sudo systemctl start nxserver.service`
    - `sudo systemctl enable nxserver.service`
    - `frp`穿透4000端口即可

???- "frp"

    - [fatedier-frp-github](https://github.com/fatedier/frp){target=_blank}
    - [frp-doc](https://gofrp.org/zh-cn/docs/overview/){target=_blank}

    ???- "frp安装（服务端、客户端都要下载和配置）"

        - `cd /opt`
        - `sudo wget https://github.com/fatedier/frp/releases/download/v0.53.2/frp_0.53.2_linux_amd64.tar.gz`
        - `sudo tar -zxvf frp_0.53.2_linux_amd64.tar.gz && cd frp_0.53.2_linux_amd64`

    ???- "frp服务端配置"

        ``` ini title="frps.ini（已不建议使用，这里仅与下面的toml配置做对比参考）"
        [common]
        bind_port = 7000
        token = 2xxxxxxxxxxxJ
        dashboard_port = 7500
        dashboard_user = sxxxxxh
        dashboard_pwd = fxxxxxxxxxxx8
        vhost_http_port = 7508
        vhost_https_port = 7509
        subdomain_host = sxxxxxh.cn
        enable_prometheus = true
        log_file = /var/log/frps.log
        log_level = info
        log_max_days = 3
        ```
        ``` toml title="/opt/frp_0.53.2_linux_amd64/frps.toml"
        bindPort = 7000
        # QUIC
        quicBindPort = 7000
        auth.token = "2xxxxxxxxxxxJ"
        webServer.addr = "0.0.0.0"
        webServer.port = 7500
        webServer.user = "sxxxxxh"
        webServer.password = "fxxxxxxxxxxx8"
        # webServer.tls.certFile = "server.crt"
        # webServer.tls.keyFile = "server.key"
        vhostHTTPPort = 7508
        # vhostHTTPTimeout = 60
        vhostHTTPSPort = 7509
        subDomainHost = "sxxxxxh.cn"
        # enablePrometheus = true
        log.to = "/var/log/frps.log"
        log.level = "info"
        log.maxDays = 7
        ```

        - 命令启动：`/opt/frp_0.53.2_linux_amd64/frps -c /opt/frp_0.53.2_linux_amd64/frps.toml`

        - 服务自启动脚本：

        ``` shell title="/etc/systemd/system/frps.service"
        [Unit]
        # 服务名称，可自定义
        Description = frp server
        After = network.target syslog.target
        Wants = network.target

        [Service]
        Type = simple
        # 启动frps的命令，需修改为您的frps的安装路径
        ExecStart = /opt/frp_0.53.2_linux_amd64/frps -c /opt/frp_0.53.2_linux_amd64/frps.toml

        [Install]
        WantedBy = multi-user.target
        ```

        - 服务启动：`sudo systemctl restart frps`
        - 服务自启动：`sudo systemctl enable frps`

    配置格式检查：`frps verify -c ./frps.toml`

    ???- "服务器开端口"

        - 查询端口是否开启：firewall-cmd --zone=public --query-port=7007/tcp
        - 开启某个端口：firewall-cmd --zone=public --add-port=7000/tcp --permanent
        - 关闭某个端口：firewall-cmd --zone=public --remove-port=7000/tcp --permanent
        - 重启防火墙使配置生效：firewall-cmd --reload
        - 阿里云/腾讯云控制台-》服务器-》安全组 开放端口

    ???- "frp客户端配置（被控端）"

        ``` toml title="/opt/frp_0.53.2_linux_amd64/frpc.toml"
        serverAddr = "frps服务器ip"
        serverPort = 7000
        auth.token = "2xxxxxxxxxxxJ"
        user = "product1"
        includes = ["./confd/*.toml"]
        log.to = "/var/log/frpc.log"
        log.level = "info"
        log.maxDays = 7
        ```
        ``` toml title="/opt/frp_0.53.2_linux_amd64/confd/ssh.toml"
        [[proxies]]
        name = "ssh"
        type = "stcp"
        secretKey = "xxxxxtoken"
        localIP = "127.0.0.1"
        localPort = 22
        ```
        ``` shell title="/etc/systemd/system/frpc.service"
        [Unit]
        # 服务名称，可自定义
        Description = frp client
        After = network.target syslog.target
        Wants = network.target

        [Service]
        Type = simple
        TimeoutStartSec = 30
        Restart = on-failure
        RestartSec = 5
        # 启动frps的命令，需修改为您的frps的安装路径
        ExecStart = /opt/frp_0.53.2_linux_amd64/frpc -c /opt/frp_0.53.2_linux_amd64/frpc.toml
        ExecStop = /bin/kill $MAINPID

        [Install]
        WantedBy = multi-user.target
        ```

    ???- "frp客户端配置（控制端）"

        ``` toml title="/opt/frp_0.53.2_linux_amd64/frpc.toml"
        serverAddr = "frps服务器ip"
        serverPort = 7000
        auth.token = "2xxxxxxxxxxxJ"
        includes = ["./confd/*.toml"]
        log.to = "/var/log/frpc.log"
        log.level = "info"
        log.maxDays = 7
        ```
        ``` toml title="/opt/frp_0.53.2_linux_amd64/confd/ssh.toml"
        [[visitors]]
        name = "ssh_visitors"
        type = "stcp"
        serverUser = "product1"
        serverName = "ssh"
        secretKey = "xxxxxtoken"
        bindAddr = "0.0.0.0"
        bindPort = 2222
        ```
        ``` shell title="/etc/systemd/system/frpc.service"
        [Unit]
        # 服务名称，可自定义
        Description = frp client
        After = network.target syslog.target
        Wants = network.target

        [Service]
        Type = simple
        TimeoutStartSec = 30
        Restart = on-failure
        RestartSec = 5
        # 启动frps的命令，需修改为您的frps的安装路径
        ExecStart = /opt/frp_0.53.2_linux_amd64/frpc -c /opt/frp_0.53.2_linux_amd64/frpc.toml
        ExecStop = /bin/kill $MAINPID

        [Install]
        WantedBy = multi-user.target
        ```

???- "向日葵（也推荐）"

    - `yay -S sunloginclient`
    - `sudo systemctl start runsunloginclient.service`
    - `sudo systemctl enable runsunloginclient.service`

### 3.23 timeshift
系统备份，滚挂必备

- `sudo pacman -S timeshift`
- `sudo timeshift-gtk`

ref: [使用参考](https://aprilzz.com/archives/%E5%9C%A8arch%E4%B8%AD%E4%BD%BF%E7%94%A8timeshift%E4%BF%9D%E7%B3%BB%E7%BB%9F%E5%B9%B3%E5%AE%89){target=_blank}

### 3.24 smaba
ref: [https://wiki.archlinuxcn.org/wiki/Samba](https://wiki.archlinuxcn.org/wiki/Samba){target=_blank}

`sudo pacman -S samba avahi`

???- "配置"
    ``` conf title="/etc/samba/smb.conf"
    # This is the main Samba configuration file. You should read the
    # smb.conf(5) manual page in order to understand the options listed
    # here. Samba has a huge number of configurable options (perhaps too
    # many!) most of which are not shown in this example
    #
    # For a step to step guide on installing, configuring and using samba,
    # read the Samba-HOWTO-Collection. This may be obtained from:
    #  http://www.samba.org/samba/docs/Samba-HOWTO-Collection.pdf
    #
    # Many working examples of smb.conf files can be found in the
    # Samba-Guide which is generated daily and can be downloaded from:
    #  http://www.samba.org/samba/docs/Samba-Guide.pdf
    #
    # Any line which starts with a ; (semi-colon) or a # (hash)
    # is a comment and is ignored. In this example we will use a #
    # for commentry and a ; for parts of the config file that you
    # may wish to enable
    #
    # NOTE: Whenever you modify this file you should run the command "testparm"
    # to check that you have not made any basic syntactic errors.
    #
    #======================= Global Settings =====================================
    [global]

    # workgroup = NT-Domain-Name or Workgroup-Name, eg: MIDEARTH
    workgroup = MYGROUP

    # server string is the equivalent of the NT Description field
    server string = Samba Server

    # Server role. Defines in which mode Samba will operate. Possible
    # values are "standalone server", "member server", "classic primary
    # domain controller", "classic backup domain controller", "active
    # directory domain controller".
    #
    # Most people will want "standalone server" or "member server".
    # Running as "active directory domain controller" will require first
    # running "samba-tool domain provision" to wipe databases and create a
    # new domain.
    server role = standalone server

    # This option is important for security. It allows you to restrict
    # connections to machines which are on your local network. The
    # following example restricts access to two C class networks and
    # the "loopback" interface. For more examples of the syntax see
    # the smb.conf man page
    ;   hosts allow = 192.168.1. 192.168.2. 127.

    # Uncomment this if you want a guest account, you must add this to /etc/passwd
    # otherwise the user "nobody" is used
    ;  guest account = pcguest

    # this tells Samba to use a separate log file for each machine
    # that connects
    #   log file = /usr/local/samba/var/log.%m
    log file = /var/log/samba/%m.log

    # Put a capping on the size of the log files (in Kb).
    max log size = 50

    # Specifies the Kerberos or Active Directory realm the host is part of
    ;   realm = MY_REALM

    # Backend to store user information in. New installations should
    # use either tdbsam or ldapsam. smbpasswd is available for backwards
    # compatibility. tdbsam requires no further configuration.
    ;   passdb backend = tdbsam

    # Using the following line enables you to customise your configuration
    # on a per machine basis. The %m gets replaced with the netbios name
    # of the machine that is connecting.
    # Note: Consider carefully the location in the configuration file of
    #       this line.  The included file is read at that point.
    ;   include = /usr/local/samba/lib/smb.conf.%m

    # Configure Samba to use multiple interfaces
    # If you have multiple network interfaces then you must list them
    # here. See the man page for details.
    ;   interfaces = 192.168.12.2/24 192.168.13.2/24

    # Where to store roving profiles (only for Win95 and WinNT)
    #        %L substitutes for this servers netbios name, %U is username
    #        You must uncomment the [Profiles] share below
    ;   logon path = \\%L\Profiles\%U

    # Windows Internet Name Serving Support Section:
    # WINS Support - Tells the NMBD component of Samba to enable it's WINS Server
    ;   wins support = yes

    # WINS Server - Tells the NMBD components of Samba to be a WINS Client
    #	Note: Samba can be either a WINS Server, or a WINS Client, but NOT both
    ;   wins server = w.x.y.z

    # WINS Proxy - Tells Samba to answer name resolution queries on
    # behalf of a non WINS capable client, for this to work there must be
    # at least one	WINS Server on the network. The default is NO.
    ;   wins proxy = yes

    # DNS Proxy - tells Samba whether or not to try to resolve NetBIOS names
    # via DNS nslookups. The default is NO.
    dns proxy = no

    # These scripts are used on a domain controller or stand-alone
    # machine to add or delete corresponding unix accounts
    ;  add user script = /usr/sbin/useradd %u
    ;  add group script = /usr/sbin/groupadd %g
    ;  add machine script = /usr/sbin/adduser -n -g machines -c Machine -d /dev/null -s /bin/false %u
    ;  delete user script = /usr/sbin/userdel %u
    ;  delete user from group script = /usr/sbin/deluser %u %g
    ;  delete group script = /usr/sbin/groupdel %g


    #============================ Share Definitions ==============================
    [homes]
    comment = Home Directories
    browsable = no
    writable = yes

    # Un-comment the following and create the netlogon directory for Domain Logons
    ; [netlogon]
    ;   comment = Network Logon Service
    ;   path = /usr/local/samba/lib/netlogon
    ;   guest ok = yes
    ;   writable = no
    ;   share modes = no


    # Un-comment the following to provide a specific roving profile share
    # the default is to use the user's home directory
    ;[Profiles]
    ;    path = /usr/local/samba/profiles
    ;    browsable = no
    ;    guest ok = yes


    # NOTE: If you have a BSD-style print system there is no need to
    # specifically define each individual printer
    [printers]
    comment = All Printers
    path = /usr/spool/samba
    browsable = no
    # Change 'guest ok' from 'no' to 'yes' to allow the 'guest account' user to print
    guest ok = no
    writable = no
    printable = yes

    # This one is useful for people to share files
    ;[tmp]
    ;   comment = Temporary file space
    ;   path = /tmp
    ;   read only = no
    ;   public = yes

    # A publicly accessible directory, but read only, except for people in
    # the "staff" group
    ;[public]
    ;   comment = Public Stuff
    ;   path = /home/samba
    ;   public = yes
    ;   writable = no
    ;   printable = no
    ;   write list = @staff

    # Other examples.
    #
    # A private printer, usable only by fred. Spool data will be placed in fred's
    # home directory. Note that fred must have write access to the spool directory,
    # wherever it is.
    ;[fredsprn]
    ;   comment = Fred's Printer
    ;   valid users = fred
    ;   path = /homes/fred
    ;   printer = freds_printer
    ;   public = no
    ;   writable = no
    ;   printable = yes

    # A private directory, usable only by fred. Note that fred requires write
    # access to the directory.
    ;[fredsdir]
    ;   comment = Fred's Service
    ;   path = /usr/somewhere/private
    ;   valid users = fred
    ;   public = no
    ;   writable = yes
    ;   printable = no

    # a service which has a different directory for each machine that connects
    # this allows you to tailor configurations to incoming machines. You could
    # also use the %U option to tailor it by user name.
    # The %m gets replaced with the machine name that is connecting.
    ;[pchome]
    ;  comment = PC Directories
    ;  path = /usr/pc/%m
    ;  public = no
    ;  writable = yes

    # A publicly accessible directory, read/write to all users. Note that all files
    # created in the directory by users will be owned by the default user, so
    # any user with access can delete any other user's files. Obviously this
    # directory must be writable by the default user. Another user could of course
    # be specified, in which case all files would be owned by that user instead.
    ;[public]
    ;   path = /usr/somewhere/else/public
    ;   public = yes
    ;   only guest = yes
    ;   writable = yes
    ;   printable = no

    # The following two entries demonstrate how to share a directory so that two
    # users can place files there that will be owned by the specific users. In this
    # setup, the directory should be writable by both users and should have the
    # sticky bit set on it to prevent abuse. Obviously this could be extended to
    # as many users as required.
    ;[myshare]
    ;   comment = Mary's and Fred's stuff
    ;   path = /usr/somewhere/shared
    ;   valid users = mary fred
    ;   public = no
    ;   writable = yes
    ;   printable = no
    ;   create mask = 0765
    ```

``` shell
sudo systemctl start avahi-daemon.service
sudo systemctl enable avahi-daemon.service

sudo smbpasswd -a samba_user
sudo systemctl start smb.service 
sudo systemctl start nmb.service
sudo systemctl enable smb.service 
sudo systemctl enable nmb.service
```

### 3.25 OBS

``` shell
sudo pacman -S obs-studio
# ndi双机推流
sudo pacman -S obs-ndi
```

### 3.26 图片编辑软件

`sudo pacman -S gimp`

inkscape
sudo pacman -S krita python-pyqt5 qt5-imageformats

### 3.27 思维导图 `mind-map`

``` yml
version: '3'

services:
  smm:
    image: shuiche/mind-map
    container_name: simplemindmap
    restart: unless-stopped
    ports:
      - 8001:8080
```

### 3.28 编程交互环境 Jupyter
https://wiki.archlinux.org/title/Jupyter

```shell
sudo pacman -S jupyterlab 
```

### 3.29 数据库管理工具

```shell
yay -S beekeeper-studio-bin
```

### 3.30 系统盘制作

``` shell
yay -S ventoy
sudo ventoyweb
```

### 3.31 游戏管理

``` shell
sudo pacman -S lutris
```
- 安装游戏时无法创建目录问题解决（甜菜）：
https://www.reddit.com/r/Lutris/comments/181ovp6/not_able_to_create_prefix_folders_anymore_steam/

新建目录中创建 xxx.txt

- 运行wine提示 `no such file or directory：xxx/wine` 解决：

安装 lib32-glibc

- 提示Wine 找不到 FreeType 字体库 解决：

本来就安装好的，重装wine

### 3.32 无损放大

``` shell
yay -S upscayl-bin
```

### 3.33 动捕、虚拟

inochi-creator inochi-session

https://lunafoxgirlvt.itch.io/inochi-creator

### 3.34 音乐

`sudo pacman -S elisa`
`yay -S moosync`

### 3.35 视频剪辑

`sudo pacman -S shotcut`

4k屏幕下字体调整：`shotcut --QT_SCALE_FACTOR 1.5`

### 3.36 画板

https://wiki.archlinux.org/title/Graphics_tablet

``` shell
# 检查按板按钮id
xev -event button
1   2
3   8
9   10

11  12
13  14
15  16
```

```
// 画笔
xsetwacom --set "Gaomon Gaomon Tablet_1060Pro Pad pad" Button 1 "key b"
// 
```

#!/bin/sh
xsetwacom –set “HUION Huion Tablet Pad pad” Button 1 key +ctrl +z -z -ctrl
xsetwacom –set “HUION Huion Tablet Pad pad” Button 2 key +shift +e
xsetwacom –set “HUION Huion Tablet Pad pad” Button 3 key +p
xsetwacom –set “HUION Huion Tablet Pad pad” Button 8 key +shift + =
xsetwacom –set “HUION Huion Tablet Pad pad” Button 9 key + –
xsetwacom –set “HUION Huion Tablet Pad pad” Button 10 key + ]
xsetwacom –set “HUION Huion Tablet Pad pad” Button 11 key + [
xsetwacom –set “HUION Huion Tablet Pad pad” Button 12 key + m


### 3.37 画图笔记

`sudo pacman -S xournalpp`

### 3.38 音乐制作

`yay -S bitwig-studio`

### 3.39 面捕

``` 
pyenv virtualenv 3.10.6 openseeface
git clone https://github.com/emilianavt/OpenSeeFace
cd OpenSeeFace
pyenv local openseeface
pip3 install onnxruntime opencv-python pillow numpy
# 启动
python facetracker.py -c 0 -W 1280 -H 720 --discard-after 0 --scan-every 0 --no-3d-adapt 1 --max-feature-updates 900

python facetracker.py --ip 127.0.0.1 --port 11573 -c 0 -W 1280 -H 720 --discard-after 0 --scan-every 0 --no-3d-adapt 1 --max-feature-updates 900 --fps 30
```
openseeface + inochi session
或者直接用 Puppetstring（推荐）

### 3.40 视频粗剪

`yay -S losslesscut-bin`

### 3.41 系统提示命令

``` shell
notify-send -t 10000 -i '/home/shafish/Pictures/icon/2.png' 'HARDWORK' '工作一个半小时，站起来走走吧。'

zenity --warning --text='工作一个半小时，站起来走走吧。'
```

## 四、问题解决
### 4.1 开机启动失败

???+ "啊，废了？？？？？"

    #### 4.1.1 问题描述：
    - 更新系统到`Linux 6.6.8-zen1-1-zen`，系统关机后重新启动
    - 开机启动时提示网络硬盘挂载失败，进入tty（ctrl+alt+f2）注释掉`/etc/fstab`对应挂载，重新启动
    - 启动一直卡在`/dev/bla: clean, xxx/xxx files, xxx/xxx blocks`

    #### 4.1.2 解决：
    1. 以为系统在检查，啥也没干预，直接放置了一个夜晚。（然并卵）
    2. 网上有说是显卡驱动问题，要执行相关命令卸载：`mhwd -l`，然而提示mhwd命令都找不到。（五二人子弟）
    3. 网上有说是 [Silent boot](https://wiki.archlinux.org/title/Silent_boot#fsck){target=_blank} 设置问题，设置完成就会跳过检查，也并未解决。
    4. 最终在 [Booting to black screen](https://forum.manjaro.org/t/boot-doesnt-continue/80873/7){target=_blank} 找到解决方法：
        tty后执行`pacman -Syu grub`就行，执行完再关机重启。（解决）

    5. 在网络硬盘挂载时加上`_netdev`参数：`//192.168.0.xxx/xxx /mnt/xxx cifs defaults,user=xxx,password=xxx,_netdev,vers=3.0 0 0`

    #### 4.1.3 记录
    很有可能是挂载网络硬盘引起的磁盘检查卡顿 [Fsck](https://wiki.archlinux.org/title/Fsck){target=_blank}

待续

*[BalenaEtcher]: Linux系统下建议的写盘工具
*[Rufus]: Window系统下建议的写盘工具
*[Archlinux镜像]: 选China下载
*[Silent boot]: 设置完后，下次正常开机确实很快