---
title: 部署Telegram Rss Bot
tags:
    - Telegram
---

IM在生活中是必不可缺的,在国内微信和QQ大行其道.

但两者都有缺点.QQ十分臃肿,携带了很多不需要的功能,而微信却功能缺失,连最基本的聊天同步都没有,更有网友为其正名,称做太多功能会和QQ太相似

<!--more-->

***

最近转战到Telegram上, Telegram作为一款简洁的IM, 功能却十分齐全, 我们来看看它能干什么

* 一对一聊天聊天
* 群组聊天
* 私密聊天
* 公告板
* 机器人

## Telegram Bot

今天我们主要讲Telegram Bot, Telegram的Bot有点像命令行的感觉, 你输入命令, 它返回结果, 或者它主动推送给你, 因此我们可以实现很多功能. 作为IM, 我也希望它能推送一些我关心的内容给我, 我不用去打开各种APP提取信息. RSS Bot就是实现这种功能的机器人, 它可以帮你订阅RSS, 在RSS有更新的时候推送给你. 在此我推荐一下RSS订阅源([RSSHub](https://docs.rsshub.app/)), 上面有各个热门网站的RSS源, 方便我们订阅.

## 部署Telegram

废话不多说, 下面演示如何部署RSS Bot.

1. 首先我们搜索BotFather, 他是分配机器人的官方Bot, 我们需要在这里申请Bot, 拿到API Token, 之后的部署需要这个Token验证

<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fv9wof9ggpj209q024dfv.jpg"/>

2. 申请机器人,输入命令/newbot,之后键入名称即可, 我们就可以拿到API Token, 我们需要把这个Token保存

<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fv9wof4oz6j20hb0jrgrh.jpg"/>

3. 下载RSSBot, 我们在Github上找到这个项目[RSSBot](https://github.com/iovxw/rssbot), 不需要编译, 只需要在Release下载程序即可

```Shell
wget https://github.com/iovxw/rssbot/releases/download/v1.4.3/rssbot-v1.4.3-linux.zip
unzip rssbot-v1.4.3-linux.zip
```

4. 运行Bot

```Shell
./rssbot rssdatabase 636680658:AAGHC33MP3xQivKpcn3LzY_6aucCVYY3RWU(这里是刚才申请到的token)
```

这样就完成了我们RSS Bot的部署是不是很简单呢？

5. 最后我们需要设置机器人的命令,也是通过BotFather设置的, 输入/mybot -> 选择自己的机器人 -> Edit Bot -> Edit Commands, 输入如下命令即可

```
rss       - 显示当前订阅的 RSS 列表，加 raw 参数显示链接
sub       - 订阅一个 RSS: /sub http://example.com/feed.xml
unsub     - 退订一个 RSS: /unsub http://example.com/feed.xml
unsubthis - 使用此命令回复想要退订的 RSS 消息即可退订, 不支持 Channel
export    - 导出为 OPML
```

<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fv9zee0429j20f50gp0tl.jpg"/>

6. 这样子我们机器人已经完全设置好了,只需要订阅RSS即可, 有更新的时候, 机器人会主动推送给你

<img src="https://ws1.sinaimg.cn/large/77cf1033gy1fv9zee2spmj20fd059dfw.jpg"/>