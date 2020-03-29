# Github的正确食用方式

Thanks~prgthk

- 浅谈使用SSH方式操作Github的项目（以linux为主）

> 1.安装openssh

大部分知名的 Linux 发行版，其官方的软件库中都已经内置了【openssh】这个软件，你只需用该发行版提供的软件包管理器，一个命令就可以把 openssh 装好。

> 2.创建"公钥/私钥对"

使用如下命令创建：

```bash
ssh-keygen -t rsa -b 4096 -f 文件路径 -C 邮箱地址
```

解释一下：

其中的 `-t rsa` 表示加密算法的类型是 RSA
其中的 `-b 4096` 表示密钥是 4096 位/比特
(请注意，不同加密算法的位数，没有可比性。对于 RSA 加密算法，如果你没有指定 -b 参数，则默认值是 1024；以目前的破解水平，2048 应该够安全了。为了保险起见，咱们这里创建 4096 比特的密钥)

> 4.密钥文件的存放

上述命令中的【文件路径】表示：【私钥】文件存放的位置。然后 `ssh-keygen` 会自动在这个路径末尾附加 `.pub` 作为【公钥文件】的路径

比如说：你输入的私钥文件路径是 `~/.ssh/xxxx` 那么生成之后对应的公钥文件的路径就是 `~/.ssh/xxxx.pub`

如果你要创建不止一个“公钥私钥对”，要使用具有一定可读性的文件名，以免自己搞混了。

【私钥文件】非常重要，【不要】泄漏给外人。

> 5.在GIthub上指派密钥

GitHub 支持两种方式的 SSH key，分别是【用户级】和【项目级】。

所谓的【用户级】就是说，使用该 key 可以操作所有项目的代码仓库；而【项目级】的 key 只能操作所属项目的代码仓库。

要设置【用户级】的 key，需要先进入“User Profile”页面，然后进入“SSH keys”页面，点击“New SSH key”按钮。

要设置【项目级】的 key，需要先进入该项目的“Settings”页面，然后进入“Deploy keys”页面，点击“Add deploy key”按钮。

点击完上述按钮之后，你需要把【公钥文件】的内容，【**原封不动地**】 copy & paste 到页面中的那个多行文本框里面。

**注意事项：**

1.别搞错文件了，需要复制的是【公钥文件】的内容！

2.粘贴的时候要小心，【不要】加入多余的空格或多余的回车。

> 6.测试SSH登录

如果你当前的系统无需代理就可以访问 GitHub，那么你可以先用命令行测试一下刚才配置好的那个 SSH。运行如下命令：

```bash
ssh -i 公钥文件路径 -T git@github.com
```

运行之后，可能会出现如下提示：

```bash
RSA key fingerprint is nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)?
```

你输入 `yes` 以便继续。之后你会看到一句 You've successfully authenticated 就表示该 ssh key 已经通过登录认证。

> 7.修改配置SSH的配置文件

​	测试通过之后，你可以修改 SSH 的配置文件，就不用每次都在命令行参数中指定密钥文件的路径了。如果你用 SSH 方式操作2个以上的项目，很有必要进行如下定制，可以节省很多命令行输入。
​	SSH 配置文件的路径是： `~/.ssh/config`
​	如果你系统中没有这个文件，就创建一个。然后用你熟悉的文本编辑器修改这个文件。

​	下面给出一个示例：
 你需要把示例中的“别名”改为你自己起的名字（用一个可读性好点的名字）；
 把“私钥文件路径”也同样替换为你本机所使用的文件路径。
 其它部分【不要】改动。

```bash
Host 别名
  HostName                  ssh.github.com
  Port                      443
  User                      git
  PreferredAuthentications  publickey
  IdentityFile              私钥文件路径
```

 　所谓的“别名”，用来替换 URL 中的主机名。比如那个 zhao 项目，我在 `~/.ssh/config` 中使用的别名就是 zhao
 　之后我如果要 clone 该项目，只需用如下命令：

```bash
git clone ssh://zhao/programthink/zhao
```

看到没？在 URL 中就不写 github.com 而改为 zhao，那么 openssh 就会去 `~/.ssh/config` 中找到 zhao 这个配置项，然后用对应的私钥文件进行 SSH 连接

> 8.后续操作

前面几个步骤都搞定之后，你就可以通过 ssh 协议把某个项目 clone 到本地，修改完，最后再 push 到 GitHub 上。整个过程都是通过【强加密的】 SSH 协议完成。

- 浅谈安全性(代理)

> C/S方式下的HTTPS协议

 　对于这种方式，你需要修改 Git 的配置参数，让 Git 知道 Tor 代理的 IP 和 端口。
 　具体的配置命令如下：

```bash
git config --global http.proxy SOCKS5h://代理地址:端口号
```

> C/S方式下的SSH协议（**侧重于这个**）

​	要让 SSH 通过 Tor 的代理，稍微麻烦一点。因为 Tor 默认提供的是 SOCKS 代理，而 OpenSSH 客户端默认又【不】支持 SOCKS 代理。
 　因此，得依靠第三方的工具，来实现“SSH through SOCKS”。
 　这里要提醒一下列位看官：
 　这个说的是“SSH through SOCKS”，而【不是】“SOCKS through SSH”（这两者完全不同）

​	为了搞定“SSH through SOCKS”，我选用大名鼎鼎的 netcat（俗称“网猫”）。
 　由于这个 netcat 名气很大，主流 Linux 发行版的软件仓库中都有它。你只需要用发行版自带的软件包管理器，把 netcat 装上。
 　说到 netcat，有一个“原版的”以及非常多的【变种】。“原版的 netcat”【不】支持代理，必须用某些支持代理的【变种】。我推荐的是  OpenBSD 社区开发的 netcat 变种（也叫“OpenBSD netcat”或“netcat-openbsd”）。
 　如何判断你是否装对了捏？
 　在装好 netcat 之后，先运行如下命令（命令中的 nc 就是 netcat 的缩写）。如果输出的第一行能看到 OpenBSD 这个单词，就说明你装对了。

```
nc -h
```

接下来，用如下命令测试“SSH through Tor SOCKS”是否成功。

```bahs
ssh -o "ProxyCommand=nc -X 5 -x 代理地址:端口号 %h %p" -T ssh.github.com
```

**注意1**：
 假如你的 Tor 客户端运行在【**本机**】，那么上述命令中的“代理地址”就替换为 `127.0.0.1`
 否则就替换为：运行 Tor 客户端的主机的 IP 地址。
 **注意2：**
 如果你的 Tor 客户端用的是 **Tor Browser**，“端口号”必须用 `9150`
 如果你用的是 Tor 的其它软件包（比如：Tor Expert Bundle），则“端口号”使用 `9050`

　　上述测试命令如果最终显示 Permession denied 就说明已经通过 SOCKS 代理连接到 GitHub 了（也就是说，你的 SSH 已经能够走 SOCKS 代理联网了）。
 　	如果没有显示这个信息，而是显示了其它其它信息，你再用如下命令重新试一次

```
ssh -o "ProxyCommand=nc -X 5 -x 代理地址:端口号 %h %p" -Tv ssh.github.com
```

这次加了一个 v 选项，可以打印出详细的诊断信息（不过都是洋文）。如果你略懂洋文并略懂网络技术，或许能判断出错的原因所在。

 　搞定之后，为了方便起见，同样可以把 SSH 的这个 `ProxyCommand` 命令行选项加入到 SSH 的配置文件。如此一来，以后每次你要连接 GitHub 的服务器，都会自动走 Tor 提供的 SOCKS 代理。
 　前面已经给出了 SSH 配置文件的示例，我把之前那个示例文件，加上 `ProxyCommand` 选项之后，变为如下:

```
Host 别名
  HostName                  ssh.github.com
  Port                      443
  User                      git
  PreferredAuthentications  publickey
  IdentityFile              私钥文件路径
  ProxyCommand              nc -X 5 -x 代理地址:端口号 %h %p
```

 　**注意1：**
 　假如你的 Tor 客户端运行在【本机】，那么上述命令中的“代理地址”就替换为 `127.0.0.1`
 　否则就替换为：Tor 客户端所在主机的 IP 地址。
 　**注意2：**
 　如果你的 Tor 客户端用的是 **Tor Browser**，“端口号”必须是 `9150`
 　如果你用的是 Tor 的其它软件包，则“端口号”使用 `9050`

- 其他注意的事项

> 邮件地址的设置

​	最好不要公开你的邮箱，这样可以避免一些不必要的风险。
​	你可以在 GitHub 的用户配置界面中，设置自己的邮箱地址为“私密”。具体操作步骤参见 GitHub 官方的帮助（链接在“[这里](https://help.github.com/articles/keeping-your-email-address-private/)”）
​	由于前面介绍了基于 SSH 客户端操作 GitHub，当你把邮箱地址设置为【私密】之后，你记得设置一下 git 客户端的 email 选项（命令如下）：

```
git config --global user.email 名称@users.noreply.github.com
```

​	把上述命令中的“名称”替换为你在 GitHub 上的帐号名。
​	经过上述命令设置之后，别人看你的提交历史，看到的是类似与 `xxxx@users.noreply.github.com` 这样的邮箱地址，看不到你真实的邮箱地址。

- **Github同步文件了啦!**

首先要进入项目目录哟：

```bash
root@kali:~# git add 文件名
```

```bash
root@kali:~# git commit -m 'the commit'
```

初次使用，要同步到的Repository,那么先创建一个test,如果以后还是要上传到这个Repository，那么就不需要再创建了，只需要引用创建好了的test即可。

```bash
git remote add test git@github.com:xxxx/xxxxx.git
```

最后，执行同步命令:(已经创建了test就不必再执行上一步的命令了，直接执行下面这条同步)

```
git push -u test master
```

> 更新被拒绝，因为远程仓库包含您本地尚不存在的提交。这通常是因为另外
>
> 一个仓库已向该引用进行了推送。再次推送前，您可能需要先整合远程变更

解决办法：现将远程库pull到本地更新同步一下，再push

```bash
root@kali:~# git pull
root@kali:~# git push -u test master
```

**如何防止信息泄漏(Github)**

- 在 add 阶段不要 `git add .` ，而是尝试使用 `git add --interactive` 来审核每一个文件
- 在 commit 前通过 `git diff --cached` 仔细检查文件
- 每个特性使用自己的 branch 并且持续往 master 分支合并，合并前需多人审核