title: Android开发杂记（一）
date: 2011-02-26 16:53
author: mcxiaoke
categories: 
- Android
tags: 
- Android
- Tips
#slug: android-dev-notes-01
---
这些是这半年做Android开发的小部分技巧记录，摘录自我的Tweets，大部分没有发在Twitter上的以后想起来了再记一下

* android:layout_alignLeft 指该控件距离边父控件的边距，android:paddingLeft 指该控件内部内容，如文本距离该控件的边距.当按钮分别设置以上两个属性时，效果不一样的 android:paddingLeft=”30px”，按钮上设置的内容（例如图片）离按钮左边边界30个像素 android:layout_marginLeft=”30px”，整个按钮离左边设置的内容30个像素

* Activity中添加 `android:windowSoftInputMode=”adjustPan”` 就可以自适应键盘的显示与隐藏

3. android:listSelector的值可以是一个Drawable，也可以是一个颜色值

* ListView的默认背景色是#FF191919，如果不做修改，而且设置了背景图片的话，那么拖动时就会出现黑色背景，解决办法是设置 `android:cacheColorHint`
为#00000000

* Android截屏代码： `view.setDrawingCacheEnabled(true);Bitmap bp =
view.getDrawingCache();`

* 当ListView中要显示的数据集合发生变化时，如集合中增删数据，这时需要刷新UI以响应数据变化，可调用adapter的notifyDataSetChanged();方法来刷新界面

* FrameLayout是最简单的一个布局对象。在它里面的的所有显示对象都将固定在屏幕的左上角，不能指定位置，但允许有多个显示对象，只是后一个会直接覆盖在前一个之上显示，会把前面的组件部分或全部挡住

* View类默认提供的是一个100*100大小的空白空间，自定义View必须重写OnDraw()和onMeasure()方法，否则父控件无法得知你自定义的View的尺寸

* 解决SeekBar下边显示不全的办法：将是layout_height为warp_content，再设置适当的minHeight和maxHeight值

* View设置背景透明的方法：`View v = findViewById(R.id.content);` //找到你要设透明背景的layout 的id 然后 `v.getBackground().setAlpha(100);` //0~255透明度值

* 由于触摸模式和普通模式切换的存在，在程序中不能依赖焦点和选中状态，因为某一时刻不一定存在焦点和/或选中状态

* 设置WebView的缩放需要设置一下 `browser.getSettings().setUseWideViewPort(true);`

* 去掉标题栏titleBar的方法：`requestWindowFeature(Window.FEATURE_NO_TITLE);`

* 全屏背景透明的设置方法：  `Activity：android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen"`

* 建议所有Android开发者升级ADT到最近的9.0，这个版本修复了以前Eclipse的logcat日志输出中不支持中文的BUG(Issue
1590)，现在采用UTF8编码，支持所有Unicode字符了

