Homework

# 文档编辑说明：

## LaTeX支持
可增加对LaTeX渲染的支持，全局搜索 mathjax 然后取消注释；
详细可参考[这篇说明和示例](https://fromendworld.github.io/LOFFER/math-test/)。

## 置顶功能
只要在一个post的YAML Front Matter（就是文章头部的这段信息）中加入 pinned: true ，这篇文章就可以置顶了。

## 更换主题颜色
用了一个开源的颜色表[Open Color](https://yeun.github.io/open-color/),
该色表提供的可选颜色有：red, pink, grape, violet, indigo, blue, cyan, teal, green, lime, yellow。

当前默认状态是teal，要更换主题颜色，只要打开文件 _sass/_variables.scss ，将文件中所有的teal全部替换成你想要的颜色。
例如，查找teal，替换indigo，全部替换，commit，完成！

## 目录功能
在post的信息中心加入` toc: true `，这篇博文就会显示目录了。

这次没有对config的修改，因此应该可以通过[这个方法](https://github.com/KirstieJane/STEMMRoleModels/wiki/Syncing-your-fork-to-the-original-repository-via-the-browser)，给自己提pull request来更新。

目录基于[jekyll-toc by allejo](https://github.com/allejo/jekyll-toc)制作。

目前试用发现了一点小问题：如果你的标题级数不按套路变化，它就会搞不懂…… 

` # 一级标题 `下面必须是` ## 二级标题 `，如果是` ### 三级标题 `它就人工智障了【手动扶额】

注意：目前目录仅在桌面版显示。

## 编写博文
在发布博文前，你需要在文章的头部添加这样的内容，包括你的文章标题，发布日期，作者名，和tag等。

    ---
    layout: post
    title: 测试示例
    date: 2019-06-02
    Author: your name
    categories: 
    tags: [tag1, tag2]
    comments: true
    --- 

完成后，保存为.md文件，文件名是date-标题，例如 2019-06-02-document.md (注意这里的标题会成为这个post的URL，所以推荐使用字母而非中文，它不影响页面上显示的标题)，然后上传到_posts文件夹，commit，很快就可以在博客上看到新文章了。

## 图片
少量图片可以上传到images文件夹，然后在博文中添加。

但是GitHub用来当做图床有滥用之嫌，如果你的博客以图片为主，建议选择外链图床，例如[sm.ms](https://sm.ms/)就是和很好的选择。

如果想要寻找更适合自己的图床，敬请Google一下。

在博文中添加图片的Markdown语法是：`![图片名](URL)`

## 评论
### Disqus

支持Disqus评论，虽然Disqus很丑，但是它是免费的，设置起来又方便，因此大家也就不要嫌弃它。

首先，注册一个[Disqus](https://disqus.com/)账户，我们可以选择这个免费方案：

![img](https://raw.githubusercontent.com/FromEndWorld/LOFFER/master/images/Disqus-plan.png)

注册成功后，新建一个站点（site），以LOFFER为例设置步骤如下：

首先站点名LOFFER，生成了shortname是loffer，类型可以随便选。

![img](https://raw.githubusercontent.com/FromEndWorld/LOFFER/master/images/Disqus-1.png)

安装时选择Jekyll。

![img](https://raw.githubusercontent.com/FromEndWorld/LOFFER/master/images/Disqus-2.png)

最后填入你的博客地址，语言可以选中文，点Complete，即可！

![img](https://raw.githubusercontent.com/FromEndWorld/LOFFER/master/images/Disqus-3.png)

然后需要回到你的博客，修改_config.yml文件，在disqus字段填上你的shortname，commit，完成！

### Gitalk

设置方法如下：

首先，创建一个[OAuth application](https://github.com/settings/applications/new), 设置如图：

![img](https://raw.githubusercontent.com/FromEndWorld/LOFFER/master/images/application_settings.png)

点Register后就会看到你所需要的两个值，clientID和clientSecret，把它们复制到你的_config.yml文件中相应的字段：

    gitalk:
      clientID: <你的clientID>
      clientSecret: <你的clientSecret>
      repo: <你的repository名称>
      owner: <你的GitHub用户名>

然后commit，你的Gitalk评论区就会出现了。对于每一篇文章，都需要你来进入文章页，来初始化评论区，这一操作会在你的repository上创建一个Issue，此后的评论就是对这个Issue的回复。

你可以进入你的repository的Issue页面，点**Unsubscribe**来避免收到大量相关邮件。

注意：出于很明显的原因，最好不要同时添加Disqus和Gitalk评论区。

### Vssue
尝试中

# 致谢
给他们点☆！

* [Jekyll](https://github.com/jekyll/jekyll)
* [LOFFER](https://github.com/FromEndWorld/LOFFER)
