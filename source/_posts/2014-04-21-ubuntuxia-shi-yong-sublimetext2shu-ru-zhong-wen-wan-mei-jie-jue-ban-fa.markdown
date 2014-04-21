---
layout: post
title: "Ubuntu下使用SublimeText2输入中文完美解决办法"
date: 2013-04-28 13:43:15 +0800
comments: true
categories: ubuntu
---
卸载ibus，安装fcitx以及fcitx下的输入法
{% codeblock lang:bash %}
$ sudo apt-get remove ibus
$ sudo add-apt-repository ppa:fcitx-team/nightly
$ sudo apt-get update
$ sudo apt-get install fcitx fcitx-bin fcitx-config-common fcitx-config-gtk fcitx-data fcitx-frontend-gtk2 fcitx-frontend-gtk3 fcitx-frontend-qt4 fcitx-googlepinyin fcitx-libs fcitx-module-dbus fcitx-module-x11 fcitx-modules fcitx-pinyin fcitx-table fcitx-table-wubi fcitx-ui-classic
{% endcodeblock %}

将以下代码保存为`sublime_imfix.c`
{% codeblock lang:c %}
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
{% endcodeblock %}

安装需要的库
{% codeblock lang:bash %}
$ sudo apt-get install build-essential libgtk2.0-dev
{% endcodeblock %}

编译成共享库
{% codeblock lang:bash %}
$ gcc -shared -o libsublime-imfix.so sublime_imfix.c `pkg-config --libs --cflags gtk+-2.0` -fPIC
{% endcodeblock %}
 
运行`$ LD_PRELOAD=./libsublime-imfix.so sublime_text`即可启动(libsublime-imfix.so为编译生成的文件，sublime_text为执行文件的绝对路径)
 

为方便使用，将`libsublime-imfix.so`放到`/usr/lib/`下
{% codeblock lang:bash %}
$ sudo cp ./libsublime-imfix.so /usr/lib/
{% endcodeblock %}

创建`SublimeText2.desktop`,放到`/usr/share/applications/`下
{% codeblock %}
[Desktop Entry]
Type=Application
Name=SublimeText2
Comment=SublimeText2
Icon=/home/hanbing/Sublime Text 2/Icon/256x256/sublime_text.png
Exec=bash -c "LD_PRELOAD=/usr/lib/libsublime-imfix.so /home/hanbing/Sublime\ Text\ 2/sublime_text"
Terminal=false
{% endcodeblock %}

