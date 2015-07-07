title: Android Handler使用笔记
date: 2010-10-09 11:00
author: mcxiaoke
categories: 
- Android
tags: 
- Android
- Handler
#slug: android-handler
---
最近项目里要用到后台任务和多线程，看了一些资料，记录一下重点备忘。

Handler可用于在主线程中处理消息队列（MessageQueue），处理其它线程发送过来的Message，例如根据后台任务的处理过程更新前台UI。  
Handler类的使用过程一般是：  
1. 主线程，即UI线程中重写handleMessage方法，处理消息队列  
2. 后台线程中使用sendMessage发送消息

下面是一个简单的例子：  

```
package com.test;

import android.app.Activity;  
import android.os.Bundle;  
import android.os.Handler;  
import android.os.Message;  
import android.widget.TextView;

/**  
* @author mcxiaoke  
* @date 2010.10.09  
* @description: Handler Test  
*  
*/  
public class TestHandler extends Activity {  
TextView label;  
Handler myHandler;

protected void onCreate(Bundle savedInstanceState) {  
super.onCreate(savedInstanceState);  
setContentView(R.layout.main);  
label = (TextView) findViewById(R.id.label);  
myHandler = new Handler() {  
@Override  
public void handleMessage(Message msg) {  
super.handleMessage(msg);  
Bundle b = msg.getData();  
String color = b.getString("text");  
TestHandler.this.label.append(color);  
}  
};  
MyThread m = new MyThread();  
new Thread(m).start();  
}

class MyThread implements Runnable {  
public void run() {  
for (int i=0; i &lt; 20; i++) {  
try {  
Thread.sleep(2000);  
} catch (InterruptedException e) {  
e.printStackTrace();  
}  
Message msg = new Message();  
Bundle b = new Bundle();  
b.putString("text", "nMessage "+i+"...");  
msg.setData(b);  
myHandler.sendMessage(msg);  
}  
}  
}  
}
```

更完整的例子可以参考：<http://www.javaeye.com/topic/435147>

推荐代码高亮插件：  
[Syntax Highlighter and Code Colorizer for Wordpress](http://wordpress.org/extend/plugins/syntax-highlighter-and-code-prettifier/)

