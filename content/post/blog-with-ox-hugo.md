+++
title = "使用Ox-hugo来写博客"
description = "最近在学习Emacs，发现了一个能够使用 ~org mode~ 来写博客的新工具：ox-hugo，下面介绍我的配置方案。"
date = 2025-05-21
tags = ["org-mode", "hugo", "emacs", "ox-hugo"]
categories = ["Emacs"]
draft = false
author = "Chris Ng"
+++

还记得大四准备考研的时候，收听了一档播客叫 `Emacs Talk` ，听了里面的一些大牛分享他们的经历，受到了很多感触。尤其是听到经济学博士Kathy的有趣经历后，开始学习了 `Emacs` 之路。在Emcas Talk的推荐下，开始跟着子龙老师学习Emcas，但是学习了一段时间后，发现Emacs在Windwos上的性能实在是太差了，于是渐渐就放弃了这个工具。直到我最近换了一台Mac Mini后，使用了Unix系统，重新开始学习Emacs。我现在是Win和Mac双持，为了让 `Emacs` 能够成为我的生产力工具，我也探索出了在Windows平台使用Emacs的更好方法，就是在Wsl下使用Emacs，并使用Git进行多平台同步。


## 安装Hugo {#安装hugo}

在Mac下通常使用包管理器 `brew` 来安装，在终端运行以下的命令即可。

```shell
brew install hugo
```

在Arch Linux环境下，通常使用包管理器 `pacman` 来进行安装，在终端下运行以下的命令

```shell
sudo pacman -S hugo
```

安装完后，可以在终端输入以下的命令来查看是否正确安装。

```shell
hugo version
```


### 新建博客站点 {#新建博客站点}

首先进入一个你存放文件的路径，然后输入以下的命令

```shell
hugo new site blog
cd blog
git init
git add .
git commit -m "first commit"
```


### 安装hugo主题Even {#安装hugo主题even}

在当前的git下添加git模块

```shell
git submodule add https://github.com/olOwOlo/hugo-theme-even themes/even
```

将 `hugo-blog/themes/even/exampleSite/config.toml` 复制到站点的根目录下，替换根目录下的配置文件。
在博客根目录下运行以下的终端命令即可

```shell
hugo server
```

然而，在我的实际测试中，发现因为版本原因报错，原本的 `config.toml` 需要进行修改

```shell
# paginate = 5
pagination.paperSize = 5
```


### 使用 Org mode 来写博客 {#使用-org-mode-来写博客}

首先安装 `ox-hugo` 模块，在emacs的配置文件中加入以下配置

```emacs-lisp
(use-package ox-hugo
  :ensure t   ;Auto-install the package from Melpa
  :pin melpa  ;`package-archives' should already have ("melpa" . "https://melpa.org/packages/")
  :after ox)
```

这里按照ox-hugo的推荐，使用一个org文件进行写博客，每一个org subtree 对应一篇博客，org文件需要提前创建。

```emacs-lisp
(with-eval-after-load 'org-capture
  (defun org-hugo-new-subtree-post-capture-template ()
    "Returns `org-capture' template string for new Hugo post.
See `org-capture-templates' for more information."
    (let* ((title (read-from-minibuffer "Post Title: ")) ;Prompt to enter the post title
           (fname (org-hugo-slug title)))
      (mapconcat #'identity
                 `(
                   ,(concat "* TODO " title)
                   ":PROPERTIES:"
                   ,(concat ":EXPORT_FILE_NAME: " fname)
                   ":END:"
                   "\n\n")          ;Place the cursor here finally
                 "\n")))

  (add-to-list 'org-capture-templates
               '("h"                ;`org-capture' binding + h
                 "Hugo post"
                 entry
                 ;; It is assumed that below file is present in `org-directory'
                 ;; and that it has a "Blog Ideas" heading. It can even be a
                 ;; symlink pointing to the actual location of all-posts.org!
                 (file+headline "~/blog/all-blog.org" "Blog Ideas")
                 (function org-hugo-new-subtree-post-capture-template))))
```

在blog根目录下创建文件， `.dir-locals.el` ，内容如下。这个文件的作用是我们后续在修改org文件的时候，都会自动生成markdown文件。

```emacs-lisp
((org-mode . ((eval . (org-hugo-auto-export-mode)))))
```

需要在 `all-blog.org` 文件中加入以下的配置

{{< figure src="/ox-hugo/1.png" caption="<span class=\"figure-number\">Figure 1: </span>all-org.org 配置" >}}


### 创建你的第一篇博客 {#创建你的第一篇博客}

如果跟着上面的步骤，在Emacs下输入 `M+x org-capture`
![](/ox-hugo/2.png)

{{< figure src="content/figure/blogs_with_ox_hugo/3.png" >}}

在输入文字后，保存，即可自动生成markdown文件的博客。

需要注意的是，ox-hugo中的博客是以 `to do` 的形式来判断是否为草稿文件。

-   `TODO` ：代表未发布的文章，在生产Markdown的元数据中， `draft=true` ，只有在 `hugo server -D` 在会显示该文章。
-   `DONE` ：代表已完成的文章， `DONE Date` 会被作为发表的时间戳，当然也可以自己指定，具体请参考 [Org meta-data to Hugo front-matter](https://ox-hugo.scripter.co/doc/org-meta-data-to-hugo-front-matter/)
