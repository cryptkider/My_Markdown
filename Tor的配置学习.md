# Tor的配置学习

- 安装

首先下载tor的核心组件(这里是指不下载封装好的tor browser，而是其核心代理组件)

```bash
root@kali:~# apt-get install tor
```

- 配置

下载好的tor核心组件，其配置目录存放在了:**/etc/tor**下的torrc配置文件

现在我们需要做一些手脚:

***如何避免“陷阱节点”/“蜜罐节点”***

在torrc配置文件的末尾，加入下面这行（`ExcludeNodes` 表示排除这些国家/地区的节点，`StrictNodes` 表示强制执行）：

```
ExcludeNodes  {cn},{hk},{mo}
StrictNodes  1
```

##注意：关于【StrictNodes】的语法，补充说明一下：
 　如果不设置 `StrictNodes 1`，Tor 客户端首先也会规避 `ExcludeNodes` 列出的这些国家。但如果 Tor 客户端找不到可用的线路，就会去尝试位于排除列表中的节点。
 　如果设置了 `StrictNodes 1`，即使 Tor 客户端找不到可用的线路，也不会去尝试这些国家的节点。

请也把这些国家也列入 Tor 的排除节点列表（当然可能不全:）

| 代码 | 国家     |
| :--- | :------- |
| {kp} | 北朝鲜   |
| {ir} | 伊朗     |
| {sy} | 叙利亚   |
| {pk} | 巴基斯坦 |
| {cu} | 古巴     |
| {vn} | 越南     |
| {ru} | 俄罗斯   |
| {by} | 白俄罗斯 |

- 组合拳之跟 Tor 组合双重代理

利用SSR作为tor的前置代理，一来是过墙，二来也是为了更加的"安全"

**如何做？**

利用proxychains工具，配置/etc/proxchains.conf文件:

添加:

socks5 127.0.0.1 2020

添加的ip和端口就是你SSR的端口哟！，然后呢，用下面的命令启动tor组件:

```bash
root@kali:~# proxychains tor
```

等待一小会儿，tor便会启动，tor的监听端口在9050，也就是说，在浏览器或者其他要使用tor代理的地方，配置其代理端口为9050就可以了,你是这样上网的:

本机-->本地ssr代理端口(如2020)-->Tor监听端口9050-->Tor的全球节点服务器

- 如何跨机器共享Tor Browser 的代理通道

打开 Tor 的配置文件/etc/tor/torrc,添加:

```
SocksListenAddress 0.0.0.0:9050
```

修改完记得保存，然后记得重启 Tor。

- 如何监控Tor流量

安装nyx:

```bash
root@kali:~# apt-get install nyx
```

然后需要修改一些配置，打开/etc/tor/torrc，添加以下内容:

```
ControlPort 9051 
CookieAuthentication 1
```

保存，再重启Tor，运行nyx即可。