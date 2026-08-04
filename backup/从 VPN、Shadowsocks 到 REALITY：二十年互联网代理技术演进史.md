---
> 本文只梳理代理技术的迭代逻辑，不涉及任何使用教程，望知悉。

---

## 概况

这些年代理技术大致经历了四个阶段，分界线都是标志性事件：**2012 年 Shadowsocks 诞生，2017 年 Trojan 诞生，2020 年之后 VLESS / REALITY 逐渐成为主流**。在这之前，则是 VPN 一统天下的年代。

| 时间 | 阶段 | 关键词 |
| --- | --- | --- |
| 2000—2011 | VPN 时代 | 加密、全局、特征明显 |
| 2012—2016 | Shadowsocks 时代 | 专用协议、分流、特征弱化 |
| 2017—2019 | 伪装时代 | TLS、模仿 HTTPS |
| 2020 至今 | 不可区分时代 | 指纹、行为、REALITY |

每个阶段解决的核心问题都不一样：

* VPN 解决了“**内容加密**”，但特征太明显；
* Shadowsocks 解决了“**特征弱化**”，但经不住主动探测；
* Trojan 把思路从“隐藏”改成“**伪装**”；
* 到了 REALITY，连 TLS 握手和流量节奏都要模仿。

---

## 古代：VPN 时代（2000—2011）

金盾工程 1998 年立项，2003 年前后开始运作。早期的封锁手段就三板斧：**IP 封禁、DNS 污染、关键词过滤**。

DNS 污染这个事，说白了就是你查域名的时候，它抢答一个假 IP。当时的应对办法也简单：把 DNS 换成国外的，或者干脆改 hosts，把解析写死在本地。但这些都是雕虫小技——一旦审查进入数据包层面，明文 HTTP 的关键词过滤一上，hosts 就废了。

这时候就得靠代理了。核心思想其实就一句话：**我访问不了，就让一台能访问的机器替我去访问，再把结果传回来**。HTTP 代理、SOCKS 代理都是这个思路，VPN 也是。

VPN 本身不是为翻墙设计的，它的本职是跨国公司、远程办公，主要协议有 PPTP、L2TP、IPSec、IKEv2、OpenVPN 这一串。但它加密、工作在底层、对所有应用透明、设置又简单，所以在很长一段时间里就是“科学上网”的代名词，以至于到现在还有很多小白把所有代理工具都叫 VPN。

VPN 的问题后来暴露得很清楚：

* **太重**：所有流量都走隧道，带宽浪费严重，还不能分流；
* **特征太明显**：协议是公开的，握手和封装都有固定样式，连接模式高度统一，审查方只要专门研究它就行；
* **定位错位**：它是为组网设计的，不是为对抗审查设计的。

那年代还有一串如今已经没多少人记得的名字：goagent、自由门、fqrouter……goagent 蹭 Google App Engine 的免费资源当代理，火了两年多，随着 GAE 收紧免费额度也就没了。它们的意义在于证明了“借大厂基础设施搭代理”这条路可行，后来各种免费节点都是这个思路的徒子徒孙。

---

## 近代：Shadowsocks 时代（2012—2016）

2012 年 4 月 22 日，V2EX 上有人发了个帖子：[《发一个自用了一年多的翻墙工具 shadowsocks》](https://web.archive.org/web/20120422191812/http://www.v2ex.com/t/32777)。作者 clowwindy，整个项目当时就九百行左右。

Shadowsocks 最大的创新是“**拆**”：把代理拆成客户端和服务端。客户端在你机器上开一个 SOCKS5 端口，任何支持代理的软件都能用；客户端把请求用预共享密钥加密后转给服务端，服务端再替你去访问。

除了拆，SS 还有一个被很多人忽略的点：**分流**。配置文件里匹配的网站走代理，其余流量直接连，国内网站不再跟着绕远路。这在当时是体验上的一大飞跃——VPN 时代是“一开全走”，SS 时代是“该走的走，不该走的留”。

它厉害在哪？**没有固定样式的握手，没有明文标识**，加密后的载荷看起来就是随机字节；连接又是无状态的，来一个请求转一个请求。特征弱到什么程度？很长一段时间里，审查设备根本不知道该拿它怎么办。

加密上也有演进。早期用流加密，比如 AES-256-CFB，密文没有完整性校验，存在篡改和重放的风险；后来社区推进到 **AEAD**，加密的同时带完整性认证，防篡改、防重放。这一步很重要，相当于给每个包都加了防伪标签。

生态起来得也很快：Python、Node、Go、C 的 libev 版本遍地开花；libev 版资源占用极低，被装到路由器上做透明代理，一机代理、全家可用。

### 风云突变

2015 年 8 月 20 日，clowwindy 在 GitHub 发了一段话，大意是：警察两天前找上门，要求停止这个项目，今天又要求删掉 GitHub 上所有代码，他没有选择。当晚，他关掉了所有 Issue 面板，删光帮助信息，仓库描述全部改成 **Something happened**。

第二天，shadowsocks-windows 的 #305 issue 下，他留下一句话：

> I was invited for some tea yesterday. I won't be able to continue developing this project.

之后他在 Twitter 发了一条 “thanks.”，证明人没事。至此，SS 原作者正式退场。

### SSR 的兴亡

SS 退场后，接棒的是 ShadowsocksR（SSR），作者 breakwa11。这个人极具争议：接手开发却违反 GPL 封闭源代码，发布时暗示自己是原作者，还搞了个 TCP 协议头混淆，声称提升安全性。clowwindy 当时在 issue 里直接开怼，大意是：真的试验性功能更应该开源让大家一起分析，闭门造车在加密领域是大忌；这段互撕到现在看都挺精彩。

不管其中是非曲直，SSR 的结局也令人唏嘘。2017 年 7 月 19 日，breakwa11 转发了深圳启用 SS 协议检测的消息，引发大量恐慌；7 月 27 日，她遭到自称 “ESU.TV” 的不明身份人士人肉威胁，对方要求停止开发，否则公开更多个人资料。几天后，breakwa11 删掉 GitHub 上的所有代码，解散交流群组，ShadowsocksR 就此落幕。

但开源这东西，**项目可以消失，社区不会**。大量的 fork 一直续到后来，各种分支至今还在更新。

---

## 现代：伪装时代（2017 至今）

2017 年出现的 Trojan，把整个行业推进了现代。

它的特别之处在于：之前的思路要么消除特征，要么做混淆，但**混淆本身也可能成为特征**——一段流量刻意不像加密流量，反而更可疑。Trojan 干脆不搞自定义加密了，直接复用 TLS，让流量长得跟普通 HTTPS 网站一模一样。它的项目说明里有句话很经典：

> Trojan 不是一个固定的程序或协议，而是一种思路——模仿最常见的服务。

它还支持 fallback：认证不过的连接直接转给真实网站，主动探测也探不出破绽。

### V2Ray：一个“框架”，不是一个协议

接下来把 V2Ray 这条线说清楚，因为它的历史经常被讲乱。

V2Ray 是 Project V 的核心，2015 年 9 月以 Victoria Raymond 的笔名发布。“V2”取自 V2 火箭，“Ray”是射线，意思就是新一代的翻墙工具。它从一开始就不是一个协议，而是一个**框架**：一个进程可以同时挂 VMess、VLESS、Trojan、Shadowsocks 多种入站协议，再把流量按规则路由到不同的出口。

架构上，V2Ray 把协议和传输层彻底解耦：

* 协议只负责鉴权和加密，比如 VMess 用 UUID 鉴权、自带 AEAD；
* 传输层可以在 TCP、mKCP、WebSocket、HTTP/2、gRPC 里随便挑；
* mux.cool 负责多路复用，多个子连接共用一条底层连接；
* 路由引擎靠 geosite / geoip 数据库做分流，国内直连、国外代理。

这也是后来所有“协议栈”设计的老祖宗。VMess 后来也老了：alterId 归零后自动切到 AEAD，旧的 MD5 认证在 2021—2022 年被彻底淘汰。社区有统计称，裸奔的 VMess、Trojan 检测率已经到了八到九成，新部署基本都转向 VLESS。

2019 年，原作者 Victoria Raymond 失联，原域名和基础设施相继废弃；2020 年，社区组建 v2fly 接手，成为事实上的官方参考实现。V2Ray 从此不是一个人的项目，而是一个社区项目。

### Xray：一次因为许可证吵出来的分家

Xray 的诞生不是因为技术，是因为许可证。

VLESS 和 XTLS 的设计者 rprx 在 V2Ray 社区里干活，但 XTLS 库的许可证不是 V2Ray 用的 MIT。Debian 的打包维护者提了个 issue，希望作者改许可证方便打包，结果吵了起来。V2fly 社区投票认定 XTLS 不符合 MIT 协议，在 v2ray-core 4.33.0 里把 XTLS 移除了。rprx 干脆拉人成立了 Project X，从 V2Ray fork 出 Xray-core——名字就是 XTLS 和 V2ray 的合体。

Xray 一开始就是 V2Ray 的超集：把 v2ray 和 v2ctl 合并成单个二进制，完整支持 VLESS 和 XTLS，还引入了 Linux 的 splice 优化、IPv4/IPv6 透明代理。之后的发展大家都看到了，V2Ray 一度停更，Xray 反而一路狂奔。

XTLS 值得单独说。传统的“伪装成 TLS”是套两层 TLS——外面一层给审查看，里面一层传数据，重复加解密又慢又费；XTLS 把内外两条 TLS 合并处理，只做一次加解密，性能大幅提升。rprx 在代码库里写的描述是 “THE FUTURE”，口气不小，但确实成了现实。

### REALITY：连握手都是真的

REALITY 是 Xray 最出圈的东西，2022 年提出，2023 年前后开始大规模落地。

它的思路比 Trojan 更进一步：Trojan 是“长得像 HTTPS”，REALITY 是“**干脆就是真的 HTTPS**”。服务端把客户端的 TLS ClientHello 原样转发给一个真实网站（比如 www.google.com），把真实证书和真实握手拿回来，再用这段真实会话派生密钥做代理鉴权。也就是说，握手、证书、指纹全是真的，只是最后一步悄悄把连接“劫持”成了代理隧道。

主动探测连上来，得到的响应和访问真实网站一模一样；想封锁它，就得先封锁它借用的那个真实网站——而这在现实中几乎做不到。这也是为什么 REALITY 被一些人称为“**结构性不可封锁**”。

### sing-box：一个二进制的“瑞士军刀”

2022 年 8 月，nekohasekai 发布了 sing-box 的 1.0 测试版。它的定位和 V2Ray、Xray 都不同：不偏袒任何协议，而是做**统一框架**——一个二进制同时支持 VMess、VLESS、Trojan、Shadowsocks、Hysteria、TUIC、Naive 和 REALITY。

配置模型也重新设计过：早期 V2Ray 那套庞大的 geosite / geoip 文件，被它改成可动态更新的规则集，内存占用更小、加载更快。很多新客户端（NekoBox、Hiddify 之类）都是基于它做的。如果说 V2Ray 是协议栈，Xray 是激进派，那 sing-box 更像是“通用底座”——想混着用协议，它是目前最方便的起点。

### 其他几条线

协议之外还有一些值得记的：

* **Hysteria2、TUIC**：基于 QUIC/UDP，主打速度和弱网优化，代价是 UDP 流量在某些环境里比较扎眼；
* **Clash / Mihomo**：Clash 用 YAML 规则分流定义了现代客户端范式，原项目 2023 年归档后由 Mihomo 接棒；
* **软路由与 OpenWrt**：OpenWrt 起源于 2004 年 Linksys 因为 GPL 问题公开 WRT54G 源码的事，后来 x86 小主机降价，软路由成了折腾圈标配，旁路由、跑满千兆一度是热门话题。

### 政策面的收紧

2017 年开始，风向明显变了：

* 2017 年 1 月，工信部印发《关于清理规范互联网网络接入服务市场的通知》；
* 2017 年 7 月底，中国区 App Store 大量 VPN 应用被下架，苹果回应称“应要求移除一些不符合规范的 VPN App”；
* 2017 年 10 月十九大期间，大量线路被封，SSR 尤其严重，会后才陆续解封；
* 2018 年 1 月及两会期间，更大规模的 IP 封杀，坊间甚至有“SS 刚搭好十几分钟就被封”的传闻；
* 2018 年 9 月，《公安机关互联网安全监督检查规定》发布，11 月 1 日起施行。

网上还流传过一份据称是运营商的协议识别对照表，真实性未知，但挺有意思：

| 方案 | 识别显示 |
| --- | --- |
| Shadowsocks / SSR | TCP 业务，显示服务器 IP |
| SS / SSR + http_simple | Web 方式访问，显示混淆域名 |
| SSR + TLS(443) | HTTPS 安全类流量 |
| V2Ray + TLS | HTTPS 网络连接，显示证书域名 |
| kcptun | UDP 业务 |

当段子看可以，别太当回事。

另外值得一提的是，除了代理这条主线，还有一条旁路思路：**内网穿透**。ZeroTier 这类方案把多台设备拉进同一个虚拟局域网，从效果上看等于人在国外。它和代理是两套完全不同的思路，但经常被混在一起讨论。

---

## 未来

说几点看法。

**第一，AI 行为识别。** 流量识别已经从规则匹配转向行为建模：连接时序、包长分布、TLS 指纹、HTTP 行为，全是特征。2014 年就有人预言过，Shadowsocks 这种无状态、特征少的协议，流量规模一上来，机器学习很容易识别（参见[这篇博客](https://blog.robotshell.org/2014/about-shadowsocks/)）。这话现在看基本说中了。反过来，伪装也在往“模拟真实浏览”的方向卷，这场博弈短期内看不到头。

**第二，去中心化。** 一个同时满足高效、匿名、完全去中心化的网络，目前还不存在。Tor 解决了匿名，但多跳中继带来的延迟和低吞吐摆在那，效率这一条始终过不了。

**第三，白名单。** 会不会有“默认全不可达，只放行白名单”的一天？这种问题变量太多，没法确定，而不确定的问题就没法当问题讨论。

---

## 总结

回看这二十年，攻防其实一直在互相逼着升级：

* 审查方：从看得懂内容，到看得懂地址，再到看不懂也要猜；
* 代理方：从换个路径，到加密内容，再到伪装成正常流量。

但有个细节值得注意：代理方的每一次进步，几乎都来自对既有技术的复用——SOCKS、加密、TLS、HTTPS，没有哪个协议是为对抗审查发明的，全是被重新组合出了新用途。开源接力也一直在起作用，从 clowwindy 退场后的分支，到 v2fly 接手、Xray 分家、sing-box 统一，技术始终跨越着个人生命周期。

**技术是中立的，用法不是。** 这大概就是这篇东西最想说的。

---

## 参考资料

* [Shadowsocks 的前世今生（shadowsockshelp）](https://shadowsockshelp.github.io/Shadowsocks/Shadowsocks-wiki.html)
* [V2Ray 详解：Project V、协议、Xray 和 sing-box](https://plisio.net/zh/cybersecurity/v2ray-vpn-project)
* [Xray 教程：项目由来与 V2Ray 的关系](https://xtrojan.org/xray-tutorial/)
* [v2fly/v2ray-core](https://github.com/v2fly/v2ray-core)
* [XTLS/Xray-core](https://github.com/XTLS/Xray-core)
* [SagerNet/sing-box](https://github.com/SagerNet/sing-box)
* [维基教科书：翻墙简史 / Shadowsocks](https://zh.wikibooks.org/zh-tw/%e7%bf%bb%e5%a2%99%e7%b0%a1%e5%8f%b2/Shadowsocks)
* [About Shadowsocks（2014 年技术博客）](https://blog.robotshell.org/2014/about-shadowsocks/)
* [网络代理工具演进史](https://blog.cuixu.cn/posts/network-proxy-tools-evolution-history/)
* [trojan 项目主页](https://github.com/tangkv/trojan)
* [EPFL Wiki：The Great Firewall](https://wiki.epfl.ch/china/firewall)