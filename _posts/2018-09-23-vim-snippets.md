---
layout: post
title: 在vim中使用snippets
guid: b29a210689a14c58bdd8a6957e38802a
categories: [VIM]
tags: [vim]
redirect_from:
    - /2018/09/23/
---
**目录：**
* Kramdown table of contents
{:toc .toc}
* * * 

# 在vim中使用snippets {#use-snippets-in-vim}
推荐使用[vim-plug](https://github.com/junegunn/vim-plug)作为vim包管理器，轻量且能异步更新，还支持插件分支。

## 1. 所用插件  {#plugin-involved}

```vim
" vim-plug配置插件方式
" supertab用来防止使用tab展开snippet与youcompleteme的tab补全发生冲突
Plug 'ervandew/supertab'
" ultisnips是引擎
Plug 'SirVer/ultisnips'
" 所有常用snippet都在vim-snippets里
Plug 'honza/vim-snippets'
```

## 2. 插件设置，以及YCM与UltiSnips不冲突的设置方法  {#plugin-config}
具体`supertab`插件以及`UltiSnip`插件设置如下：
```vim
" make YCM compatible with UltiSnips (using supertab)
let g:ycm_key_list_select_completion = ['<C-n>', '<Down>']
let g:ycm_key_list_previous_completion = ['<C-p>', '<Up>']
let g:SuperTabDefaultCompletionType = '<C-n>'

" better key bindings for UltiSnipsExpandTrigger
let g:UltiSnipsSnippetDirectories = ['~/.vim/UltiSnips', 'UltiSnips']
let g:UltiSnipsExpandTrigger = "<tab>"
let g:UltiSnipsJumpForwardTrigger = "<tab>"
let g:UltiSnipsJumpBackwardTrigger = "<s-tab>"
" If you want :UltiSnipsEdit to split your window.
let g:UltiSnipsEditSplit="vertical"
```

这样YouCompleteMe就被绑定到了`Ctrl+n`，然后这个快捷键又被`SuperTab`绑定到`tab`。

## 3.  自定义snippet {#how-to-customize-snippet-use-UltiSnips}
可以使用`UltiSnips`的语法自定义代码块，比如`jekyll`博客，我们每次新建博客的时候都要加入固定的博客头，我们想要实现输入`head`然后按下`tab`键自动补全固定表头。

自定义snippet如下：
```vim
# ~/.vim/bundle/UltiSnips/markdown.snippets 

snippet head "Jekyll post header" b
---
title: ${1:title}
layout: post
published: false
guid: `!p
import uuid
if not snip.c:
    guid = uuid.uuid4().hex
snip.rv = guid
`
date: `!v strftime("%Y-%m-%d %H:%M:%s")`
categories: [${2:Python}]
tags: [${3:code, python}]
---
**目录：**
* Kramdown table of contents
{:toc .toc}
* * * 

${0}
endsnippet
# vim:ft=snippets:

```

`UltiSnips`自定义snippet的语法为:`snippet trigger_word ["description" [options] ]`

在上例中，我们的`trigger_word`为`head`，`description`为`jekyll post header`, `options`为`b`，`b`意味着只有`trigger_word`在行首的时候，才执行对应操作; 其中的变量`${n}`，代表是第几个可以用`tab`跳转的可输入块，默认开始时光标停留在`${1}`上，最后一个可跳转位置是`${0}`。

