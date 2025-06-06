+++
title = "orgmode 显示数学公式"
date = 2025-06-06
tags = ["Emacs", "Org-Mode", "Mathjax"]
categories = ["Emacs"]
draft = false
author = "Chris Ng"
+++

在使用Emacs前习惯使用Markdown来记笔记，使用Typora所见即所得的方式，又或者是使用Vscode的preview插件来实时查看数学公式，但是切换到Emacs的Org Mode后，显示数学公式似乎很困难，经过我的探索，总结出了几个比较好用的 `Org Mode` 显示数学公式的方式


## 使用 org-letex-impatient 实时显示公式 {#使用-org-letex-impatient-实时显示公式}

这个教程介绍 `org-latex-impatient` ，这个插件的主要原理是使用 Mathjax 的 `tex2svg` 命令生成svg图像来实现实时预览，具体可以查看链接 [org-latex-impatient](https://github.com/yangsheng6810/org-latex-impatient)


### Installation {#installation}

官方教程中介绍的方法是安装 `mathjax-node-cli` ，借助命令 `tex2svg` 来实现预览，但是在实操中发现失败，因为 `mathjax-node-cli` 这个项目太老了，已被归档不再更新， `Mathjax V3` 提供了类似的功能，下面的教程主要介绍如何使用 `mathjax v3` 配置 `org-latex-impatient` 。下面的教程在mac下进行，linux的教程类似，请自行探索。


#### 安装 Node.js {#安装-node-dot-js}

```shell
brew install node
```


#### 自定义转化工具 {#自定义转化工具}

在随机的一个目录下创建一个项目文件夹

```shell
mkdir ~/script/mathjax-converter
cd ~/script/mathjax-converter
```

初始化一个新的 node.js 项目

```shell
npm init -y
```

安装 `Mathjax`

```shell
npm install mathjax-full
```

在 `mathjax-converter` 目录下创建 `convert.js` 文件，并复制以下的内容进去。

```js
// 导入 MathJax v3 的核心组件
const { mathjax } = require('mathjax-full/js/mathjax.js');
const { TeX } = require('mathjax-full/js/input/tex.js');
const { SVG } = require('mathjax-full/js/output/svg.js');
const { liteAdaptor } = require('mathjax-full/js/adaptors/liteAdaptor.js');
const { RegisterHTMLHandler } = require('mathjax-full/js/handlers/html.js');

// 初始化转换环境
const adaptor = liteAdaptor();
RegisterHTMLHandler(adaptor);

const tex = new TeX({ packages: require('mathjax-full/js/input/tex/AllPackages.js').AllPackages });
const svg = new SVG({ fontCache: 'none' }); // 'none' 表示不缓存字体，更适合命令行一次性使用

// 创建一个异步函数来执行转换
async function texToSvg(texInput) {
    const math = await mathjax.document('', {
        InputJax: tex,
        OutputJax: svg
    });

    // 将 TeX 转换为 SVG
    const node = math.convert(texInput, {
        display: true, // true 表示显示模式 (display style), false 表示行内模式 (inline style)
        em: 16,
        ex: 8,
        containerWidth: 80 * 16
    });

    // 从结果中提取 SVG 代码并输出
    return adaptor.innerHTML(node);
}

// 获取命令行传入的第一个参数作为 TeX 输入
const texInput = process.argv[2];

if (!texInput) {
    console.error("用法: node convert.js \"<Your-TeX-String>\"");
    process.exit(1);
}

// 运行转换并打印结果到控制台
texToSvg(texInput).then(svgOutput => {
    console.log(svgOutput);
}).catch(err => {
    console.error(err);
});
```

接下来在终端中进行测试，运行下面的命令，当前目录下应该会出现一个svg文件

```shell
node convert.js "E = mc^2" > formula.svg    # 简单公式
node convert.js "\\frac{-b \\pm \\sqrt{b^2-4ac}}{2a}" > quadratic.svg    # 复杂公式
```

查看 `org-latex-impatient` 的教程，这个插件主要是依赖svg命令，因此还需要对 `convert.js` 文件进行包装，在当前路径先创建 `tex2svg` 文件，粘贴以下的内容进去

```shell
#!/bin/bash

# 这是一个给 Emacs 插件使用的包装脚本。
# 它接收来自 Emacs 的参数，然后传递给我们的 Node.js 脚本。

# 填写你的 convert.js 的绝对路径
NODE_SCRIPT_PATH="/path/to/your/mathjax-converter/convert.js"

# 使用 node 执行脚本, "$@" 会将所有从 Emacs 传来的参数原封不动地传过去
node "$NODE_SCRIPT_PATH" "$@"
```

接下来赋予可执行权限。

```shell
chmod +x tex2svg
```

然后，即可在命令行中进行测试

```shell
./tex2svg "E=mc^2"
```

为了方便我们直接调用，我们可以将这个命令放入系统的path路径下

```shell
sudo cp tex2svg /usr/local/bin
```

重启终端即可在系统中直接正确使用 tex2svg 命令。


#### 配置 init.el {#配置-init-dot-el}

在 `~/.emacs.d/init.el` 中添加以下的配置

```emacs-lisp
(use-package org-latex-impatient
  :defer t  ; 延迟加载，直到需要时才加载
  :hook (org-mode . org-latex-impatient-mode) ; 在打开 Org 文件时自动启用
  :init
  (setq org-latex-impatient-tex2svg-bin
        "/usr/local/bin/tex2svg")) ; 切换成命令的实际绝对路径
```

接下来，应该在编辑org文件的时候，就可以实时查看Emacs了


## org 导出 html 渲染latex公式 {#org-导出-html-渲染latex公式}

我们使用 `c-x c-o h o` 导出网页时，发现导出的网页并不能够实时显示数学公式，实际上，orgmode的官网教程对此有介绍，具体参考 [Math formatting in HTML export](https://orgmode.org/manual/Math-formatting-in-HTML-export.html)

只需要在org文件的头部添加以下的内容

```org
#+HTML_MATHJAX: align: left indent: 5em tagside: left
```

以及下面的命令其中一个即可

```org
#+OPTIONS: tex:dvipng
```

```org
#+OPTIONS: tex:dvisvgm
```

在导出的时候即可显示数学公式


## 所见即所得 `org-fragtog` {#所见即所得-org-fragtog}

第一种方法介绍了在编辑的过程中实时显示数学公式，那么，有没有一种方法能够实现 Typora 编写数学公式的效果呢？ 答案是有一个插件确实能够实现，具体可以参考[org-fragtog](https://github.com/io12/org-fragtog)

只需安装 org-fragtog 插件，然后在 `init.el` 添加配置即可

```emacs-lisp
(add-hook 'org-mode-hook 'org-fragtog-mode)
```

当然，我们不想在使用org后就启动，可以在我们编辑好后，通过 `M+x org-fragtog` 来启动。

但是，我发现使用这种方式显示的数学公式会出现过小看不清的问题，可以在 `init.el` 添加以下配置进行缩放

```emacs-lisp
;; 调整 Org Mode LaTeX 片段预览的显示比例
;; 默认值是 1.0 (100%)
;; 1.5 表示放大到 150%，通常是一个比较舒适的大小
(setq org-format-latex-options (plist-put org-format-latex-options :scale 1.5))
```
