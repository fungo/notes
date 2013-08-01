Fedora 17 安装 Fctix 输入法
=============================

原因
------
主要是因为要使用 Sublime Text 记笔记，但是 Sublime Text 对中文输入法的支持很烂，网上给出了一种 hack 的方法解决中文输入法问题，但是这个要求是 fctix 输入法，Fedora 17 默认的 iBus 在这种解决方法下不起作用。

安装过程
--------

* 安装 fcitx ::
	
	yum install fcitx
	yum install fcitx fcitx-table-chinese

	# fcitx-configtool 是 Fcitx 的可视化配置工具
	yum install fcitx-configtool

* 设置输入法为 fcitx

	按 Alt+F2，输入命令 im-chooser，然后选择 Fcitx

* 设置环境变量 ::

	# 在 .bahsrc 最后添加
	export XMODIFIERS="@im=fcitx"
	export QT_IM_MODULE=fcitx
	export GTK_IM_MODULE=fcitx

* 注销重新登录就可以使用 Fcitx 啦

参考
------
`Fcitx 在 Fedora 17(Gnome3) 上的安装及配制 <http://songyueju.blogbus.com/logs/226417286.html>`_