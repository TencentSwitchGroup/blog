title: vim 插件记录
date: 2017-02-20 10:45:33
toc: true
tags: [vim]
categories: programmer
keywords: [vim, 配置, plugin, 插件]
description: vim 插件备忘
---

powerline
---------

[Powerline](https://github.com/powerline/powerline) 是 vim 的状态行插件，让你的状态行更加酷炫，装 13 利器。 有图有真相：
![Powerline示例图1](https://camo.githubusercontent.com/3c3a1717e42f17651f688ecc19f87e7433275098/68747470733a2f2f7261772e6769746875622e636f6d2f706f7765726c696e652f706f7765726c696e652f646576656c6f702f646f63732f736f757263652f5f7374617469632f696d672f706c2d6d6f64652d6e6f726d616c2e706e67)
![Powerline示例图2](https://camo.githubusercontent.com/627cc0d6ca618d9480dfbeeaf1f35ca0cc30781b/68747470733a2f2f7261772e6769746875622e636f6d2f706f7765726c696e652f706f7765726c696e652f646576656c6f702f646f63732f736f757263652f5f7374617469632f696d672f706c2d6d6f64652d696e736572742e706e67)
![Powerline示例图3](https://camo.githubusercontent.com/4a4454311274d16299aef08227496c5d1a7a9fd5/68747470733a2f2f7261772e6769746875622e636f6d2f706f7765726c696e652f706f7765726c696e652f646576656c6f702f646f63732f736f757263652f5f7374617469632f696d672f706c2d6d6f64652d76697375616c2e706e67)
![Powerline示例图4](https://camo.githubusercontent.com/28430e34155892705de259c0e5fb0eec63825856/68747470733a2f2f7261772e6769746875622e636f6d2f706f7765726c696e652f706f7765726c696e652f646576656c6f702f646f63732f736f757263652f5f7374617469632f696d672f706c2d6d6f64652d7265706c6163652e706e67)

全局示意图：来自[CENALULU'S TECH BLOG](http://cenalulu.github.io/linux/mac-powerline/)。
![全局示意图](http://cenalulu.github.io/images/linux/powerline/whole.png)

> Powerline is a statusline plugin for vim, and provides statuslines and prompts for several other applications, including zsh, bash, tmux, IPython, Awesome, i3 and Qtile.

windows 下的字体可直接在 github 上搜，个人比较喜欢 [consolas](https://github.com/eugeii/consolas-powerline-vim) 字体。

* 下载字体
* 双击字体，点击安装
* 在 console 上（比如 SecureCRT 或 putty）选择字体为 `Consolas for Powerline`。

winmanager
----------

有个 bug，使用 `wm` 启用 winmanager 的时候会出现一个空的窗口。详见 [将Vim改造为强大的IDE—Vim集成Ctags/Taglist/Cscope/Winmanager/NERDTree/OmniCppComplete（有图有真相）
](http://blog.csdn.net/bokee/article/details/6633193#)。

* 找到.vim 中 的`winmanager.vim`
* 添加如下的 `exe 'q'`

```
function! <SID>ToggleWindowsManager()  
   if IsWinManagerVisible()  
      call s:CloseWindowsManager()  
   else  
      call s:StartWindowsManager()  
      exe 'q'  
   end  
endfunction
```

vundle
------

* `git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle`
* .vimrc 配置好要安装的插件，可参考我的[配置文件](http://sunyongfeng.com/201605/programmer/tools/vimrc.html)
* 用 vim 随便打开一个文件，`:BundleInstall` 安装配置好的插件

cscope for c like language
--------------------------

例如配置 [P4 lang](p4.org):

* mkdir ~/.vim/plugin
* cd ~/.vim/plugin
* wget http://cscope.sourceforge.net/cscope_maps.vim
* cd your_p4_src
* cscope -R *.p4
* vim xxx.p4, 尽情享用 tag 跳转等功能。


