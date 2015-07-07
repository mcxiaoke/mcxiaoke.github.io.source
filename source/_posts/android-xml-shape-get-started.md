Title: Android XML Shape使用入门
Date: 2011-05-15 18:12
Author: mcxiaoke
Category: Android
Tags: Android,Drawable
Slug: android-xml-shape-get-started

XML
Drawable是google为android定义的一种声明式图形文件格式，类似于SVG，但更简单，官方文档非常少，网上有人整理了一份文档，值得细看：

[Android Drawable XML
Documentation](http://idunnolol.com/android/drawables.html)

下面是一个简单的使用例子：

Activity代码  

``` 
package com.test;

import android.app.Activity;  
import android.os.Bundle;

public class DrawableTest extends Activity {  
@Override  
public void onCreate(Bundle savedInstanceState) {  
super.onCreate(savedInstanceState);  
setContentView(R.layout.main);  
}  
}  
```

布局文件  

```
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout
xmlns:android="http://schemas.android.com/apk/res/android"  
android:orientation="vertical" android:layout_width="fill_parent"  
android:layout_height="fill_parent">  
<TextView android:textSize="16sp"
android:layout_width="fill_parent"  
android:layout_height="wrap_content" android:text="@string/hello"  
android:background="@drawable/rect_gradient_linear" />  
<TextView android:textSize="16sp"
android:layout_width="fill_parent"  
android:layout_height="wrap_content" android:text="@string/hello"  
android:background="@drawable/oval_gradient_linear" />  
<TextView android:textSize="16sp"
android:layout_width="fill_parent"  
android:layout_height="wrap_content" android:text="@string/hello"  
android:background="@drawable/rect_solid_stroke" />  
<TextView android:textSize="16sp"
android:layout_width="fill_parent"  
android:layout_height="wrap_content" android:text="@string/hello"  
android:background="@drawable/rect_solid_corners" />  
</LinearLayout>  
```

Shapre使用实例，共四个：  

``` 
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="rectangle">  
<gradient android:startColor="#ffff0000"
android:endColor="#FF4488ee"  
android:type="linear" android:angle="270" />  
<padding android:left="2dp" android:top="2dp" android:right="2dp"  
android:bottom="2dp" />  
</shape>  
[/xml]  
[xml]  
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="oval">  
<gradient android:startColor="#FFaa6622"
android:endColor="#FF2288ee"  
android:type="linear" />  
<padding android:left="2dp" android:top="2dp" android:right="2dp"  
android:bottom="2dp" />  
</shape>  
```
 
``` 
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="rectangle">  
<solid android:color="#ffff0000" />  
<padding android:left="5dp" android:top="2dp" android:right="2dp"  
android:bottom="5dp" />  
<stroke android:width="2dp" android:color="#ffffffff"
android:dashGap="5dp"  
android:dashWidth="10dp" />  
</shape>  
```

```
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="rectangle">  
<solid android:color="#8800ff00" />  
<padding android:left="5dp" android:top="5dp" android:right="5dp"  
android:bottom="5dp" />  
<corners android:radius="15dp" />  
</shape>  
```

下面是系统源码里的例子，文件为：source/frameworks/base/core/res/res/drawable/progress_horizontal.xml  

```
<layer-list
xmlns:android="http://schemas.android.com/apk/res/android">

<item android:id="@android:id/background">  
<shape>  
<corners android:radius="5dip" />  
<gradient  
android:startColor="#ff9d9e9d"  
android:centerColor="#ff5a5d5a"  
android:centerY="0.75"  
android:endColor="#ff747674"  
android:angle="270"  
/>  
</shape>  
</item>

<item android:id="@android:id/secondaryProgress">  
<clip>  
<shape>  
<corners android:radius="5dip" />  
<gradient  
android:startColor="#80ffd300"  
android:centerColor="#80ffb600"  
android:centerY="0.75"  
android:endColor="#a0ffcb00"  
android:angle="270"  
/>  
</shape>  
</clip>  
</item>

<item android:id="@android:id/progress">  
<clip>  
<shape>  
<corners android:radius="5dip" />  
<gradient  
android:startColor="#ffffd300"  
android:centerColor="#ffffb600"  
android:centerY="0.75"  
android:endColor="#ffffcb00"  
android:angle="270"  
/>  
</shape>  
</clip>  
</item>

</layer-list>  
```

可供参考的资料：  
Layout secrets: “layer-list” and “include”
http://android.amberfog.com/?p=9

