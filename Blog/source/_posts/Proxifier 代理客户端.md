---
title: Xcode 设置 SPM 全局代理(Proxifier)
---

### **一、前提**

目前 iOS 项目管理第三方库的方式有多种，这里使用的是官方推荐的`SPM`进行包管理，大部分开源项目都在`github`，不过如果使用`SPM`进行包管理，需要使用代理模式才可以进行下载。

目前使用的`ClashX`作为代理工具，Xcode 使用的是自带的 git，可是即使设置全局或控制台的代理都无法使`SPM`走代理模式。故我们可以使用 Proxifier 抓包工具进行全局网络代理设置。

### **二、Xcode 自带 git**

经过查询 Xcode 的使用的自带`git`程序为`com.apple.dt.Xcode.sourcecontrol.Git`，在`Xcode`里面的进行`SPM`下载第三方库的时候可以到控制台查看.具体命令行为`ps aux|grep com.apple.dt.Xcode`，显示结果如下：

![](</images/Proxifier/7aafecc0-902a-11ee-a20b-a714d8de737e.jpeg?v=1&type=image>)

### 三、Proxifier 配置

利用抓包软件`Proxifier`进行代理设置，该软件为*付费*，官网地址为[Proxifier](https://www.proxifier.com/)，可以下载使用版本，后期也可以购买。

打开 Proxifier 之后会需要权限，需要输入系统密码授权，在网络部分会显示这两个菜单:

![](</images/Proxifier/00b1b0f0-902c-11ee-a20b-a714d8de737e.jpeg?v=1&type=image>)

![](</images/Proxifier/9b6c2260-902c-11ee-a20b-a714d8de737e.jpeg?v=1&type=image>)

然后打开`Proxies`，添加本机的代理端口，这里可以使用`http`或者`socks`，配置如下：

![](</images/Proxifier/79da83b0-9240-11ee-a20b-a714d8de737e.jpeg?v=1&type=image>)

这里使用的是本机`socks5`端口`7890`，然后再打开`Rules`进行规则配置，其中`Default`没法配置，默认直接走`Direct`即可(不然全都会走代理)，新增一个规则，添加应用程序`Xcode`，然后再把上面找到的`com.apple.dt.Xcode.sourcecontrol.Git`追加到后面去，下面`Action`添加刚才设置的`Proxy`即可，配置参照

![](</images/Proxifier/b5cf7ba0-9240-11ee-a20b-a714d8de737e.jpeg?v=1&type=image>)![](</images/Proxifier/bf39e9f0-9240-11ee-a20b-a714d8de737e.jpeg?v=1&type=image>)

配置生效之后就可以看到`Xcode`里面`SPM`拉取`github`时候流量会走到代理，速度明显提升。

##### 最后来一波福利（proxifier 是收费的，下面免费提供 v2/v3 版本的注册码）

proxifier for Mac 注册码

**V2：**

**P427L-9Y552-5433E-8DSR3-58Z68**

**V3：**

**3CWNN-WYTP4-SD83W-ASDFR-84KEA**
