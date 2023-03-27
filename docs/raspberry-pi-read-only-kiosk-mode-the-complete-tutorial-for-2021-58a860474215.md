# Raspberry Pi 只读信息亭模式:2021 完整教程

> 原文：<https://itnext.io/raspberry-pi-read-only-kiosk-mode-the-complete-tutorial-for-2021-58a860474215?source=collection_archive---------0----------------------->

## 包括网络设置软件和远程控制

![](img/9b68e00a5106b78820cd9fa0c302c3bb.png)

这篇文章，这篇教程，是因为一个简单的原因而写的:我在其他地方找不到类似的东西。

**更新:这个教程有了新版本**

[](https://petrjahoda.medium.com/raspberry-pi-read-only-kiosk-mode-2022-complete-tutorial-df7fc051fdaf) [## Raspberry Pi 只读信息亭模式:2022 完整教程

### 包括网络设置软件和远程控制

petrjahoda.medium.com](https://petrjahoda.medium.com/raspberry-pi-read-only-kiosk-mode-2022-complete-tutorial-df7fc051fdaf) 

我想要一套完整的说明，如何改变一个新的 unboxed Raspberry Pi 变成一个设备，可以在后台使用触摸屏的其他设备。

我想让结果看起来像你在下面看到的那样(捷克语版)。一个带有触摸显示屏和树莓派的设备。我想要一个平稳的启动进入一些“设置”屏幕，在那里我可以改变网络和设置一个服务器地址，这将自动加载，如果一切正常。在这种情况下，服务器地址是一个包含最终软件的网页。像一些工厂的软件(像[这种](https://github.com/petrjahoda/sklabel_cutting_webservice))、银行、停车场等等。

![](img/ec6ad29a5aeff2d1fd488465f5913dc6.png)

本教程是一个完整的教程，你可以使用和改变你想要的。本教程是一个工作教程(截至 2021 年 4 月)，基于他人的工作。我只是把它们结合在一起。因为我结合了互联网上不同的东西，很多时候我不确定谁是真正的原作者，所以我不能写下每篇教程的每个作者。如果你是一个作者，并且正在阅读这篇文章，请随时联系我，我会很乐意更新这篇文章。

我还增加了一些功能，比如远程重启、远程关机、远程截图等。因此，用户可以创建网页并管理所有连接的设备。

在该过程中使用了树莓 Pi 4b。

亲爱的读者，如果你发现一些事情没有按预期工作，请与我联系，我们将努力找到解决问题的办法，并更新文章，使之更加准确。

为了使它完整，这里是一个最终结果的照片，显示了带有触摸键盘的“设置屏幕”(捷克版本)。

![](img/5336d3de31c9ba28b4fe31dd2f2754c1.png)

后端 web 服务是用 Go 编写的(前端是普通的 html+css+js ),使用本教程:

[](https://medium.com/swlh/create-go-service-the-easy-way-de827d7f07cf) [## 以简单的方式创建 Go 服务

### 适用于 Windows、Linux、MacOS 和 Docker

medium.com](https://medium.com/swlh/create-go-service-the-easy-way-de827d7f07cf) 

源代码可以在这里找到:

[](https://github.com/petrjahoda/rpi_kiosk_webservice) [## petrjahoda/rpi _ kiosk _ web service

### 通过在 GitHub 上创建一个帐户，为 petrjahoda/rpi _ kiosk _ web service 开发做出贡献。

github.com](https://github.com/petrjahoda/rpi_kiosk_webservice) 

# 1.准备树莓酱

*   从[官方网站](https://www.raspberrypi.org/software/)使用 raspbian Pi 成像仪安装 raspbian lite
*   使用`sudo apt-get update && sudo apt-get upgrade`进行更新和升级
*   使用`sudo raspi-config`:启用控制台自动登录、启用 ssh、启用过扫描(或禁用欠扫描)
*   使用`sudo apt-get install maim`安装 maim
*   使用`sudo apt-get install network-manager`安装网络管理器
*   使用`sudo systemctl enable NetworkManager`启用网络管理器作为服务
*   使用`sudo systemctl start NetworkManager`将网络管理器作为服务启动
*   使用`sudo systemctl mask dhcpcd`禁用旧的 dhcp 服务
*   使用`sudo apt-get install ufw`安装 ufw
*   使用`sudo ufw allow 9999`启用端口 9999
*   使用`sudo reboot now`重启

# 2.在 kiosk 模式下安装 Chromium

*   使用`sudo apt-get install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox`安装先决条件
*   使用`sudo apt-get install --no-install-recommends chromium-browser`安装铬合金
*   使用`sudo nano /etc/xdg/openbox/autostart`编辑自动启动文件，插入以下行:

```
# Disable any form of screen saver / screen blanking / power management
xset s off
xset s noblank
xset -dpms# Allow quitting the X server with CTRL-ATL-Backspace
setxkbmap -option terminate:ctrl_alt_bksp# Start Chromium in kiosk mode
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences
chromium-browser --start-fullscreen --kiosk --incognito --noerrdialogs --disable-translate --no-first-run --fast --fast-start --disable-infobars --disable-features=TranslateUI --disk-cache-dir=/dev/null  --password-store=basic --disable-pinch --overscroll-history-navigation=disabled --disable-features=TouchpadOverscrollHistoryNavigation 'http://localhost:9999'
```

***附加信息:****chromium-browser 有一个附加标志:* `*temporary-unexpire-flags-m80*` *。这个标志从 2021 年 6 月 20 日起从 chromium 中删除，所以上面的命令被更新。感谢 Paul den Hertog 提供的信息。有一种可能性是，当你将来阅读这篇文章时，有些标志是不必要的。*

*   使用`sudo nano .bash_profile`使一切在引导时启动，插入以下行:

```
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor
```

> *提示:按下* `*Ctrl-Alt-Backspace*` *可以杀死 chromium 并进入命令行*

# 3.将程序数据复制到 Raspberry

*   将 rpi_linux 复制到/home/pi 中
*   将/config/*复制到/home/pi/*中
*   将/html/*复制到/home/pi/*中
*   将/css/*复制到/home/pi/*中
*   将/js/*复制到/home/pi/*中
*   将/font/*复制到/home/font/*中

> *示例:使用 scp 复制 rpi 程序文件:* `*scp rpi/rpi_linux pi@<ipaddress>:/home/pi*` *>*
> 
> *示例:使用 scp 复制所有目录:* `*scp -r config html css js font pi@<ipaddress>:/home/pi*`

# 4.使程序作为服务运行

*   使用`sudo nano /lib/systemd/system/zapsi.service`创建新文件，插入以下行:

```
[Unit]
Description=Zapsi Service
ConditionPathExists=/home/pi/rpi_linux
After=network.target
[Service]
Type=simple
User=root
Group=root
LimitNOFILE=1024
Restart=on-failure
RestartSec=10
startLimitIntervalSec=60
WorkingDirectory=/home/pi
ExecStart=/home/pi/rpi_linux
PermissionsStartOnly=true
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=zapsi_service
[Install]
WantedBy=multi-user.target
```

*   使用`sudo chmod 755 /lib/systemd/system/zapsi.service`确保文件可执行
*   使用`sudo systemctl enable zapsi.service`使服务自动启动
*   使用`sudo systemctl start zapsi.service`立即启动服务

> *提示:使用*和`*journalctl -f -u zapsi.service*`搜索日志

# 5.干净的启动屏幕和信息

*   使用`sudo nano /boot/config.txt`禁用启动彩虹:添加线`disable_splash=1`
*   使用`sudo nano /boot/cmdline.txt`禁用引导信息:在第一行`silent quiet splash loglevel=0 logo.nologo vt.global_cursor_default=0`的末尾添加，用`console=tty3`替换`console=tty1`
*   禁用引导自动登录终端信息:运行`touch ~/.hushlogin`，运行`sudo nano /etc/systemd/system/getty@tty1.service.d/autologin.conf`，用`ExecStart=-/sbin/agetty --skip-login --noclear --noissue --login-options "-f pi" %I $TERM`替换`ExecStart=-/sbin/agetty --autologin pi --noclear %I xterm-256color`行

# 6.将 Raspberry Pi 设为只读

*   移除交换

```
sudo dphys-swapfile swapoff
sudo dphys-swapfile uninstall
sudo update-rc.d dphys-swapfile remove
```

*   更新和升级一切

```
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get upgrade
sudo BRANCH=next rpi-update
```

*   使用`sudo reboot now`重启
*   使用`sudo mkinitramfs -o /boot/initrd`创建 initramfs
*   使用`sudo curl https://gist.githubusercontent.com/paul-ridgway/d39cbb30530442dca416734c3ee70162/raw/c490df8be1976dd062a8b5f429ef42ed1b393ecb/ro-root.sh -o /bin/ro-root.sh`添加脚本
*   使用`sudo chmod +x /bin/ro-root.sh`使脚本可执行
*   使用`sudo nano /boot/config.txt`将这些行添加到配置文件的末尾

```
initramfs initrd followkernel
ramfsfile=initrd
ramfsaddr=-1
```

*   使用`sudo nano /boot/cmdline.txt`将此文本添加到 cmdline 文件的末尾

```
init=/bin/ro-root.sh
```

*   使用`sudo reboot now`重启

> *提示:根分区挂载为* `*/ro*` *，您可以使用* `*sudo mount -o remount,rw /ro*` *使其可写，例如，使用* `*touch /ro/home/pi/test.txt*` *创建一个永久文件，并使用* `*sudo mount -o remount,ro /ro*` *使其全部恢复为只读，该文件将在引导后保留。这是你可以做一些改变的方法，这些改变在重启后仍然存在。*

# 7.密码

*   密码为`3600`的用户`pi`
*   设置密码为`3600`

# 8.远程管理

*   在`http://<ipaddress>:9999/screenshot`的实际截图
*   使用 javascript 远程重启(比如绑定某个按钮):

```
let data = {
  password: "3600"
};
fetch("/restart", {
  method: "POST",
  body: JSON.stringify(data)
}).then((result) => {
  console.log(result)
}).catch((error) => { 
  console.log(error)
});
```

*   使用 javascript 远程关机(比如绑定某个按钮):

```
let data = {
  password: "3600"
};
fetch("/shutdown", {
  method: "POST",
  body: JSON.stringify(data)
}).then((result) => {
  console.log(result)
}).catch((error) => {
  console.log(error)
});
```

*   使用 javascript 设置 dhcp(比如绑定某个按钮):

```
let data = {
  password: "3600",
  server: server.value,         // server web, example: 192.168.86.100:80/terminal/1
};
fetch("/dhcp", {
  method: "POST",
  body: JSON.stringify(data)
}).then((result) => {
  console.log(result)
}).catch((error) => {
  console.log(error)
});
```

*   使用 javascript 设置 static(比如绑定到某个按钮上):

```
let data = {
  password: "3600",     
  ipaddress: ipaddress.value,   // ip address, example: 192.168.86.128
  mask: mask.value,             // mask, example: 255.255.255.0
  gateway: gateway.value,       // gateway, example: 192.168.86.1
  server: server.value,         // server web, example: 192.168.86.100:80/terminal/1
};
fetch("/static", {
  method: "POST",
  body: JSON.stringify(data)
}).then((result) => {
  console.log(result)
}).catch((error) => {
  console.log(error)
});
```

*   检查电缆是否已连接(如捆绑在某个按钮上):

```
fetch("/checkCable", {
  method: "POST",
}).then((result) => {
  result.text().then(function (data) {
    let connected = JSON.parse(data);
    console.log(connected["Result"])
    });
}).catch((error) => {
  console.log(error)
});
```