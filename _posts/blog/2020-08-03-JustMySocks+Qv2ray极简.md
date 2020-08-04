---
layout: post
title: "JustMySocks+Qv2ray极简教程"
modified:
categories: blog
excerpt:
tags: []
image:
  feature:
date: 2020-08-03

---
justmysocks [官网链接](https://justmysocks.net/members/aff.php?aff=259)

标准价格：$5.88/月 起

最低价格：年付只需交10个月+优惠码（JMS9272283）永久优惠5.2% = 389 人民币/年

下面这段广告引用自[搬瓦工JMS](https://bwgjms.com/post/how-to-buy-justmysocks/)



> justmysocks 是搬瓦工官方提供的优质 V2Ray 服务和 Shadowsocks 服务，最好的 CN2 GIA 线路，月付仅 $5.88 起，更有高达 5G 带宽的 CN2 GIA 线路可选！被墙自动换 IP，无须担心 IP 被墙！！！
>
> justmysocks 支持 V2Ray 协议！V2Ray 是最新的黑科技加持的技术，相比 Shadowsocks 来说更加安全，高效，隐蔽。而 JustMySocks 又同时支持 V2Ray 和 Shadowsocks ，为我们提供更多选择。
>
> justmysocks 使用优质 CN2 GIA 高速线路，无论白天还是晚上，你都无须担心网络问题，并且有五组服务器节点备用，最最最重要的一点是无须担心 IP 被墙问题！！！这可是解决了所有人最担心的痛点啊哈哈哈哈
>
> justmysocks 适合那些不想自己买 VPS 折腾搭建的用户，又或者是担心 IP 被墙的，然后网络速度也要比较优秀稳定的，还不用担心跑路的。当然这对新手小白用户是特别友好的，买了就能用，其他任何屁事都不用自己打理，只管用就好。
>
> 我们一般将提供 V2Ray 服务或 Shadowsocks 服务的商家，俗称机场，或者飞机场。 所以可以将 JustMySocks 称作为搬瓦工机场。或者叫 JMS 机场！
>
> 由于是搬瓦工提供的服务，所以不用担心跑路之类的啦~ 当然，搬瓦工机场不同于其他机场，如果购买了觉得不满意，也是支持退款的！
>
> 并且最重要的一点是 justmysocks 不同于那些国人的机场，justmysocks 是搬瓦工官方背书的，数据安全是绝对靠谱放心的，无须有任何顾虑。
>
> 
>
> Just My Socks 优势
>
> 
>
> 搬瓦工出品，服务安全可靠
>
> 被墙自动换 IP，无须担心 IP 被墙
>
> 支持退款，放心无忧
>
> 支持支付宝付款
>
> 最好的 CN2 GIA 线路
>
> 高达 5G 带宽的 CN2 GIA 线路可选
>
> TCP & UDP 协议支持
>
> 支持更改密码
>
> 支持更改服务器端口



购买的部分我们就略过了，总之购买之后你的service界面划到最下面会有五个二维码和五个链接（只考虑via jamjams.net domain name 的五个，因为会动态更新ip）。它们就是我们的通关密语。下面教大家如何使用：

# 电脑端

推荐使用[Qv2ray](https://qv2ray.net/)，教程参考：https://qv2ray.net/getting-started/。教程细节很多，实际就四步：

1. 从(Github Release)[https://github.com/Qv2ray/Qv2ray/releases/]下载对应系统的Qv2ray版本
2. 从(V2Ray Core)[https://github.com/v2fly/v2ray-core/releases]下载对应系统的v2ray core版本
3. 打开Qv2ray首选项-内核设置， 找到核心可执行文件目录路径，把下载的v2ray core解压拷贝进去然后验证就可以了
4. 新建链接，把五个通关密语链接拷贝进来即可

选择任意一个连接，连接之后Mac和Windows电脑会自动使用Qv2ray代理。可以在设置里选择绕过中国大陆，这样大陆网站就不会走代理，直连速度会更快。

# 手机端

ios端推荐Shadowrocket， 海外账号app store可下，app里直接扫上面的二维码就可添加连接。android端推荐Kitsunebi， play store可下。两个app都支持绕过大陆及局域网。

其他客户端请参考官方推荐<https://www.v2ray.com/awesome/tools.html>
