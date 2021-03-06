Linux 下 sublime text 2 中文输入法解决
=====================================

原因
------
Sublime Text 2 在 Linux 下使用，中文输入法一直有问题，网上给了一种 hack 方法，写一段代码生成 share lib, 在 subl 启动前先加载这个库。

方法
------
* save below code to sublime_imfix.c ::

	/*
	sublime-imfix.c
	Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
	By Cjacker Huang <jianzhong.huang at i-soft.com.cn>

	gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
	LD_PRELOAD=./libsublime-imfix.so sublime_text
	*/
	#include <gtk/gtk.h>
	#include <gdk/gdkx.h>
	typedef GdkSegment GdkRegionBox;

	struct _GdkRegion
	{
	  long size;
	  long numRects;
	  GdkRegionBox *rects;
	  GdkRegionBox extents;
	};

	GtkIMContext *local_context;

	void
	gdk_region_get_clipbox (const GdkRegion *region,
	            GdkRectangle    *rectangle)
	{
	  g_return_if_fail (region != NULL);
	  g_return_if_fail (rectangle != NULL);

	  rectangle->x = region->extents.x1;
	  rectangle->y = region->extents.y1;
	  rectangle->width = region->extents.x2 - region->extents.x1;
	  rectangle->height = region->extents.y2 - region->extents.y1;
	  GdkRectangle rect;
	  rect.x = rectangle->x;
	  rect.y = rectangle->y;
	  rect.width = 0;
	  rect.height = rectangle->height; 
	  //The caret width is 2; 
	  //Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
	  if(rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
	        gtk_im_context_set_cursor_location(local_context, rectangle);
	  }
	}

	//this is needed, for example, if you input something in file dialog and return back the edit area
	//context will lost, so here we set it again.

	static GdkFilterReturn event_filter (GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
	{
	    XEvent *xev = (XEvent *)xevent;
	    if(xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
	       GdkWindow * win = g_object_get_data(G_OBJECT(im_context),"window");
	       if(GDK_IS_WINDOW(win))
	         gtk_im_context_set_client_window(im_context, win);
	    }
	    return GDK_FILTER_CONTINUE;
	}

	void gtk_im_context_set_client_window (GtkIMContext *context,
	          GdkWindow    *window)
	{
	  GtkIMContextClass *klass;
	  g_return_if_fail (GTK_IS_IM_CONTEXT (context));
	  klass = GTK_IM_CONTEXT_GET_CLASS (context);
	  if (klass->set_client_window)
	    klass->set_client_window (context, window);

	  if(!GDK_IS_WINDOW (window))
	    return;
	  g_object_set_data(G_OBJECT(context),"window",window);
	  int width = gdk_window_get_width(window);
	  int height = gdk_window_get_height(window);
	  if(width != 0 && height !=0) {
	    gtk_im_context_focus_in(context);
	    local_context = context;
	  }
	  gdk_window_add_filter (window, event_filter, context); 
	}
* compile a shared library ::

	gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC

* LD_PRELOAD it ::

	LD_PRELOAD=./libsublime-imfix.so sublime_text

* use a script to simplify this work ::

	#! /usr/bin/env bash

	SUBLIME_HOME="/home/fungo/Software/Sublime_Text_2"
	LD_LIB=$SUBLIME_HOME/lib/libsublime-imfix.so
	sh -c "LD_PRELOAD=$LD_LIB $SUBLIME_HOME/sublime_text $@"

参考
------

| `Ubuntu 12.10 sublime text 2 中文输入完美解决 <http://pobeta.com/ubuntu-sublime.html>`_
| `完美解决 Linux 下 Sublime Text 中文输入 <http://my.oschina.net/tsl0922/blog/113495>`_
| `Input method support <http://www.sublimetext.com/forum/viewtopic.php?f=3&t=7006&start=10#p41343>`_