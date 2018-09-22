---
title: 迷你路由器-Au Home Spot Cube
date: 2018-09-20 11:00:40
tags: 
    - 路由器
    - 数码
---

&ensp;&ensp;自从搬到新的出租屋之后, 我需要一个路由器, 之前就听说小米路由器不可靠, 但我觉得两部设备不至于会有问题吧, 就从京东买了[小米路由器4Q](https://item.mi.com/1182400090.html?cfrom=search). 购买之后我就发现问题了, 过一段时间就很卡, 重启才能解决问题. 上周末终于受不了, 买了个二手路由器替换小米路由器.**不推荐购买小米路由器, 很不稳定**

<!--more-->

---

### 前言
&ensp;&ensp;这个路由器型号我已经在大学用了快三年了, 一直很稳, 价格便宜, 包邮只要22块钱, 这是我入手它的原因. 大学宿舍有6个人, 通常在网设备有10台左右, 没出过啥问题, 舍友也靠着这个路由器打上钻石..., 这个路由内部运行一个类Unix系统, 但是没有开放权限, 只能设置一些对外开放的接口, 所以我们也不能在里面运行程序

<center>
<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fvicqek80xj23342bkn7m.jpg"/>
路由器
</center>

<center>
<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fvicqcv36uj23342bkarp.jpg"/>
路由器接口
</center>

<center>
<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fvid01nausj20fc0lpabg.jpg"/>
路由器进程
</center>

### 设置路由器上网

&ensp;&ensp;这个路由器是日本运营商的, 所以到手之后需要改一些配置

1. 将开关拨到Router, 后台界面才能出现拨号选项
2. 登录192.168.0.1, 账号root, 密码plumeria0077, 用Root是因为后面还需要修改内部配置, 普通账号并不能修改
3. 将右上角的Language换成英文, 保存之后和普通路由器一样设置拨号即可

### 关闭WPS

&ensp;&ensp;WPS这个功能有可能导致有些蹭网的人破解, 我们可以把它关掉, 减少被破解的几率

1. 登录http://192.168.0.1/syscmd.asp, 账号root, 密码plumeria0077
2. 在输入框里面输入以下两个命令, 分别关闭2.4G和5G的WPS功能

```Shell
flash set WLAN0_WSC_DISABLE 1
flash set WLAN1_WSC_DISABLE 1
```

### 更换路由器时区

&ensp;&ensp;因为路由器是日本用的, 所以后台时间为东九区, 所以看着有点别扭, 可以通过设置改回东八区

1. 登录http://192.168.0.1/syscmd.asp, 账号root, 密码plumeria0077
2. 在输入框里面输入以下命令

```Shell
flash set NTP_TIMEZONE -8\ 1
```

### 切换成中国区的5G频道

&ensp;&ensp;日本的5G频道和中国的5G频道不一样, 有一部分设备是接收不到这个路由器发出的5G频道, 所以我们可以修改地区来更换频道

1. 登录http://192.168.0.1/syscmd.asp, 账号root, 密码plumeria0077
2. 输入以下三个指令

```Shell
flash sethw HW_WLAN1_REG_DOMAIN 2
flash sethw HW_WLAN0_REG_DOMAIN 2
flash set WLAN0_CHANNEL 157
```

3. 重启路由器(在Console界面选择Reboot, 不要拔电源, 拔电源可能导致一段时间故障, 以下我会说明原因)

### 总结

&ensp;&ensp;以上就是设置这个路由器全部教程, 总结下这个路由器优缺点

**优点**

1. 稳定, 长时间不重启(建议凉爽的地方, 该路由器发热量较大, 但没遇到过死机的情况)
2. 价格便宜, 支持5G频道
3. 迷你, 放在哪里都不碍事
4. 5V供电, 现在淘宝配的都是USB插线, 可以插在USB插板上(我用的是小米插线板), 不占其他电源位置

**缺点**

1. 型号很老了, 只有二手, 成色较差
2. 虽然支持5G, 但是其实它是802.11a, 速率只有150Mbps, 所以一般用在50-100兆宽带上, 再高就不行了
3. 没有外置天线, 信号弱, 只适合单间这种房子(单间用是完全没问题, 信号很好, 多房间就不推荐)
4. 发热量大, 有人还进行了散热改装, 但我用着是没问题的
5. 拔电源再插上这种重启办法, 会让这个小路由器"死机", 需要等待3-4分钟, 才插上电源才能恢复. 建议在Web端重启

<center>
<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fvicqg4ekcj23342bkdrx.jpg">
</center>