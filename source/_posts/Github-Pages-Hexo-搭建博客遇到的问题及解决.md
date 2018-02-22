---
title: Github-Pages-Hexo-搭建博客遇到的问题及解决
categories: 编程
tags:
  - php
  - 格式化输出
translate_title: githubpageshexo-build-blog-problems-encountered-and-solutions
date: 2016-07-12 14:37:11
---

本文基本是以Mac OS作为环境搭建，有部分摘自
http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2

这里对作者表示感谢！

## Github Pages环境搭建
这几天恶补了下Git基础知识，推荐看  [廖雪峰的Git基础教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000),说的比较通俗，比较浅显易懂，看完后最好自己动手实践下是最好的。下面主要说说在链接Github中遇到的问题和解决方式

### 如何添加Git代理
Git 目前支持的三种协议 `git://`、`ssh://` 和 `http://`，其代理配置各不相同：core.gitproxy 用于 git:// 协议，http.proxy 用于 http:// 协议，ssh:// 协议的代理需要配置 ssh 的 ProxyCommand 参数。

#### git协议代理

建立 /path/to/socks5proxywrapper 文件，使用 https://bitbucket.org/gotoh/connect 工具进行代理的转换，各发行版一般打包为 proxy-connect 或者 connect-proxy。
>/path/to 可以为你的本机路径，比如/usr/local/bin

```bash
#!/bin/sh
connect -S 127.0.0.1:7070 "$@"
```

配置git

```bash
git config --global core.gitproxy /path/to/socks5proxywrapper
```
或者设置成环境变量

```bash
export GIT_PROXY_COMMAND="/path/to/socks5proxywrapper"
```
#### HTTP协议代理配置
http协议的配置比较简单，使用场景一般有2种：

* 内网 到 外网的代理
* Github无法访问被墙了
内网到外网的代理比较简单，只要一个内外网通吃的服务器做个跳板即可，下面主要说下Github扶墙的方法。本人使用的是Lantern

1. Lantern代理

```bash
git config --global http.proxy http://127.0.0.1:8787
git config --global https.proxy https://127.0.0.1:8787
```

#### SSH协议代理配置
博主购买的Bandwagon的VPS做的Shawdowsocks做的socks5代理

> 主要方法：使用polipo做映射，将socks5->http来进行处理，如果使用的是Lantern，道理是一样的，只要将本地端口改成8787即可。然后通过corkscrew进行http->ssh的转换，实现SSH协议代理

1. 首先安装Homebrew，具体方法见[这里](http://brew.sh/),如果已经安装过的请看下一步

2. 安装corkscrew，corkscrew是一个能让你通过http代理，使用SSH客户端进行SSH隧道连接的软件

```bash
brew install corkscrew
```

3. 安装polipo，我们通过polipo实现socks5->http 代理的转换

```bash
brew install polipo
```
这里我们要对polipo做一点配置，将其设置为自启动模式，这样每次开机后就自动开启了这个服务。

进入到polipo的配置list文件，博主的在
`/usr/local/Cellar/polipo/1.1.1`下，编辑文件

```bash
vi homebrew.mxcl.polipo.plist
```

加入一行string的配置

```xml
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>homebrew.mxcl.polipo</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/local/opt/polipo/bin/polipo</string>
      <string>socksParentProxy=localhost:1080</string>
    </array>
    <!-- Set `ulimit -n 65536`. The default OS X limit is 256, that's
         not enough for Polipo (displays 'too many files open' errors).
         It seems like you have no reason to lower this limit
         (and unlikely will want to raise it). -->
    <key>SoftResourceLimits</key>
    <dict>
      <key>NumberOfFiles</key>
      <integer>65536</integer>
    </dict>
  </dict>
</plist>
```
设置每次启动登陆polipo

```bash
ln -sfv /usr/local/Cellar/polipo/1.1.1/*.plist ~/Library/LaunchAgents
```
这样就配置号了，下面对`~/.ssh`下新增编辑config文件

```bash
cd ~/.ssh
vi config
```
填写如下内容

```bash
Host github.com
User username   #这里填写你的Github注册邮箱或者账户名均可
Hostname ssh.github.com
PreferredAuthentications publickey
#polipo默认监听的是8123端口，也可以在刚才的plist文件指定
#通过corkscrew进行http代理->ssh的代理配置
ProxyCommand corkscrew 127.0.0.1 8123 %h %p
IdentityFile ~/.ssh/id_rsa
Port 443
```

这样就实现了`socks5->http->ssh`的转换配置

如果要测试是否连接成功，可以输入

```bash
ssh -T git@github.com
```
界面返回

```bash
Hi XXX! You\'ve successfully authenticated, but GitHub does not provide shell access.
```
即为连接成功~

## hexo环境搭建
主要环境搭建请移步 https://hexo.io/zh-cn/docs/
这里主要描述下如何将Github Pages+Hexo集成的

#### 博客搭建流程
1. 创建仓库，cometlj.github.io；
2. 创建两个分支：master 与 hexo；
3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；
4. 使用git clone git@github.com:cometlj/cometlj.github.io.git拷贝仓库；
5. 在本地cometlj.github.io文件夹下通过Git bash依次执行

```bash
hexo init
npm install
npm install hexo-deployer-git --save
```
>**注意！！**这里有一个大坑，因为`git clone` 的时候，已经将写`.git`文件夹写入了，但是通过`hexo init`会**先清空所有的文件，这样会导致git仓库损坏**，网上找了找了半天也没有解决办法，这里最好在做`hexo init`操作之前，先把`.git`文件夹备份出来，执行完`hexo init`操作后，再挪回来即可。

6.修改_config.yml中的deploy参数，配置hexo的git部署地址，分支应为master；例如：

```bash
deploy:
  type: git
  repo: git@github.com:xxx/xxx.github.io.git
  branch: master
```
依次执行

```bash
git add .
git commit -m “…”
git push origin hexo
```

提交网站相关的文件，这里可以使用`.gitignore`文件将没必要上传的文件过滤，例如：

```bash
vi .gitignore
```
写入

```bash
/.deploy_git
/public
/_config.yml
```

这样每次hexo分支上就不会有多余的其他文件了
6. 执行hexo generate -d生成网站并部署到GitHub上。

这样一来，在GitHub上的CrazyMilk.github.io仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。


#### 博客管理流程
**日常修改**
在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理：
依次执行

```bash
git add .
git commit -m “…”
git push origin hexo
```
hexo指令将改动推送到GitHub（此时当前分支应为hexo）；
然后才执行hexo generate -d发布网站到master分支上。
虽然两个过程顺序调转一般不会有问题，不过逻辑上这样的顺序是绝对没问题的（例如突然死机要重装了，悲催….的情况，调转顺序就有问题了）。

**本地资料丢失**
当重装电脑之后，或者想在其他电脑上修改博客，可以使用下列步骤：
使用

```bash
git clone git@github.com:cometlj/cometlj.github.io.git
```
拷贝仓库（默认分支为hexo）
在本地新拷贝的cometlj.github.io文件夹下通过Git bash依次执行下列指令：

```bash
npm install
npm install hexo-deployer-git --save
```
（*记得，不需要`hexo init`这条指令*）

OK,现在可以使用Hexo编写你的博客了