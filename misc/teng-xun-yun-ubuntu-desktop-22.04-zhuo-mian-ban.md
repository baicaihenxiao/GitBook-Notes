# 腾讯云 Ubuntu Desktop 22.04 桌面版



{% embed url="https://blog.csdn.net/qq_40829735/article/details/130475741" %}

[https://blog.csdn.net/qq\_40829735/article/details/130475741](https://blog.csdn.net/qq\_40829735/article/details/130475741)

#### 安装 Ubuntu Server 22.04

1. 安装 Ubuntu Server 22.04
2. 重置登陆密码， 默认用户名 ubuntu

#### 添加用户 <a href="#t1" id="t1"></a>

不建议直接使用默认用户， 添加用户（demodeom）

```
sudo adduser demodeom
1
```

将 demodeom 添加到 sudoer 文件

```
sudo visudo
1
```

在文件末尾添加

```
demodeom  ALL=(ALL:ALL) NOPASSWD: ALL
1
```

`ctrl + o` 然后回车保存， `ctrl + x` 退出编辑模式

切换用户， 后续操作使用 demodeom 用户操作

#### 安装桌面版 <a href="#t2" id="t2"></a>

更新软件仓库

```
sudo apt update
1
```

升级软件

```
sudo apt upgrade
1
```

安装桌面

```
sudo apt install ubuntu-desktop
1
```

重启

```
sudo reboot
1
```

重启之后， 使用 腾讯云提供的 [VNC](https://so.csdn.net/so/search?q=VNC\&spm=1001.2101.3001.7020) 登陆方式即可访问 Ubuntu 桌面版

#### VNC <a href="#t3" id="t3"></a>

VNC [远程桌面](https://so.csdn.net/so/search?q=%E8%BF%9C%E7%A8%8B%E6%A1%8C%E9%9D%A2\&spm=1001.2101.3001.7020)登陆工具

**安装兼容的显示管理器**

Ubuntu Linux 使用 [GNOME](https://so.csdn.net/so/search?q=GNOME\&spm=1001.2101.3001.7020) 桌面管理器 (GDM) 作为默认显示管理器。较新版本的 Ubuntu 使用 gdm3。不幸的是，GDM 通常不能很好地与 x11vnc 服务器一起工作。要克服这个问题，您将必须安装 Light Display Manager 或 lightdm。

```
sudo apt install lightdm
1
```

安装过程中会出现以下画面。按键盘上的 Enter 键继续。

![img](https://img-blog.csdnimg.cn/img\_convert/4167525fff5603ef93f54f00cf6c1b5e.png)

接下来，选择 lightdm 选项并按键盘上的 Enter 键

![img](https://img-blog.csdnimg.cn/img\_convert/85f99e3aa3a285023b42ece53e91a996.png)

重新启动您的 PC 以使显示管理器更改生效。

```
sudo reboot
1
```

**安装 x11vnc 服务器**

安装 x11nvc 服务器

```
sudo apt install x11vnc
1
```

**配置 x11vnc 服务器**

您现在将配置用于启动 x11nvc 服务器的服务。在 /lib/systemd/system/ 目录中创建一个名为 x11nvc.service 的文件

```
sudo vim /lib/systemd/system/x11vnc.service
1
```

将以下内容复制并粘贴到新创建的服务文件中 (注意不要全部复制， 一定要修改 登陆密码 )

```
[Unit]
Description=x11vnc service
After=display-manager.service
network.target syslog.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -forever -display :0 -auth guess -passwd 登陆密码 
ExecStop=/usr/bin/killall x11vnc
Restart=on-failure

[Install]
WantedBy=multi-user.target
12345678910111213
```

重新加载 systemd 管理器配置

```
sudo systemctl daemon-reload
1
```

然后，启用 x11vnc 服务

```
sudo systemctl enable x11vnc.service 
1
```

最后，使用以下命令启动 VNC 服务器。

```
sudo systemctl start x11vnc.service
1
```

使用 systemctl 检查 x11vnc 服务的状态

```
systemctl status x11vnc.service
1
```

输出应与下图类似。

![img](https://img-blog.csdnimg.cn/img\_convert/91adfef69de225c3973b6e7d82123b82.png)

**在防火墙中启用服务器端口**

x11vnc 服务器使用的端口 `5900`, 需要在腾讯云的防火墙开放 5900 端口\
![在这里插入图片描述](https://img-blog.csdnimg.cn/0e94f0eb30934821b1596ff8a19d46d5.png)

**从另一台计算机连接**

您现在可以使用 VNC 通过远程桌面连接连接到您的 Ubuntu 系统

您可以使用任何 VNC 客户端连接到 Ubuntu Linux PC。推荐的 VNC 查看器之一是 RealVNC 的 VNC Connect。它几乎适用于所有主要平台，包括 macOS、Linux、Windows、iOS、Android 等

#### RealVNC <a href="#t9" id="t9"></a>

**安装 VNC Viewer**

1. 下载 Linux 版本的压缩包 VNC-Connect-Installer-1.3.0-Linux-x64.tar.gz
2. 解压之后, 运行 VNC-Connect-Installer-1.3.0-Linux-x64
3. 选择安装VNC Viewer **Install VNC Viewer**

**VNC Viewer**

1. 运行 VNC Viewer
2. 输入您要连接的 PC 的 IP 地址，然后输入 x11vnc 服务器使用的端口号。然后，按键盘上的 Enter 键进行连接。

![img](https://img-blog.csdnimg.cn/img\_convert/086df2540c763a0d202cb9f591407fd6.png)
