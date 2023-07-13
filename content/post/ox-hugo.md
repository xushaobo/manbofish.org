+++
title = "博客写作流程之工具篇： emacs, orgmode, hugo & ox-hugo"
date = 2018-03-31
lastmod = 2022-11-17T15:14:36+08:00
tags = ["emacs", "editor", "orgmode", "hugo", "写作", "tool", "博客"]
categories = ["计算机"]
draft = false
+++

本文将对我个人的博文写作流程 **所用到的工具** 做一个总结与分享。从标题就可以看出来，主要有这几个工具： `emacs`, `orgmode` &amp; `hugo` ，另外还有两个配合 `hugo` 的辅助包 `easy-hugo` （可选） &amp; `ox-hugo` 。

-   `hugo` : <https://gohugo.io/>
-   `orgmode` : <https://orgmode.org/>
-   `ox-hugo` : <https://ox-hugo.scripter.co/>
-   `easy-hygo` : <https://github.com/masasam/emacs-easy-hugo>

<!--more-->


## 自问自答 {#自问自答}


### 问： 为什么写这篇文章？ {#问-为什么写这篇文章}

答： 中文搜索居然搜索不到一篇有关 `ox-hugo` 的内容。


### 问： 这篇文章主要解决什么问题？ {#问-这篇文章主要解决什么问题}

答： orgmode 配合 hugo 来写作、发布、管理博文的一种便捷方案。


### 问： 为什么用 emacs 和 orgmode ？ {#问-为什么用-emacs-和-orgmode}

答： 谁让我当年入了 emacs 和 orgmode 的「坑」 :joy: ，这只是习惯而已。这俩工具还是需要一定的学习成本的，因此，本文对不熟悉 emacs 和 orgmode，或者使用其它编辑器的用户没多大帮助，但多少可以了解一下。


### 问： 为什么用 hugo ？ {#问-为什么用-hugo}

答： 最开始只是因为 hugo 原生支持 orgmode ，事后来看，其实支持的不是很好，但是 ox-hugo 解决了用 orgmode 写博文的问题。当然还有一点，在生成静态网站的诸多工具（如 jekyll, hexo 等等）中，我发现 hugo 的安装是最便捷的，只需要下载一个二进制文件直接安装就能用了，而其它工具却总要安装一些别的依赖。网上还有种说法是 hugo 的网站生成速度是最快的，这个我没测试过，也不太确定。


## 发布博文的一般流程 {#发布博文的一般流程}

1.  创建一个 orgmode heading 作为博文标题；
2.  写文章（废话）；
3.  使用 ox-hugo 生成 markdown 文件；
4.  使用 easy-hugo 预览；
5.  发布（废话）。

如前文所说，如果你不熟悉 emacs 和 orgmode ，一定会觉得这个流程好麻烦，何必用这些工具，把写作这回事儿弄得复杂了。我下面会对这个流程进行补充说明，并阐述它们的优点。


## 创建博文 {#创建博文}

正常我们使用 hugo 是怎样创建一篇新博文的呢？在命令行下，敲击命令 `hugo new posts/my-first-post.md` ，然后用自己熟悉的编辑器编辑这个文件。这个文件的头部包含我们这篇文章的一些基本信息，比如像这样：

```yaml
---
title: "An Example Post"  #标题
date: 2018-01-01T16:01:23+08:00 #发布时间
lastmod: 2018-01-02T16:01:23+08:00 #修改时间
draft: false  #是否是草稿？
tags: ["tag-1", "tag-2", "tag-3"]  #标签
categories: ["index"]  #分类
---
```

那么，使用 `ox-hugo` 之后，是怎样表示这些基本信息的呢？

一种是使用 orgmode 的一个标题（heading），这种方式比较适合一些随笔或者简短的文章，像这样：

{{< figure src="~/Dropbox/Write/blog/static/image/other/ox-hugo-01.png" >}}

**说明：**

-   1 是 ox-hugo 的单文件配置
-   2 是这篇文章的完成情况， `DONE` 说明已完成
-   3 是这篇文章的标题
-   4 是这篇文章的 tag ，tag 前带有 `@` 表示 category
-   5 是 `DONE` 自动生成的时间戳，表示这篇文章的完成时间
-   6 是导出为 markdown 文件后的文件名

还有一种是使用单独的文件 `example.org` ，像这样：

{{< figure src="~/Dropbox/Write/blog/static/image/other/ox-hugo-02.png" >}}

这两种方式都可以配合 emacs 强大的功能来快速生成，而不必每次都手动输入，亦或是复制粘贴。


### 使用 org-capture 创建博文 {#使用-org-capture-创建博文}

以下配置摘自 [官方介绍](https://ox-hugo.scripter.co/doc/org-capture-setup/) ：

```elisp
;; org-hugo capture
(with-eval-after-load 'org-capture
  (defun org-hugo-new-subtree-post-capture-template ()
    "Returns `org-capture' template string for new Hugo post.
See `org-capture-templates' for more information."
    (let* (;; http://www.holgerschurig.de/en/emacs-blog-from-org-to-hugo/
           (date (format-time-string (org-time-stamp-format :long :inactive) (org-current-time)))
           (title (read-from-minibuffer "Post Title: ")) ;Prompt to enter the post title
           (fname (org-hugo-slug title)))
      (mapconcat #'identity
                 `(
                   ,(concat "** TODO " title "     :@随笔:")
                   ":PROPERTIES:"
                   ,(concat ":EXPORT_FILE_NAME: " fname)
                   ;; ,(concat ":EXPORT_DATE: " date) ;Enter current date and time
                   ":END:"
                   "%?\n")          ;Place the cursor here finally
                 "\n")))

  (add-to-list 'org-capture-templates
               '("h"                ;`org-capture' binding + h
                 "Hugo post"
                 entry
                 ;; It is assumed that below file is present in `org-directory'
                 ;; and that it has a "Blog Ideas" heading. It can even be a
                 ;; symlink pointing to the actual location of all-posts.org!
                 (file+headline "~/Dropbox/Write/blog/orgpost/0000-posts.org" "INBOX")
                 (function org-hugo-new-subtree-post-capture-template))))

(spacemacs/set-leader-keys "av" 'org-capture)

```


### 使用 YASnippet 创建博文 {#使用-yasnippet-创建博文}

我在文末的附录中，会附上我使用的一些 snippet 模板。

模板的用法：
输入字母 `h` ，按下 `M-/` ，它就会自动帮你生成以下内容：

```nil
#+HUGO_BASE_DIR: ../
#+TITLE:
#+DATE:
#+HUGO_AUTO_SET_LASTMOD: t
#+HUGO_TAGS:
#+HUGO_CATEGORIES:
#+HUGO_DRAFT: false
```


## 由 ox-hugo 生成 markdown {#由-ox-hugo-生成-markdown}

尽管说 `hugo` 原生支持渲染 orgmode 文件，但它所使用的 markdown 渲染引擎比 orgmode 的渲染引擎要强大的多，谁让 markdown 更流行呢？这是使用 `ox-hugo` 的原因之一。

使用 `ox-hugo` 可以将你的 orgmode 博文，生成指定的 markdown 文件，这只需要一条指令就够了（官方默认 `C-c C-e H h` ），并且，它还会自动更正你的文章 **修改时间** ，markdown 文件中的 `lastmod` 值。

至此，你就已经可以按照常规的方法来预览、发布你的博文了。不过还有一个扩展没有介绍—— `easy-hugo` 。


## 使用 easy-hugo 管理你的博客（可选） {#使用-easy-hugo-管理你的博客-可选}

{{&lt; figure src="/image/other/ox-hugo-00.png" title="Easy-Hugo 主界面" class="center" &gt;}}

常规的方法是怎样来预览你的博客呢？先进入你的博客目录，然后键入命令 `hugo server` 。

使用 `easy-hugo` 之后，打开它的主界面（我的快捷键 `<SPA> a h` ），再按 `p` 键就可以直接预览了。它还有快速打开你的博客配置文件，快速发布到 github，等等功能，可以单独写篇介绍了。不过我只是用它来管理博文而已。


## 最后的总结 {#最后的总结}

`emacs` ，想说爱你不容易。:sweat_smile:

用惯了一种编辑器，任何文本编辑相关的工作都想通过它来完成，换用其他的编辑器总觉得缺了些什么。这个写作流程，在我短暂的使用过程中（不到一年），总体上还是挺舒适的。这篇文章算是做一个梳理。

为什么我从头到尾都没提到如何配置 `ox-hugo` 和 `easy-hugo` 呢？ emacs 的配置千差万别，怎样配置还是自己去看它们的文档吧。


## 附录A：几个辅助写作的 emacs 扩展 {#附录a-几个辅助写作的-emacs-扩展}

1.  [joostkremers/writeroom-mode: Writeroom-mode: distraction-free writing for Emacs.](https://github.com/joostkremers/writeroom-mode)
2.  spacemacs 的 emoji Layer ，方便插入 emoji 表情


## 附录B：快速获取 orgmode 或 markdown 格式的网页标题和链接 {#附录b-快速获取-orgmode-或-markdown-格式的网页标题和链接}

将以下代码存储为浏览器的书签，以后获取网页的标题和链接只需要点击这个书签即可。链接的标题默认是网站的标题，也可以选中一段文字作为标题。

orgmode 的 bookmarklet ：

```javascript
javascript:(function () { var selection = window.getSelection().toString(); var anchor = selection ? selection : document.title; void(prompt('', '[[' + location.href + '][' + anchor + ']]')); })();
```

markdown 的 bookmarklet ：

```javascript
javascript:(function () { var selection = window.getSelection().toString(); var anchor = selection ? selection : document.title; void(prompt('', '[' + anchor + '](' + location.href + ')')); })();
```


## 附录C：我的 YASnippet 模板 {#附录c-我的-yasnippet-模板}


### hugo 文章信息模板 {#hugo-文章信息模板}

```nil
# -*- mode: snippet -*-
# name: hugo
# key: h
# --
#+HUGO_BASE_DIR: ../
#+TITLE: $1
#+DATE: `(format-time-string "%Y-%m-%d")`
#+HUGO_AUTO_SET_LASTMOD: t
#+HUGO_TAGS: $2
#+HUGO_CATEGORIES: $3
#+HUGO_DRAFT: false

$0
```


### more {#more}

我也搞不清为什么我总是记不住 `<!--more-->` 这个标签

```nil
# -*- mode: snippet -*-
# name: more
# key: m
# --
<!--more-->

$0
```

有待补充...
