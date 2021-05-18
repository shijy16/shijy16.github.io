---
title: Hexo博客搭建
date: 2021-05-13 11:48:00
tags: 博客
categories: 配置
description: Hexo博客搭建教程。
---

## Hexo环境安装

### 安装node.js

下载安装[node.js](https://nodejs.org/)

更换国内源:

```
npm config set registry https://registry.npm.taobao.org
```

### 安装Hexo

```
mkdir blog && cd blog	#创建hexo目录
npm i hexo-cli -g		#安装hexo
hexo -v					#验证是否成功
hexo init				#初始化
npm install				#安装必备库
```

### 网页基本操作

```
hexo g				#生成网页
hexo s				#创建本地服务器
hexo clean			#clean
hexo new post "article title"	#创建新文章
```

至此，hexo基本环境安装完成。

## next主题安装和背景设置

### 安装next

下载next主题:

```
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

在`_config.yml`中修改`theme`属性为:

```
theme: next
```

### 设置背景图片

- 将背景图片`bg.jpg`复制到`themes\next\source\images`中

在`themes/next/_config.yml`中取消注释:

```
style: source/_data/styles.styl
```

并在根目录下的`source`文件夹创建`_data/styles.styl`，在`styles.styl`中添加:

```
body {
 	background:url(/images/bg.jpg);
 	background-repeat: no-repeat;
    background-attachment:fixed;
    background-position:50% 50%;
    background-size: cover;
}
```

背景图片会铺满整个页面

### 设置文章透明度

默认所有内容背景透明，看不清字，所以需要设置文章背景透明度:

```
.main-inner { 
    margin-top: 60px;
    padding: 60px 60px 60px 60px;
    background: rgba(255,255,255,0.9);
    min-height: 500px;
}

//博客内容透明化
.content-wrap{
  opacity: 0.9;
}

//侧边框的透明度设置
.sidebar {
  opacity: 0.9;
}

//菜单栏的透明度设置
.header-inner {
  background: rgba(255,255,255,0.9);
}

//搜索框（local-search）的透明度设置
.popup {
  opacity: 0.9;
}
.footer-inner {
  background: rgba(255,255,255,0.9);
}
```

## 主题相关设置

### 修改整体scheme

默认Muse有点丑，在`next`文件夹下的`config`中改为其他`scheme`:

```
# Schemes
# scheme: Muse
# scheme: Mist
scheme: Pisces
# scheme: Gemini
```

这里直接修改后在本地看起来没问题，部署到`github.io`上侧边栏就会出错，查了一下发现是设置里自带了`sidebar`属性，两个会冲突，所以修改`next`的配置文件中的`sidebar`为`hide`:

```
sidebar:
  display: hide
```

问题解决。

因为该scheme自带了一个`content-wraper`，所以透明度需要修改，把 `themes\next\source\css\_schemes\Pisces\_layout.styl` 文件 `.content-wrap` 标签下 `background: white`改为完全透明:

```
background: rgba(255,255,255,0);
```

然后根目录下`source/_data/_styles.styl`中的透明度设置:

```
.main-inner { 
    margin-top: 60px;
    padding: 60px 60px 60px 60px;
    background: rgba(255,255,255,0.9);
    min-height: 500px;
}

//侧边框的透明度设置
.sidebar {
  opacity: 0.9;
}

//菜单栏的透明度设置
.header-inner {
  background: rgba(255,255,255,0.9);
}

//搜索框（local-search）的透明度设置
.popup {
  opacity: 0.9;
}

.footer-inner {
  background: rgba(255,255,255,0.8);
}
```

### 添加分类和标签

创建分类和标签页面:

```
hexo new page categories
hexo new page tags
```

分别编辑 `categories/index.md` 和 `tags/index.md` 文件，将内容替换为：

```
---
title: 分类
date: 2021-05-10 12:22:12
type: "categories"
---
```

和

```
---
title: 标签
date: 2021-05-10 12:21:48
type: "tags"
---
```

配置`themes/_config.yml`中的菜单属性:

```
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

完成后就可以设置文章的属性和标签了:

```
title: Hexo博客搭建
date: 2021-05-11 22:38:54
tags: 教程
categories: 其他
```

多标签:

```
tags: [教程,其他]
```

### 限制目录深度标题

修改`themes/_config.yml`中的`max_depth`属性为`3`

### 侧边栏社交信息

修改`themes/_config.yml`中的`social`属性

### 首页不显示全文

在每个文章头添加`description`属性。

### 更改网站图标

找一张想做图标的图，在[bitbug](http://www.bitbug.net/)里生成`16x16`和`32x32`的各一张，然后放到`next`里面的`source/images`文件下。

然后配置next主题的`config`文件，修改favicon:

```
favicon:
small: /images/my16x16.png
medium: /images/my32x32.png
```

### 更改个人头像

把放到`next`里面的`source/images`文件下，然后配置next主题的`config`文件，修改avatar:

```
avatar:
  # Replace the default image and set the url here.
  url: /images/avatar.jpg
```

##### 更改网站签名

在根部录下的`config`文件中，修改`description`:

```
description: 'I am a noob.'
```

## 功能相关设置

### 添加本地搜索功能

在hexo的根目录下执行命令：`npm install hexo-generator-searchdb --save`

在根目录下的`_config.yml`文件中添加配置：

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

在修改`next`目录下的`config`文件中的`local_search`为`enable`：

```
local_search:
  enable: true
```

### 访客统计

修改`themes/_config.yml`中的`busuanzi_count`为`enable`

### 添加评论功能

##### Valine

使用**Valine 评论系统**，访客不需要登录即可评论，而且支持markdown。

首先注册一个账号:[LeanCloud官网登录入口](https://leancloud.cn/dashboard/login.html#/signin)。注册后访问控制台，创建开发版应用，在设置中获取 `App ID` 和 `App Key`。在next的config文件中，修改`valine`相关配置:

```
# Valine.
# You can get your appid and appkey from https://leancloud.cn
# more info please open https://valine.js.org
valine:
  enable: true # 是否开启
  appid:   # 上一步获取的 App ID
  appkey:  # 上一步获取的 App Key
  notify: false # 新留言是否需要通知 https://github.com/xCss/Valine/wiki
  verify: false # 是否需要验证，验证比较反人类建议false关闭
  placeholder: 请在此输入您的留言 # 默认留言框内的文字
  avatar: mm # 默认头像
  guest_info: nick,mail # 默认留言框的头部需要访问者输入的信息
  pageSize: 10 # pagination size #默认单页的留言条数
```

##### gitalk

后面了解到还有gitalk评论系统，要求用github账户登录评论，基于仓库的issue实现，感觉这个更靠谱，就改成了这个。

注册gitalk应用，到[github申请页面](https : //github.com/settings/applications/new)申请，名称和描述随意填，两个url填自己博客地址，我的是`https://blog.shijy16.cn`，注册后，获得ID和Secret。

在next的config文件中，修改`gitalk`相关配置:

```
gitalk:
  enable: true
  github_id:  # GitHub repo owner
  repo:  # Repository name to store issues
  client_id: # GitHub Application Client ID
  client_secret: # GitHub Application Client Secret
  admin_user: # GitHub repo owner and collaborators, only these guys can initialize gitHub issues
  distraction_free_mode: true # Facebook-like distraction free mode
  # Gitalk's display language depends on user's browser or system environment
  # If you want everyone visiting your site to see a uniform language, you can set a force language value
  # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
  language:
```

repo一般就直接填博客仓库名称就好了，`github_id`、`admin_user`就填自己的github id。然后记得把其他评论配置disable。

最后部署后，自己进入博客页面，查看文章末尾评论，应该会提示`issues not initailized`，点进去授权一下就可以了，在博客仓库里面的issue中看到gitalk自动创建的issue就代表配置成功。

### 部署到github

修改根目录下的配置文件最后一行的配置为:

```
deploy:
  type: git
  repository: https://github.com/yourname/yourname.github.io
  branch: main
```

安装部署插件:

```
npm i hexo-deployer-git
```

部署:

```
hexo d
```

### 插入PDF

因为我用latex写的大部分笔记，所以有在网页中插入pdf这个需求。

在hexo中插入pdf有两个方案，第一种是用插件，我失败了，这里就不讲了(不管插入本地PDF还是远程PDF)。

这里讲第二种方案，用html。

首先在要插入文章的同级目录中创建文章同名文件夹，然后把PDF放进去:

```
-CTF学习笔记
	-note.pdf
-CTF学习笔记.md
```

然后在markdown中插入:

```
<object data="./note.pdf" type="application/pdf" width="100%" height="900px">This browser does not support PDFs. Please download the PDF to view it: <a href="/index.pdf">Download PDF</a>
</object>
```

这里我还把根目录的`config`文件中的url属性改为了:

```
url: https://shijy16.github.io
```

然后就可以在博客中正常预览PDF了。

> PS: 尝试用远程PDF链接失败，用其他仓库中的PDF链接时，会直接弹出下载窗口，无法直接预览。
>
> 要直接预览的话需要为仓库创建github pages，但这样之后又不能方便更新了，和把pdf放到github.io仓库一样。
>
> 有解决方案的同学可以通过email联系我在后面评论。

### 个人域名重定向和HTTPS

首先在腾讯、阿里等域名代理商购买一个个人域名，我购买的是`shijy16.cn`，接下来对域名进行设置，使`blog.shijy16.cn`被解析到`shijy16.github.io`:

- 添加CNAME记录，在域名控制台添加CNAME，将`blog.shijy16.cn`指向`shijy16.github.io`
- 修改github pages设置，把`Custom domain`设为``blog.shijy16.cn`

一段时间后可以正常从`blog.shijy16.cn`访问。

由于直接访问不是HTTPS链接，浏览器会提示不安全，可以用[CloudFlare](https://www.cloudflare.com/) 提供的服务来强制使用HTTPS。

- 注册CloudFlare账号，注册成功后在返回的页面中添加域名，点击扫描 DNS 记录，等待大约一分钟之后继续下一步。
- 添加域名解析，即把域名指向`github.io`，和之前的CNAME记录一样。
- 修改域名服务商中的DNS服务器为[CloudFlare](https://www.cloudflare.com/) 的DNS服务器。
- 在[CloudFlare](https://www.cloudflare.com/)引导下完成剩余步骤，重点是强制使用HTTPS访问。

这一部分[CloudFlare](https://www.cloudflare.com/)中有详细引导。完成之后，直接访问[blog.shijy16.cn](https://blog.shijy16.cn/2021/05/11/配置/Hexo博客搭建/blog.shijy16.cn)就是HTTPS连接了。

## 参考文章

> [hexo目录设置](https://www.dazhuanlan.com/2019/11/06/5dc26196564d8/)
>
> [hexo+github博客搭建教程](https://zhuanlan.zhihu.com/p/35668237)
>
> [next主题配置透明色等](https://blog.qsong.fun/2018/01/31/next主题配置透明色等/)
>
> [hexo页脚添加访客人数和总访问量](https://chrischen0405.github.io/2018/09/11/post20180911/)
>
> [超详细Hexo+Github博客搭建小白教程](https://zhuanlan.zhihu.com/p/35668237)
>
> [hexo的next主题个性化配置教程](http://shenzekun.cn/hexo的next主题个性化配置教程.html)
>
> [Hexo 博客创建 categories 和 tags 页面](https://xring.info/2018/hexo-category-and-tag-page.html)
>
> [hexo中插入pdf解决方法](http://miracle778.site/pdf-test/pdf-test.html)
>
> [为 Github Pages 自定义域名博客开启 HTTPS](https://maiyang.me/post/2018-04-09-using-https-with-custom-domain-name-on-github-pages/)
>
> [hexo - Next 主题添加评论功能](https://yashuning.github.io/2018/06/29/hexo-Next-主题添加评论功能/)
>
> [更换Hexo的网页图标/小图片Hexo change page favicon](https://blog.csdn.net/Olivia_Vang/article/details/92976637)
>
> [Hexo博客Next主题配置](https://blog.winsky.wang/Hexo博客/Hexo博客Next主题配置/)
>
> [Hexo解决页面过小问题与设置透明背景](https://www.cnblogs.com/Mayfly-nymph/p/10622307.html)