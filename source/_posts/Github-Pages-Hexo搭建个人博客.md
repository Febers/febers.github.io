---
title: Github Pages+Hexo搭建个人博客
date: 2019-01-11 23:22:07
tags:
- GitHub
- Hexo
categories: 
- 互联网
---

上一个博客是使用 WordPress 搭建在云服务器上的，郁闷的是这个位于美国的服务器 ip 被墙了，无奈放弃。正好最近抽出时间，又迫切地想记录点东西，选择了基本不会被墙的 Github Pages，搭配 Hexo 框架，构建了一个轻量级的博客，在此将其过程记录下来。<!--more-->

作者的环境基于 Windows，其他环境的搭建过程大同小异。

## 准备工作

### Github Pages

创建 Github Pages，首先需要一个 Github 帐号，新建一个仓库，仓库名为`{账户名}.github.io`，系统会自动识别并将其设为 Github Pages。注意的是仓库权限要设置为`public`。之后浏览器打开：`https://{账户名}.github.io`便可以看到缺省的界面。

### Hexo

Hexo 是一个基于 Node.js 的静态博客框架，快速、高效且简洁。需要安装在本地，具体过程如下：
- 安装 Git 客户端，前往 Git 主页下载，安装后登录帐号。
- 安装 Node.js 环境，前往 Node 主页下载。
- 安装 Hexo，新建一个命名任意的文件夹，在其中打开 Git Bash，键入 Hexo 的安装命令：
```bash
npm install -g hexo-cli
```

- 初始化 Hexo，`projectname`省略时，要求当前文件夹为空。
```bash
hexo init [projectname]
```

## 关联

经过前面的过程，已经获得了 Github Pages 和本地的 Hexo 环境，接下来就是关联两者。
在博客的文件夹中有`_config.yml`文件，为 Hexo 的配置文件，打开并将相应位置设置为：

```yaml
deploy: 
type: git
repo: 该处填写仓库的完整路径
branch: master
```
该过程其实是给`hexo d`这一命令做相应的配置，让 hexo 知道 blog 部署的位置，显然部署在 GitHub 的仓库里。
保存文件之后，安装 Git 部署插件，在 Git Bash 中键入命令：

```bash
npm install hexo-deployer-git --save
```

接下来就是清除 hexo 缓存：
```bash
hexo clean 
```

生成静态文件
```bash
hexo g
```

部署网站，d 的意思是 deploy

```bash
hexo d
```

后两个命令可以合并为一条，关于 Hexo 的命令请访问：[Hexo指令](https://hexo.io/zh-cn/docs/commands.html)。

```bash
hexo g -d
```

至此，博客已经搭建完毕，浏览器键入`https://{账户名}.github.io`，发现打开了一个使用 Hexo 搭建的 Github Pages 博客。

## 配置

博客文件夹下的`_config.yml`文件可以配置整个博客的名称、主题等基本功能，`/theme/`文件夹下的`_config.yml`文件则用于配置具体的主题配置。

### 发表与删除

在博客文件夹打开 Git Bash，键入

```
hexo n "文章的标题"
```

之后便会生成一个 md 文件，在 md 文件中编辑文章保存，然后键入`hexo g -d`便会发布文章。

如果要删除文章，直接删除对应的 md 文件。基本上每一次对博客的修改，都需要生成静态文件和部署命令。

### 主题

网络上有很多优秀的 Hexo 主题，关于主题选择，可以查看这个知乎问答：[有哪些好看的 Hexo主题](https://www.zhihu.com/question/24422335)。笔者使用的是[NexT](https://github.com/iissnan/hexo-theme-next)。

在博客文件夹下的`_config.yml`文件中，将theme选项设置为下载的主题，注意的是所有的选项冒号之后要有一个半角空格，否则设置将失效。Hexo会自动检测博客`_config.yml`文件的更改，保存片刻即生效。

### 域名绑定

具体过程如下：
- 需要一个域名，可以前往万网等提供商购买，之后在后台设置添加解析，一般需要解析三个，Github 的 IP 地址，`{账户名}.github.io`的 IP 地址，还有一个的记录类型为`CNAME`，记录值为：`{账户名}.github.io`。
- 进入 GitHub Pages 的仓库，点击 settings，设置 Custom domain，输入购买的域名，保存。
- 进入博客文件夹中的`/source/`，新建一个记事本文件，只需填写域名即可。如果在域名前填了 www，则每次浏览器访问都要填入 www，不填则没这个必要。之后将该文件保存为**所有文件**，名称为**CNAME**。

## 备份与恢复

由于在 Github Pages 的仓库中，只有生成的静态网页的文件，没有整个博客的源文件，如果当前电脑出现问题，或者需要多设备操作时，就会很麻烦。所以需要对博客进行备份与恢复。

### 备份

创建一个分支，用来保存网站源文件，具体步骤如下:
- 新建一个分支，如 hexo，并将其设置为默认
- 本地 clone 你的 Github Pages 仓库，得到一个 io 文件夹：`{账户名}.github.io`的文件夹。
- 将原来博客文件夹中的`_config.yml，themes/，source/，scffolds/，package.json，.gitignore`复制到 clone 下来的文件夹，注意要将`theme/`主题的`.git/`删除。
- 在 clone 下的文件夹执行`npm install`，`npm install hexo-deployer-git`。

此时`{账户名}.github.io`文件夹已经成为包含你博客所有文件的工作文件夹，在部署(`hexo g -d`)之前，执行下面三条命令，以使用 hexo 分支更新 git 上的源文件:
```bash
git add .
git commit -m "更新源文件"
git push origin hexo
```

这样一来，达到了在 master 分支上部署网页静态文件，在hexo分支上备份网站源文件的目的。

### 恢复

当在新的环境中需要对博客进行操作，包括写文章、配置博客时，需要获取整个网站的源文件，当然前提是Node.js 的安装，步骤如下:
- clone 你的 Github Pages 仓库，得到一个 io 文件夹。
- 在文件夹中打开 Git Bash，键入以下命令:
```bash
npm install hexo-cli -g
npm install 
npm install hexo-deployer-git 
```
现在就可以在该文件夹进行博客的操作了，如果多终端同时工作时，记得使用 pull 命令更新本地文件，且分支始终为 hexo。

如果在拉取过程中提示本地分支与远程仓库冲突，可以使用`git reset --hard`命令重置之后再次 pull。如果提示
> The following untracked working tree files would be overwritten by merge

可以使用`git clean -d -fx`，关于该命令

> git clean -f -n       //选项-n将显示执行下一步时将会移除哪些文件。
 git clean -f            //该命令会移除所有上一条命令中显示的文件。
 git clean -fd           //移除文件夹，使用选项-d。
 git clean -fX           //只想移除已被忽略的文件，使用选项-X。
 git clean -fx           //想移除已被忽略和未被忽略的文件，使用选项-x。

## Hexo各文件（夹）说明

- _config.yml：站点的配置文件，备份过程中需要拷贝；
- themes/：主题文件夹，需要拷贝；
- source：博客文章的.md文件，需要拷贝；
- scaffolds/：文章的模板，需要拷贝；
- package.json：安装包的名称，需要拷贝；
-  .gitignore：限定在push时哪些文件可以忽略，需要拷贝；
-  .git/：主题和站点都有，标志这是一个git项目，不需要拷贝；
- node_modules/：是安装包的目录，在执行npm install的时候会重新生成，不需要拷贝；
- public：hexo g生成的静态网页，不需要拷贝；
-  .deploy_git：同上，hexo g也会生成，不需要拷贝；
- db.json：文件，不需要拷贝。