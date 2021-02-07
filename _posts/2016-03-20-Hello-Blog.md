---
layout:		post
title:		"Create and Open My Blog"
description: "Hello Blog"
date:		2016-03-21
author:		"PfCStyle"
header-img:	"img/post/2016-03-20/head.jpg"
categories: "杂记"
keywords:
    - Blog
    - Github
    - Jekyll
    - Bootstrap
    - Markdown
---

> It's My First Blog! Go down

这是我第一次自己搭建博客，以前虽然在CSDN上写过，但跟这个比显然有些索然无味了。我使用的这个模板是基于[@Hux](http://huangxuan.me) 分享的模板进行修改替换的，非常感谢！

第一篇博文写什么呢？最后还是决定把自己搭建的过程记录下来。虽然网上已经有很多类似的了，但我仍觉得有些细节需要总结的，不废话了，Begin!

![](/img/post/2016-03-20/blog-create.png)

# 从Github开始

GitHub Pages是免费的静态站点，三个特点：免费托管、自带主题、支持自制页面和Jekyll。本博客即是建立在Github上的，所以下面介绍的也是基于Github的。

### 拥有一个Github账号

首先大家应该有一个Github账号，作为一个程序员，如果你还没有加入Github,那么你显然out了！当然这不是为了赶时髦，github现在已经是全球最大的开源社区了，所以，加入的必要性就不言而喻了。

怎么注册账号呢？呃，这个不写了，推荐一篇博文：[创建GitHub技术博客全攻略](http://blog.csdn.net/renfufei/article/details/37725057/)

好吧，推荐了这篇博文之后感觉基本什么都不用写了。但是我还是有一些自己经验的补充（耍无赖:)）。

### 创建个人博文仓库

Github是这样规定的，以 username.github.io 命名的仓库才可以作为自己的个人（公司）主页，并且此时你的主页代码等等都是在master分支中的，浏览器访问地址是username.github.io (比如我的是：pfcstyle.github.io)。如果你想要为自己的某个项目写文章的话，那么你需要在项目中新建gh-pages分支，浏览器访问地址：username.github.io/yourRepName。

> 自己DIY才有感情吧，也更有成就感噻！

没错，正如你所理解的那样，上面推荐的那篇博文里介绍了如何github帮助你自动生成一个博文分支（包括个人的master和项目的gh-pages），但是，这些都是完全自己新建，自己写代码，自己写样式，自己……总之完全可以自定义咯。

### 本地Git配置

远程的配置好了，该配置本地的了。我这里只介绍windows平台上的，linux/unix步骤也是差不多的。windows上最简单的应该就是这个[github for windows](https://desktop.github.com/) 了，直接下载安装就好。但是这个显然不是我介绍的重点，看下面。

#### 1.安装Git

使用命令行的好处，大概就是装逼了。。。但是我觉得命令行更原生，更能看到一些本质。好了，不装了，来[这里](https://git-for-windows.github.io/) 下载吧。版本你自己选，这个也是有界面的，但是，真的是丑陋无比，反正我是从来没有用过，都是直接用命令行。

安装好就是配置了，你可以添加环境变量，比如我的是D:\Program Files\Git\bin。其实我并没有配置环境变量，因为我觉得根本不需要，安装好之后根目录下有一个git-bash，哪里需要，哪里运行，尤其是自动添加了右键菜单，还有高亮显示，很是方便。

#### 2.配置本地ssh到github

首先你要生成自己本地的sshkey，打开git-bash

{% raw %}

```bash
//邮箱地址后面的是指定生成公钥的文件名，不过不指定，默认生成~/.ssh/id_rsa.pub
ssh-keygen -t rsa -C "邮箱地址" -f ~/.ssh/githug_blog_keys
```

{% endraw  %}

回车之后还会让你设置提交用户名、密码，可以直接跳过，如图：

![](/img/post/2016-03-20/ssh.png)

生成之后，打开你的pub文件（我的是githug_blog_keys.pub，一般在C:\Users\用户名\.ssh\目录下），全选复制里面的内容,来到github进行配置：

![](/img/post/2016-03-20/github-setting.png)

![](/img/post/2016-03-20/ssh-settting.png)

如果你已经使用github生成好了，那么来到你想存放的目录下，右键打开git-bash，clone下来就好了：

![](/img/post/2016-03-20/self-ssh.png)

{% raw %}

```git
git clone 上图中的ssh
```
{% endraw  %}

如果你是完全自定义的……那我相信你一定会搞的

# Jekyll的安装与配置

Jekyll是一种简单的、适用于博客的、静态网站生成引擎。它使用一个模板目录作为网站布局的基础框架，支持Markdown、Textile等标记语言的解析，提供了模板、变量、插件等功能，最终生成一个完整的静态Web站点。说白了就是，只要安装Jekyll的规范和结构，不用写html，就可以生成网站。[ [jekyll介绍](http://jekyllbootstrap.com/lessons/jekyll-introduction.html) ] [ [jekyll on Github](https://github.com/jekyll/jekyll) ][ [jekyllbootstrap](http://jekyllbootstrap.com/) ]

Jekyll使用Liquid模板语言，\{\{page.title\}\}表示文章标题，\{\{content\}\}表示文章内容。我们可以用两种Liquid标记语言：输出标记（output markup）和标签标记 (tag markup)。输出标记会输出文本（如果被引用的变量存在），而标签标记不会。输出标记是用双花括号分隔，而标签标记是用花括号-百分号对分隔[ [Liquid模板语言](https://github.com/shopify/liquid/wiki/liquid-for-designers) ][ [Liquid模板变量参考](https://github.com/jekyll/jekyll/wiki/Template-Data) ]

jekyll与github的关系：GitHub Pages一个由 GitHub 提供的用于托管项目主页或博客的服务，jekyll是后台所运行的引擎。

jekyll安装之前需要先安装DevKit,DevKit是windows平台下编译和使用本地C/C++扩展包的工具。它就是用来模拟Linux平台下的make,gcc,sh来进行编译。但是这个方法目前仅支持通过RubyInstaller安装的Ruby,先下载[RubyInstaller](http://rubyinstaller.org/downloads/) ,直接安装就好了，设置环境变量，path中配置C:\Ruby193\bin目录，然后在命令行终端下输入

{% raw %}

```gem
gem update --system
```

{% endraw %}

来升级gem。这里有可能会遇到问题,因为被墙原因导致ssl错误，这时可以替换一下gem软件源：

{% raw %}

```gem
//查看当前所有软件源
gem sources -l
//自带的源是https://rubygems.org/ 删除
gem source -r https://rubygems.org/
//添加新源
gem source -a http://rubygems.org/
```
{% endraw %}

网上许多解决此问题是通过替换淘宝源http://ruby.taobao.org/成功的，刚开始笔者也是这样干的，但是并没有卵用，最后还是去掉了's'成功的。还有一点需要注意的是，在添加源的时候，你不能在git-bash中添加，会提示：

{% raw %}

```bash
ERROR:  While executing gem ... (Gem::OperationNotSupportedError)
    Not connected to a tty and no default specified
```
{% endraw %}

这是说连接不到终端，所以，你需要使用cmd或者是微软新出的windows powershell来添加源才可以。

然后[下载DevKit](http://rubyinstaller.org/downloads/) ，跟ruby在同一个下载页面。安装后找到DevKit目录，输入以下命令：

{% raw %}

```ruby
ruby dk.rb init
ruby dk.rb install
```

{% endraw %}

这里需要说明的是，你在dk初始化之后,会提示你查看config.yml中的根目录是否正确，而事实上这个文件里根本就没有配置，你需要添加上自己的ruby目录，如下：
{% raw %}

```ruby
---
- D:\Ruby22-x64
```

{% endraw %}

然后才去执行下面的install命令。

好啦，接下来就可以安装jekyll了：

{% raw %}

```gem
//安装 注意，jekyll都是小写，大小写是敏感的
gem install jekyll
//查看版本号，以确定是否安装成功
jekyll --version
```

{% endraw %}

### 安装Jekyll-Bootstrap：

来到你的本地仓库目录，打开git-bash:

{% raw %}

```git
git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com

cd USERNAME.github.com

git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git

git push origin master
```

{% endraw %}

### 启动jekyll服务

{% raw %}

```jekyll
jekyll server --port 4000//4000是默认端口号
```

{% endraw %}

激动人心的时刻到啦，现在去浏览器输入：[http://127.0.0.1:4000](http://127.0.0.1:4000) ，看看自己的本地博客吧。

不过还没结束呢，激动之余，是不是感觉还有点乱呢？分析一下目录：

- **_posts:** _posts中的数据文档，通过注入_layouts定义的模板，通过jekyll server最终生成的静态页面在_sites目录。目录是用来存放你的文章的，一般以日期的形式书写标题。
- **_layouts：** _layouts中的模板一般指向了_includes/themes中的模板。目录是用来存放模板的，在这里你可以定义页面中不同的头部和底部。
- **_includes：** 

> _includes/JB中有一些常用的工具，用于列表显示、评论等；
> _includes/themes中可参看主题的相关html文档。
> _includes/themes中的主题一般包含default.html、post.html和page.html三个文档。default.html定义了网站的最上层框架（模板），post.html和page.html是其子框架（模板）
> 生成好的html子页面通过default.html的\{\{ content \}\}变量调用，生成整个页面。

- **asset:** 渲染页面的CSS和JS文档在assets/themes中。
- **_config.yml:** 站点生成需要用到_config.yml配置文件，站点的全局变量在_config.yml中定义，用site.访问；页面的变量在YAML Front Matter中定义，用page.访问，更多的模板变量可参考模板数据。[jekyll配置详解](http://www.zhanxin.info/jekyll/2013-08-07-jekyll-configuration.html)

当然，并不是一定要这样做，只是一个习惯而已，如果你完全是自定义的，那么你可能都没有这些东西。

# 第一篇博客开始之前

其实博客开始前的工作基本算是说完了，可是我觉得还是要对这个layout进行补充一下：

### 去网上找现成的layout

layout即是你的blog的样式和布局，网上有许多精美的主题供[下载](http://jekyllthemes.org/)

### 自己定义layout

从头自定义的话，无疑很是费劲,所以我建议也像我一样，找到现成的模板，慢慢修改。

# 写第一篇博客

直接使用markdown来写好了，唯一需要说一下的的是文件头的声明格式：

{% raw %}

```markdown
---
layout:		post
title:		"Create and Open My Blog"
description: "Hello Blog"
date:		2016-03-20 17:45:00
author:		"PfCStyle"
keywords:
	- Github
	- Blog
	- Jekyll
	- Markdown
	- CDName
	- layout
---
```

{% endraw %}

看意思应该都明白是什么，不再赘述了，但是，需要提醒大家的是这些不是固定的，不同的layout是有不同参数的。再来张图吧：

![](/img/post/2016-03-20/blog_head.png)

# 最后是找个自己的域名

首先是购买一个域名，域名购买之后，要先设置解析，你购买域名的地方都是可以设置的，过程大致相似：

![](/img/post/2016-03-20/cdname.png)

最后还需要在你的博客分支根目录下添加CNAME文件，里面放上你的域名就可以了，github就会为你自动跳转。

![](/img/post/2016-03-20/cnamefile.png)

![](/img/post/2016-03-20/cname_config.png)

ok,大功告成！



