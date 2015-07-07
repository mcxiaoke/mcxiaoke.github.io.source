Title: Android IntentService使用简介
Date: 2011-02-01 22:46
Author: mcxiaoke
Category: Android
Tags: Android,Service
Slug: android-intentservice-getting-started

春节放假有点自己的时间，准备抽空做个饭否上传照片的工具，其中上传功能需要用到后台任务，Android上的后台任务主要有三种实现方式，一是按照常规的Java方式，自己写线程，二是使用SDK封装好的后台任务类AsyncTask，三是使用Service，线程和AsyncTask都是和Activity的生命周期绑定的，而Service有自己的独立生命周期，考虑到以后的扩展和代码的清晰，决定使用Service，看到国外的博客和Google
Groups里面都推荐使用IntentService，网上搜了一下，几乎没有中文的资料，于是决定记录一下学习使用IntentService的笔记。

IntentService的位置是android.app.IntentService，官方文档的介绍我翻译一下：

```
IntentService is a base class for Services that handle asynchronous requests (expressed as Intents) on demand. Clients send requests through startService(Intent) calls; the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.
```

IntentService是一个用于按需处理异步请求的Service基类，调用方通过
startService(Intent)启动服务，IntentService为每一个Intent开启一个单独的工作线程，并且在任务完成时自动终止服务。

```
This "work queue processor" pattern is commonly used to offload tasks from an application's main thread. The IntentService class exists to simplify this pattern and take care of the mechanics. To use it, extend IntentService and implement onHandleIntent(Intent). IntentService will receive the Intents, launch a worker thread, and stop the service as appropriate.
```

这种“工作队列处理器”模式通常用于某个程序主线程之外的后台任务。IntentService类简化了这种机制。要使用这种工作队列模式，只使用继承IntentService并实现onHandleIntent(Intent)方法。IntentService会接受Intents，启动工作线程，并在合适的时候终止服务。

```
All requests are handled on a single worker thread -- they may take as long as necessary (and will not block the application's main loop), but only one request will be processed at a time.
```

文档的最后说，所有的请求都由一个单独的线程处理，它们可以占用任意长的时间（不会造成主线程阻塞），但是同一时间智慧处理一个请求。我的理解是先进先出的队列模式。

下面首先看看我的示例代码，这个是写的饭否照片上传的代码，

    :::java
	package com.mcxiaoke.fanfou.photo;

	import java.io.File;  
	import android.app.IntentService;  
	import android.content.Intent;  
	import android.os.IBinder;  
	import android.util.Log;

	public final class PhotoUploadService extends IntentService {  
	@Override  
	public IBinder onBind(Intent intent) {  
	Log.d(tag, "onBind()");  
	return super.onBind(intent);  
	}

	@Override  
	public void onCreate() {  
	Log.d(tag, "onCreate()");  
	super.onCreate();  
	}

	@Override  
	public void onDestroy() {  
	Log.d(tag, "onDestroy()");  
	super.onDestroy();  
	}

	@Override  
	public void onStart(Intent intent, int startId) {  
	Log.d(tag, "onStart()");  
	super.onStart(intent, startId);  
	}

	@Override  
	public int onStartCommand(Intent intent, int flags, int startId) {  
	Log.d(tag, "onStartCommand()");  
	return super.onStartCommand(intent, flags, startId);  
	}

	@Override  
	public void setIntentRedelivery(boolean enabled) {  
	Log.d(tag, "setIntentRedelivery()");  
	super.setIntentRedelivery(enabled);  
	}

	private final static String tag="PhotoUploadService";

	// public PhotoUploadService(String name) {  
	// super(name);  
	// Log.d(tag, "PhotoUploadService()");  
	// }

	public PhotoUploadService() {  
	super("PhotoUploadService");  
	Log.d(tag, "PhotoUploadService()");  
	}

	@Override  
	protected void onHandleIntent(Intent intent) {  
	Log.d(tag, "onHandleIntent()");  
	File photo=new File(intent.getStringExtra("photo"));  
	FanfouApi api=new FanfouApi("username", "password");  
	api.uploadPhoto(photo, "upload photo use service test.");

	}  
	}  


你要进行的后台任务写在onHandleIntent方法里就可以了，上面那个Override的方式都是我用于打印调试信息的，都可以不用重写。这里说一下IntentService的生命周期，根据Logcat打印的消息，那几个OnXXX的执行顺序从前到后依次是：

```
PhotoUploadService() -- 构造函数  
onCreate() -- 只在Service第一次启动时调用  
onStartCommand() -- 每次调用startService时都会调用这个方法  
onStart() -- SDK将这个方法标记为过时的，用onStartCommand()替代  
onHandleIntent() -- 要做的任务都写在这个方法里  
onDestroy() -- 服务终止时调用
```

关于继承IntentService，构造函数需要注意一个地方，Eclipse默认生成的构造函数是  

```
public PhotoUploadService(String name) {  
super(name);  
}  
```

使用这个默认构造函数的话会报一个运行时错误： `java.lang.InstantiationException `，在Google Groups里找到了解决办法，继承IntentService的类必须有一个public的无参的构造函数，将上面Eclipse自动生成的构造函数改为下面这样的就可以了：  
```
public PhotoUploadService() {  
super("someone");  
}  
```

为什么要这样改，看IntentService构造函数的源码：  

```
public IntentService(String name) {  
super();  
mName = name;  
}  
```

SDK文档里说构造函数里面的服务名字只在调试时有用，可以随便写一个名字。

另一个需要特别指出的时，在onHandleIntent里不需要自己处理线程，或者新启线程，IntentService默认会为队列中的任务启动后台线程，源码中的实现是这样的：  

	:::java
	private final class ServiceHandler extends Handler {  
	public ServiceHandler(Looper looper) {  
	super(looper);  
	}

	@Override  
	public void handleMessage(Message msg) {  
	onHandleIntent((Intent)msg.obj);  
	stopSelf(msg.arg1);  
	}  
	}

	@Override  
	public void onCreate() {  
	super.onCreate();  
	HandlerThread thread = new HandlerThread("IntentService[" + mName +
	"]");  
	thread.start();

	mServiceLooper = thread.getLooper();  
	mServiceHandler = new ServiceHandler(mServiceLooper);  
	}  


看源码可以得知onHanldeIntent()是在新的后台线程HandlerThread里执行的，所以不需要要我们自己新开线程。

参考资料：  

[IntentService](http://androidappdocs.appspot.com/reference/android/app/IntentService.html)  

[Service](http://androidappdocs.appspot.com/reference/android/app/Service.html)  
[Android: restful API
service](http://stackoverflow.com/questions/3197335/android-restful-api-service)  
[IntentService instantiation
exception](https://groups.google.com/forum/#!topic/android-developers/YTXqvKP526w)  
[android design considerations: AsynchTask vs
Service](http://stackoverflow.com/questions/3817272/android-design-considerations-asynchtask-vs-service-intentservice)

