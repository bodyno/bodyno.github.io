---
layout: post
title:  "如何在Linux上部署node应用"
date:   2016-05-27
tags: [node, linux]
categories: Full-Stack
---

翻译自[https://certsimple.com/blog/deploy-node-on-linux](https://certsimple.com/blog/deploy-node-on-linux)

> 关于如何“如何在Linux上部署node应用”，自从2013年以来，就是最流行的言论。本文的版本已经更新到适用于node的长期支持版本，当前Linux的发行版，添加了SSL终端说明，以及一些小调整——Mike。

本文就是关于如何在现代的Linux上部署node应用的。阅读完之后，你的成就将会从“我已经写好了一个node应用”提升到“我已经部署了一个node应用”。

如果你是一个web开发者，但不是一个运维人员，那么下面这些就是做运维需要用到的：

- 长期支持的node版本，并且安装在长期支持的Linux版本上
- 你的node应用包含一个可以自动启动，登录以及其它便捷的功能的服务
- 你的应用可以通过443和80端口访问，但是用户权限为普通用户
- 可以使用SSH key作为你的用户名来自动登录，并且有sudo操作权限
- 可以使用deploy keys来进行自动化部署，并且不需要密码
- 允许SSL，并且在负载均衡上将其设置为默认

下面的步骤只需要一些基本的shell技能，并且机器上可以运行SSH。我们并不关心你使用的是Debian/Ubuntu或者Red Hat/CentOS，对于这两种，我们都有相关的命令。

这只是基础，你完全可以做一些更深入的东西，比如使用Ansible playbooks，Dockerfiles，使用你自己的云API来建立部署软件或者建立你自己的云平台等。但是，在你开始深入之前，你必须了解如何来启动一个服务器并且让他运行，这才是本文的目的所在。

---

### 安装一个操作系统，这是node需要的一部分

- 安装一个长期支持的发行版，比如RHEL/CentOS7，Debian 8 stable (Jessie)。使用Ubuntu 16.04代替Ubuntu 14.04，因为14.04使用了比较旧的shell脚本，下面的指南在这上面没法用
- 无论选择哪个发行版，最好是用64位版本的操作系统，这样才能充分使用你的内存
- 安装所有的更新：

{% highlight bash %}
# RHEL/CentOS
$ yum -y update
{% endhighlight %}

{% highlight bash %}
# Debian / Ubuntu
$ apt-get update; apt-get upgrade
{% endhighlight %}

- 安装开发工具，node需要编译二进制文件，比如`npm install node-sass`需要编译器来编译libsass，所以，只需要这样：

{% highlight bash %}
# RHEL / CentOS
$ yum -y group install "Development Tools"
{% endhighlight %}

{% highlight bash %}
# Debian / Ubuntu
$ sudo apt-get install build-essential
{% endhighlight %}

### 让你的团队成员可以使用各自的公钥来登录

你的云平台可能已经设置了帐户以及公钥的访问权限，比如亚马逊云服务使用`sudo`创建了`ec2-user`用户，并且你的公钥已经通过了认证，可以登入这个用户。如果是这样的话，可以跳过这一步，继续下面的步骤。

但是，一些云平台(比如Digital Ocean)只是给了你一个root用户权限。使用root用户登录是不安全的做法。所以，首先，每个人都知道了用户名，就来创建一个普通用户：

{% highlight bash %}
$ user add myaccount
{% endhighlight %}

Wheel是一个Unix概念，来自于[‘big wheel’ meaning a powerful person](https://en.wikipedia.org/wiki/Wheel_%28Unix_term%29)。在大多数的情况下，就是指你的管理员用户组。将刚才创建的用户添加到wheel用户组中。

{% highlight bash %}
$ usermod -G wheel myaccount
{% endhighlight %}

如果你在使用Github，你可以在Github的ssh keys里找到你的用户的公钥的一个副本。

否则，让你的团队的每一个成员都要求一份它们的公钥。如果他们不知道什么，在Mac或者Linux上，可以通过查看`~/.ssh/id_rsa.pub`文件来得到他们自己的公钥。如果没有这个文件的话，运行`ssh-keygen`来生成一个新的公钥。

确保每个人给你的仅仅是公钥，而不是私钥，否则，他们需要重新生成所有东西。

拿到这些key了吗？把这些key复制，然后，粘贴到`~/.ssh/authorized_keys `文件里，每一个`key`就是一行。把这些公钥放进认证文件里，就可以允许有着相应的私钥的用户登录了。

让wheel组里的用户使用sudo时不需要重复的输入密码。编辑`sudoers`文件的时候，有一点特殊的地方需要注意，如果你把这个文件搞乱了，那么你自己也没辙了。所以运行：

{% highlight bash %}
$ visudo
{% endhighlight %}

这个命令只是使用编辑器打开你的sudo文件，并且在你保存之前检查你的语法，将这一行注释掉：

{% highlight bash %}
%wheel ALL = (ALL) NOPASSWD: ALL
{% endhighlight %}

现在，试试从其他电脑上通过普通用户进行登录。你应该可以通过你的公钥进行登录，而且不需要密码，并且可以运行下面这条命令来列出你的权限：

{% highlight bash %}
$ sudo -l
{% endhighlight %}

是不是就是说完了呢？很好，你现在有管理员账户了，你可以这样运行其他命令了：

{% highlight bash %}
$ sudo someCommand
{% endhighlight %}

让我们来禁止root用户通过SSH登录。

编辑`/etc/ssh/sshd_config`，找到`PermitRootLogin`这一行，并且把它改为`no`，然后运行来使更改起效：

{% highlight bash %}
$ systemctl reload sshd
{% endhighlight %}

---

### 将时区设置到UTC

如果想要知道为什么要将服务器时区设置为UTC，阅读这篇文章吧：[ want to set the time on your server to UTC](http://serverfault.com/questions/191331/should-servers-have-their-timezone-set-to-gmt-utc)。首先，更新时区：

{% highlight bash %}
$ rm /ect/localtime
{% endhighlight %}

{% highlight bash %}
$ ln -s /usr/share/zoneinfo/UTC /etc/localtime
{% endhighlight %}

要检查更改是否正确，看看时间就知道了：

{% highlight bash %}
$ date
{% endhighlight %}

结果应该和使用UTC的结果一样：

{% highlight bash %}
date -u
{% endhighlight %}

---

### 安装长期支持的node版本

我们安装Node 4这个长期支持的版本。

Node源码工作人员为我们提供了官方的DEB和RPM的安装包，你可以在https://nodejs.org进行下载，根据官方的文档进行安装。

如果你没找到适合你的操作系统的node，看看这个网址http://nodejs.org/download/。下载好了之后，将其解压到`/usr/local`：

{% highlight bash %}
$ tar -xf node-someversion.tgz
{% endhighlight %}

在你的`PATH`变量中配置符号链接：

{% highlight bash %}
$ ln -s /usr/local/node/bin/node /usr/local/bin/node
$ ln -s /usr/local/node/bin/npm /usr/local/bin/npm
{% endhighlight %}

---

### 添加一个`shrinkwrap`文件来持续化部署

npm内建的`shrinkwrap`工具可以确保你指定的依赖不会改变。所以，你在你的生产环境下使用的依赖版本与你在服务器上部署的时候的依赖版本相同：

{% highlight bash %}
$ npm shrinkwrap
{% endhighlight %}

### 配置git

我们将使用git来对我们的应用进行更新。如果你使用的是Github或者Gitlab而不是个人的SSH key，那么，你可以创建一个特殊的`deploy key`，这个key对于你的项目只有读取权限。你可以在机器上这样设置：

{% highlight bash %}
$ ssh-keygen -t rsa -C "myapp@mycompany.com"
{% endhighlight %}

然后，再将这个`deploy key`添加到Github或者Gitlab。

---

### SSL

最近你真的该使用SSL了。

**针对小型应用**

如果你的应用没有超过一个实例，并且，你可以在升级期间将其下线。那么，node内建的HTTPS栈就是最简单的选择。对于小型应用使用node内建的HTTPS栈也就意味着你需要维护的软件更少。而且，与Python和Ruby不同的是，node的时间基于IO，不需要一个单独的时间服务器。

在node4中使用Express的一个经典例子：

{% highlight javascript %}
var server = https.createServer({
    key: privateKey,
    cert: certificate,
    ca: certificateAuthorityCertificate
}, app);
{% endhighlight %}

在[SSL labs](https://www.ssllabs.com/ssltest/)测试这段代码，至少会得到A。

**针对大型应用**

如果你期望使用多个服务器，那么，最好是将SSL放置在负载均衡上。这样，在通过直接的HTTP协议传递进你的某个node服务器之前，就有一个单独的地方来处理SSL了。

- 在亚马逊云里，设置一个ELB(AMS Elastic Balancer)。ELBs是亚马逊云定制的负载均衡应用，你的SSL证书和ky就安装在这里。
- 另外一种方式，使用[HAProxy](http://blog.haproxy.com)，这是一个可以高度利用的代理，它会在处理SSL，引导用户抵达当前活动的服务器。我们待会儿会讲解[HAProxy](http://blog.haproxy.com)，下面是一点简介：

- 我们使用haproxy.com的主被动设置。这样，我们就可以在部署之后，手动检查服务器。但是仅仅是在我们在前一个服务器运行`service myapp stop`之后，才将用户引导到当前的活动服务器。
- SSL在过期后就会改变，可以使用[Mozilla's SSL Config Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/)来获取最新的
- 在使用`systemd`命令的时候，使用`log /dev/log local0 info`来让`haproxy`正常运作。

---

### 让你的应用端口有本地访问权限

切换到`/var/www`目录，然后`git clone`你的应用。

在Unix上，默认只有root用户可以绑定1024以下的端口号。但是，我们并不想使用root来运行我们的app，那样不安全。当然，如果你使用HAProxy或者ELB，那么，你可以跳过这一步，因为HAProxy和ELB会把进来的流量直接导向相关的端口。

所以，如果要让我们的应用能够在1024以下的端口运行，我们需要给他一个特权：

{% highlight bash %}
$ sudo setcap 'cap_net_bind_service=+ep' $(readlink -f $(which node))
{% endhighlight %}

你是不是在想为什么要使用`readlink`，因为`setcap`需要真正的文件来运行，而不只是使用一个链接。你的文件系统将会保存下这个特权，但是，当node更新的时候，你需要重新运行这条命令，就是这样，当然，重启之后依然可以运行。

---

### 安装，启动服务

在所有的Linux发行版上，都使用`systemd`来进行服务操作。意味着，我们不需要再另外写shell脚本来查看守护进程，改变用户，清理环境变量，设置自动重启，以及一堆繁琐的事情了。

我们只需要为应用创建一个`.service`文件，让操作系统来关心这些事情。看下面这个`myapp.service`的例子：

{% highlight bash %}
[Unit]
Description=Your app
After=network.target
{% endhighlight %}

{% highlight bash %}
[Service]
ExecStart=/var/www/myapp/app.js
Restart=always
User=nobody
Group=nobody
Environment=PATH=/usr/bin:/usr/local/bin
Environment=NODE_ENV=production
WorkingDirectory=/var/www/myapp
{% endhighlight %}

{% highlight bash %}
[Install]
WantedBy=multi-user.target
{% endhighlight %}

下面介绍更多的配置：

- `After`意思就是我们在`network`服务启动之后再启动
- `ExecStart`就是要运行的应用。在第一行应该指定编译器，比如添加`#!/usr/bin/env node`到`app.js`。同时，要让这个文件是可执行的，要运行`chmod +x app.js`。
- `Environment`这一行配置的是环境变量。设置`NODE_ENV`到生产环境是node标准，用来告诉Express和gulp运行生产环境的配置。

将你的服务文件复制到`/etc/systemd/system`目录下，然后，让`systemd`知道有这个服务：

{% highlight bash %}
systemctl daemon-reload
{% endhighlight %}

启动服务：

{% highlight bash %}
systemctl start myapp
{% endhighlight %}

所有的node控制台输出，都会被记录到以你的`.service`文件名命名的日志文件里。要实时查看输出的话，使用：

{% highlight bash %}
journalctl --follow -u myapp
{% endhighlight %}

除非第一次你是完美运行，否则，你会有各种古怪的问题，比如你忘了运行gulp，文件权限不对等等。所以需要使用`journalctl`来阅读日志，通过日志的信息来修复问题，然后重新启动应用：

{% highlight bash %}
systemctl restart myapp
{% endhighlight %}

你的应用马上就可以运行起来了。

---

### 部署

部署包括了清理当前生成的文件，拉取最新的代码，安装你的`node-shrinkwrap`文件所指定的依赖，然后重启服务：

{% highlight bash %}
git clean -f -d
git pull origin/master
npm install
gulp build
systemctl restart myapp
{% endhighlight %}

如果你每一步都使用`haproxy`来运行主被动设置，你应该部署到没有运行的服务器上面检查。如果我们乐意，我们可以在没有启动的服务器之前运行`myapp stop`来启动服务器。这是一个很普通的技术，可以在升级期间将我们的网站放置于外，并且，很容易回到工作环境。

---

### 说在最后

所以，总结一下我们所做的：

- 长期支持的node，安装在长期支持的Linux上
- 你的node应用包含合适的服务，可以自动启动，合理输出日志等
- 你的应用在443和80端口可以访问，但是，要在你的`.service`文件里配置可以运行的用户
- 能使用你的SSH key来进行登录作为你的账户，并且拥有sudo权限
- 能够通过使用deploy key来使用git部署
- 允许SSL，将其设置在负载均衡上
- 为每一样都设置公钥，这样，访问各处都不需要密码了

现在，我们就得到了一个正在运行的基本的环境了。