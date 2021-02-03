# Ubuntu 安装与配置

## 1. Ubuntu 安装（N卡）

### 显卡驱动配置

Ubuntu 桌面版对 N 卡的支持做的不好,默认情况下会启用开源 N 卡驱动 *nouveau*。为了避免安装失败，最好先禁用显卡驱动，在安装界面将光标放在`install ubuntu`，然后进入编辑模式，在倒数第二行*quiet splash*后加入语句`$vt_handoff acpi_osi=linux nomodeset`之后可以正常安装系统。

由于禁用了驱动，安装过程中分辨率会比较低，可能导致部分页面显示不全。在显示不全的页面，按下`Alt+F7`或者右击窗口状态栏选择`move`，此时移动光标，窗口也会跟着移动。

安装完成后分辨率是不可调的，此时需要安装N卡驱动
安装方式有三种：
1. 在官网手动下载安装包（不推荐，难以成功。一般官网只用来查询显卡对应的驱动版本）
2. 利用 Ubuntu 官方的源进行安装（推荐）
3. 添加 PPA 源，从添加的源里面寻找合适的驱动安装（适用于安装新版本的驱动，步骤2中可能没有）
   
**方法2**
```
sudo apt-cache search nvidia
sudo apt install nvidia-drivers-***
```
**方法3**
```
sudo add-apt-repository ppa:graphics-drivers
sudo apt-get update
ubuntu-drivers devices # 查看适用显卡驱动
sudo apt-get install nvidia-drivers-440
```
安装完成后，重启电脑运行nvidia-smi即可查看是否安装正确

### 系统分区

关于分区类型，Ubuntu 默认使用的是 *ext3* 文件系统，相较于较新的 NTFS 以及常见的 FAT16/32 来说，默认的系统类型虽然比较老旧，性能并非出众，但是确实比较靠谱的 Linux 文件系统。值得一提的是，Ubuntu 是可以对新的文件系统进行读写的。

安装系统时，我们需要为空闲的磁盘空间进行分区，分区的含义是在磁盘空间里面单独划分一部分供某个目录使用。拥有单独分区的目录与其他分区隔离开来，互不干扰，即便其它分区受到损坏，此部分依然被完整保留。常见的会为以下的路径划分分区：
1. `/`：根目录，用于存储系统文件。
2. `swap`：交换分区，Linux环境下的虚拟内存。
3. `/boot`：存放操作系统的内核和在启动系统过程中所需要的文件。
   在旧的 Linux 教程下，一般会建议单独对此目录进行分区，主要原因是 Lilo 无法引导 1024 柱面后的操作系统内核。但是现阶段的 Grub 可以引导。但是 Grub 仍然存在无法引导 137 GB 之后的分区中的内核。所以如果硬盘容量比较大，那么还是建议将 `/boot` 单独挂载到 137 GB 之前的分区中去。否则没有必要。
4. `/usr`：用于存放软件的地方
   如果系统里面存在一些自己编译的软件，那么建议为`/usr`,`/opt`单独划分分区。这样在重装系统时，就不用重新编译这些软件，因为有单独分区的情况下，重装系统时可以保留这些分区的文件。但是如果都是使用 linux 的包管理器安装软件，那么没有必要单独为他们划分分区，因为可以非常便捷的重新安装。
5. `/home` 系统用户的 home 路径
   一般**强烈建议**为此目录单独划分分区，并且分配比较大的空间。因为我们日常使用的**文件资料**以及某些开发工具可能都位于此目录下。如果不为此目录单独划分分区，那么如果遇到系统奔溃需要重装的局面，里面的文件可能全都损失。而单独划分分区就不会有这个尴尬的情况发生，我们可以在保留这个分区的条件下，重新安装系统，并将这个保留分区挂载到新系统的`/home`路径下即可。

上述路径，除了`swap`以外，其余所有的路径都位于根目录这一路径下，是根目录的子目录。如果不为他们单独分区的话，相当于所有目录共享根目录所在分区的空间。如果有单独的分区给某一路径，那么这一路径只会占用他被分配的那一个分区的空间，与其他路径的空间不共享。

### wayland

目前 wayland 已经不再支持 Ubuntu 系统。

Wayland是一种全新的显示服务器，Ubuntu现在开始用来替代原来的xorg，个人使用觉得Wayland对N卡的匹配要更好，启动更流畅。
但是目前Wayland依然存在问题。
1. 对于fcitx的匹配做得不够好，原因目前还不清楚，下面的解决办法可行：
打开`/etc/profile.d/input-method-config.sh/`:
```
修改：
export XMODIFIERS
export GTK_IM_MODULE
export QT_IM_MODULE

为如下：
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
```
重新启动即可。

2. 在VScode下使用matplot绘图不显示，目前没有发现解决方法。

### 输入法问题

20.04尚不支持搜狗输入法，目前中文输入可以使用fcitx-googlepinyin和fcitx-cloudpinyin(提供云候选)

## 2. Grub

如果系统开机引导失败，可能进入Grub界面，此时需要进行引导修复。修复步骤：
1. 准备一个ubuntu安装u盘;
2. 开机从u盘启动，试用ubuntu;
3. 进入终端，安装boot-repair工具
```
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install boot-repair
```
4. 运行boot-repair，进入高级选项设置完后进行修复，基本问题能解决。

如果修复成功以后，引导界面出现很多`.efi`项，此时需要修改grub文件：
1. cd进入`/boot/grub/`
2. 打开**grub.cfg**文件，注释掉.efi项即可。

## 3. 网络配置

### v2ray配置

#### 时间校准

对于 V2Ray，它的验证方式包含时间，就算是配置没有任何问题，如果时间不正确，也无法连接 V2Ray 服务器的，服务器会认为你这是不合法的请求。所以系统时间一定要正确，只要保证时间误差在90 秒之内就没问题。

Linux 下可以使用命令 `date -R` 来查看时间，如果时间不准确，可以使用如下命令：
```
date --set="year-month-date hour:minute:second"
```
将时间设置正确就可以了，不需要修改时区，因为 v2ray 会自己转换时区。

#### 服务端安装
下载官方脚本`wget https://install.direct/go.sh`, 然后执行安装脚本`bash go.sh`

#### 服务端配置

主要是配置 Vmess 通讯协议和 KCP 传输协议

```
{
  "inbounds": [
    {
      "port": PortNumber,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "IdNumber",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "mkcp",
        "kcpSettings": {
          "uplinkCapacity": 5,
          "downlinkCapacity": 100,
          "congestion": true,
          "header": {
            "type": "none"
          }
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
启动v2ray
`systemctl start v2ray`
修改服务器的防火墙配置：
```
firewall-cmd --zone=public --add-port=PortNumber/udp --permanent
firewall-cmd --reload
```

#### 客户端安装与配置

Linux客户端安装过程与服务端是一样的，但是由于v2ray的安装文件源不在国内，直接执行安装脚本无法正常进行，所以需要提前下载好v2ray安装文件。

获取安装脚本：
`wget https://install.direct/go.sh`
本地安装v2ray：
`sudo bash go.sh --local ./v2ray-linux-64.zip`

配置文件：
```
{
  "policy": null,
  "log": {
    "access": "",
    "error": "",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "tag": "proxy",
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "auth": "noauth",
        "udp": true,
        "ip": null,
        "address": null,
        "clients": null
      },
      "streamSettings": null
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "Vps IdNUmber",
            "port": PortNumber,
            "users": [
              {
                "id": "IdNumber",
                "alterId": 74,
                "email": "t@t.tt",
                "security": "aes-128-gcm"
              }
            ]
          }
        ],
        "servers": null,
        "response": null
      },
      "streamSettings": {
        "network": "kcp",
        "security": null,
        "tlsSettings": null,
        "tcpSettings": null,
        "kcpSettings": {
          "mtu": 1350,
          "tti": 50,
          "uplinkCapacity": 5,
          "downlinkCapacity": 100,
          "congestion": false,
          "readBufferSize": 2,
          "writeBufferSize": 2,
          "header": {
            "type": "none",
            "request": null,
            "response": null
          }
        },
        "wsSettings": null,
        "httpSettings": null,
        "quicSettings": null
      },
      "mux": {
        "enabled": true,
        "concurrency": 8
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {
        "vnext": null,
        "servers": null,
        "response": null
      },
      "streamSettings": null,
      "mux": null
    },
    {
      "tag": "block",
      "protocol": "blackhole",
      "settings": {
        "vnext": null,
        "servers": null,
        "response": {
          "type": "http"
        }
      },
      "streamSettings": null,
      "mux": null
    }
  ],
  "stats": null,
  "api": null,
  "dns": null,
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "type": "field",
        "port": null,
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "ip": null,
        "domain": null
      }
    ]
  }
}

```

启动v2ray：
`systemctl start v2ray`

需要配置代理的话，则参看各应用**socks5**代理配置规则即可。

### 新版本v2ray配置

最新版本的v2ray启用了新的安装脚本，之前的脚本已经无法使用。新的安装脚本获取方式为`curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh`

安装完成以后的配置基本不变，唯一的区别是`config.json`的位置移到了`/usr/local/etc/v2ray/config.json`路径下。

## 4. snap 

snap是Ubuntu母公司开发的全新的包管理方式，区别于apt和dpkg. snap对其管理的每个程序都单独用一个目录存储（这些目录都位于 */snap* 路径下，并且在目录中包含了该程序运行所依赖的所有库。这带来一个优点就是，我么可以同时在一台机器上部署同一个程序的不同版本而不用担心冲突问题，类似于conda的环境管理；缺点就是每一个应用占用的存储空间比较大

snap所安装的软件源都在国外，国内安装最好走代理，socks5代理设置命令
```
sudo snap set system proxy.http="socks5：//127.0.0.1:portnumber"
sudo snap set system proxy.https="socks5://127.0.0.1:portnumber"
```
查看代理设置命令
```
sudo snap get system proxy
```
删除代理命令
```
sudo snap unset system proxy.http
sudo snap unset system proxy.https
```
