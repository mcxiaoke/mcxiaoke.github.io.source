title: Android开发杂记（二）
date: 2014-02-25 16:28
author: mcxiaoke
categories: 
- Android
tags: 
- Android
- Tips
#slug: android-dev-notes-02

---
2013年下半年的Android开发过程中记录的一些经验和教训，整理出来一部分


###Fragment的状态恢复问题 [20131218]

在FragmentActivity里，如果存在Fragment，系统恢复被销毁的Activity的同时会回复所有FragmentManager里的Fragment列表，然后添加到当前的Activity中，但问题是，Fragment虽然恢复了，状态却没有回复，这些都需要在onCreate或onRestoreInstanceState中手动处理。

FragmentActivity恢复状态的源码如下：

```
    // onCreate()中
    if (savedInstanceState != null) {
        Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
        // FragmentManager的restoreAllState会将之前保存的Fragment重新添加到FragmentManager中，并恢复BackStack
        mFragments.restoreAllState(p, nc != null ? nc.fragments : null);
    }
```

保存状态的源码如下：

```
    /**
     * Save all appropriate fragment state.
     */
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        Parcelable p = mFragments.saveAllState();
        if (p != null) {
            outState.putParcelable(FRAGMENTS_TAG, p);
        }
    }
```

Google建议onCreate中只在savedInstanceState为null的时候才创建和初始化Fragment，代码如下：

```
if (savedInstanceState == null) {
            // During initial setup, plug in the details fragment.
            DetailsFragment details = new DetailsFragment();
            details.setArguments(getIntent().getExtras());
            getFragmentManager().beginTransaction().add(android.R.id.content, details).commit();
    }
```

补充说明：onCreate()和onRestoreInstanceState()都可以用于应用的状态恢复，区别是onRestoreInstanceState()是在onStart()之后调用，可以根据具体情况选择时机。



###裁剪图片的Intent [20131218]

Android系统自带了Gallery和Camera，提供了剪裁和编辑图片功能，虽然没有提供一个文档化的接口，但通常还是可以安全的调用，两种方法都需要在onActivityResult()里处理裁剪结果。

方法一：对已有的图片进行裁剪，Intent如下：

```
    public static final String ACTION_CROP = "com.android.camera.action.CROP";
    /**
     *   裁剪已有的图片
     * @param activity 接受裁剪结果的Activity
     * @param srcUri 原始图片Uri
     * @param destUri 保存裁剪后图片的Uri
     */
    public static void showCrop(Activity activity, Uri srcUri, Uri destUri) {
        Intent intent = new Intent(ACTION_CROP);
        // 看源码得知，使用setData会清除type，直接setType会清除data
        intent.setDataAndType(srcUri, "image/*");
        //set crop properties
        intent.putExtra("crop", "true");
        //indicate aspect of desired crop
        intent.putExtra("aspectX", 1);
        intent.putExtra("aspectY", 1);
        intent.putExtra("scale", true);
        //indicate output X and Y
        intent.putExtra("outputX", Constants.AVATAR_DIMEN_MEDIUM);
        intent.putExtra("outputY", Constants.AVATAR_DIMEN_MEDIUM);
        //retrieve data on return
        intent.putExtra("return-data", false);
        intent.putExtra("noFaceDetection", true);
        intent.putExtra("setWallpaper", false);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, destUri);
        activity.startActivityForResult(intent, Constants.REQUEST_CROP);
    }
```

方法二，选择图片的同时执行裁剪操作：

```
    /**
     *   选择并裁剪图片
     * @param activity 接受裁剪结果的Activity
     * @param destUri 保存裁剪后图片的Uri
     */
    public static void showCrop(Activity activity, Uri destUri) {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("image/*");
        //set crop properties
        intent.putExtra("crop", "true");
        //indicate aspect of desired crop
        intent.putExtra("aspectX", 1);
        intent.putExtra("aspectY", 1);
        intent.putExtra("scale", true);
        //indicate output X and Y
        intent.putExtra("outputX", Constants.AVATAR_DIMEN_MEDIUM);
        intent.putExtra("outputY", Constants.AVATAR_DIMEN_MEDIUM);
        //retrieve data on return
        intent.putExtra("return-data", false);
        intent.putExtra("noFaceDetection", true);
        intent.putExtra("setWallpaper", false);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, destUri);
        activity.startActivityForResult(intent, Constants.REQUEST_GALLERY);
    }
```



###加载资源的另一种方法 [20131218]

Android系统中有多重方法加载资源文件，比如读取drawable目录的图片可以用getResources().getDrawable()，读取values目录的字符串可以用getResources().getString()，读取raw目录的文件可以用getResources().openRawResource()，读取assets目录的文件可以用getAssets().open()，这些都是Android系统特有的方法，除此之外，我们还可以用Java提供的加载资源的方法，这种方法不受位置限制，可以放在源文件相同的目录，下面是例子：


```
// 比如要加载src/com/douban/shuo/res/目录下的icon.png图片
// 可以这样做：
String path = "com/douban/shuo/res/icon.png";
InputStream is = getClassLoader().getResourceAsStream(path);
Drawable.createFromStream(is, "src");
```

getResourceAsStream是JDK 1.1开始就有的方法，可以通过ClassLoader加载任何格式的资源，返回一个InputStream供使用，具体路径解析规则可以看看这篇文章[ClassLoader.getResourceAsStream() 与 Class.getResourceAsStream()的区别](http://blog.csdn.net/ouyang_peng/article/details/8856764)


###在onActivityResult中显示Dialog [20131218]

有时会遇到这样的场景，通过startActivityForResult调用某一个应用等待返回结果，然后在onActivityResult中需要异步处理的同时显示对话框，如果直接在onActivityResult中调用Dialog.show()会报错，因为onActivityResult是在onResume之前，一个类似的问题是，在onSaveInstanceState调用之后也不能进行FragmentTransaction的commit操作，这两个都影响到DialogFragment，正确的做法有两种：

* 方法一：在onPostResume显示对话框
* 方法二：在onActivityResult设置一个标志(比如mPendingShowDialog)为true，然后在onResume()的时候检查标志，如果为true就显示对话框

注意：不仅是Dialog，处理Fragment相关的transactions和commit操作都需要考虑这个问题，在onSaveInstanceState调用之后如果需要commit，请替换成commitAllowingStateLoss，如果不允许状态丢失，就需要寻找其它替代方法。

详细解释可以参考：[“Failure Delivering Result ” - onActivityForResult](http://stackoverflow.com/questions/16265733/failure-delivering-result-onactivityforresult)。



###建议所有ID都定义在values里面 [20131218]
MenuItem ID最好和其它的resource id一样，定义在values里面，可以避免冲突，直接使用系统的Menu.FIRST递增容易冲突，比如和ShareActionProvider冲突(测试时看到使用的ID是2，Menu.FIRST值为1)



###Fragment的背景问题 [20131017]

如果使用Activity+Fragment的结构的话，一般至少会有三种背景，最底层的是Theme的背景，属性为`android:windowBackgroun d`，这个如果没有设置默认是白色/黑色（依主题不同），中间层是Activity的`android:background`，上面一层是Fragment的`android:background`,如果Fragment里面的View也有背景的话就会有多层背景，会有过度绘制的问题，性能会下降很多，4.1以上的系统的开发者选项里有一个**[显示GPU过度绘制]**选项，开启后会用颜色表示过度绘制的区域，从最优到最差依次是蓝绿淡红和红，除了个别图标和层次较深的文字以外，一般如果大片区域出现淡红色和红色就是比较严重的性能问题，特别是在ListView等多层嵌套的控件里。

这时候就需要针对性的进行优化，一般有两种方法：

1. 优化布局，尽量减少层次，典型的如使用RelativeLayout，使用merge，使用Canvas直接绘制（极端情况下，阅读类应用很多都是这样）等；
2. 去除不必要的背景，没有特殊需要的话，Activity不设置背景，大部分情况下Fragment也可以不设置背景，网上建议将android:windowBackground设置为@null在很多时候会有问题，比如Activity被销毁再打开的时候会呈现短暂的黑色背景（其实是没有背景），如果Activity不设置背景在某些情况下也会出现这种情况，特别是在内存不足Activity被销毁后从最近任务列表返回时。

> Fragment不设置背景存在的问题是，如果采用的是一个Activity+多个Fragment切换的结构，在部分机型上，Fragment切换后，前一个Fragment的画面会残留在背景上，造成重影现象，原因应该是系统认为Fragment没有设置背景所以没有强制刷新，真正的原因需要看源码才能确定。

更详细的绘图性能调优方法可以参考[Android Performance Case Study](http://www.curious-creature.org/2012/12/01/android-performance-case-study/)，具体应用时，还需要考虑到小米/魅族等机型和2.3系统的兼容性问题



###魅族M9的资源解析 [20131017]

如果在res目录里存在不能识别的资源ID会直报InflateException导致Crash，M9使用2.3系统，不能识别3.0以上才支持的ActionBar相关的属性，比如 `android:actionBarStyle` ，所以使用ActionBarSherlock时，ActionBar相关的属性必须分开放，不带android命名空间的放在values目录，带android命名空间的必须放在values-v14目录，这个之前Bear的日记里也有提到，具体见 [ABS causes an InflateException on some devices](https://github.com/JakeWharton/ActionBarSherlock/issues/446)



###摩托ME525的ListView显示 [20131017]

如果ListView设置了MATCH_PARENT但是内容太少又没有撑满空间，ListView会自动缩小至显示内容所需区域，空白空间会显示默认的背景色，在ME525上是灰色块，解决办法是在主题里加上：

```
	<item name="android:overScrollFooter">@android:color/transparent</item>
```

或者在布局里使用

```
	android:overScrollFooter="@android:color/transparent"
```

具体见 [Background color (listview?)](http://stackoverflow.com/questions/10655646/background-color-listview)



###ImageView的adjustViewBounds属性 [20131017]

1. adjustViewBounds属性设置为true的作用是保持原始图片的长宽比，某些情况下这可能不是你想要的效果，比如广播里缩略图需要保持正方形，这时候你需要手动设置长和宽，不能使用WRAP_CONTENT，也可以重写ImageView的onMeasure()方法强制保持长宽一致，广播源码里的SquaredCheckableImage就是重写了onMeasure()方法：

  ```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	setMeasuredDimension(getMeasuredWidth(), getMeasuredWidth());
}
```

2. `maxHeight`和`maxWidth`属性必须同时设置`adjustViewBounds`为true才能生效，原因见`ImageView.onMeasure()`方法的源码，只有`mAdjustViewBounds`为true时才会检查并设置`resizeWidth`和`resizeHeight`两个布尔值，只有当这两个值至少一个为true时才会进行调整ImageView大小的缩放操作；3. 设置`adjustViewBounds`为true会导致`scaleType`重置为FIT_CENTER，源码里setAdjustViewBounds方法里有这样一句：

  ```
public void setAdjustViewBounds(boolean adjustViewBounds) {
	mAdjustViewBounds = adjustViewBounds;
	if (adjustViewBounds) {
		setScaleType(ScaleType.FIT_CENTER);
	}
}
```

具体见官方文档和ImageView的源码，还有这里 [android adjustViewBounds bug?](http://stackoverflow.com/questions/10917388/android-adjustviewbounds-bug)



###ListView中Item的点击状态 [20131017]

在小米1S等手机中，如果ListView的Item非常复杂，Item的子View的PRESS_STATE可能有问题，在小米1S上的表现就是ListView的onItemClick事件会触发该Item所有子View的PRESSED和FOCUSED状态，如果某些View设置了StateListDrawable为背景，就可以看到Drawable的状态变了，设置`android:duplicateParentState` `android:descendantFocusability` `android:focusable` `android:focusableInTouchMode` `android:clickable` 均没有效果，StackOverFlow上给出的解决办法（以LinearLayout为例）：

```
public class NoPressStateLinearLayout extends LinearLayout {
	......
    @Override
    public void setPressed(boolean pressed) {
//        super.setPressed(pressed);
    }
}
```

如果上面的办法还不行，也可以这样：

```
public class NoPressStateLinearLayout extends LinearLayout {
	......
    @Override
    protected void dispatchSetPressed(boolean pressed) {
        // avoid handing on the event to the child views
    }
}
```

需要注意的是，如果子View需要处理这些状态，可能会有冲突，ListView的事件响应问题还可以参考 [ListView Tips & Tricks #4: Add Several Clickable Areas](http://cyrilmottier.com/2011/11/23/listview-tips-tricks-4-add-several-clickable-areas/)



###Shape Drawable填充问题 [20131017]

广播里需要用到一个自定义的带边框的Button背景，开始我写的是这样的：

```
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle">
    <stroke
            android:width="1dp"
            android:color="@color/soft_white"/>
    <corners
            android:radius="1dp"/>
</shape>
```

只是一个边框，大部分机子上这样就可以了，但是在部分索尼（如我的Xperia U上）和摩托的机子（如ME525）上会出现只有边框，中间部分是全黑的，原因是这个Shape只有边框，中间的区域没有东西（没有任何颜色绘制），正常情况没有东西就不会绘制任何东西，但是部分机子不知道为何会填充默认的黑色背景，解决办法是中间区域设置为透明，修改后的drawable如下：

```
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle">
   <!--内部用透明色填充-->
    <solid
            android:color="@color/transparent"/>
    <stroke
            android:width="1dp"
            android:color="@color/soft_white"/>
    <corners
            android:radius="1dp"/>
</shape>
```



###HTTP发送大量数据 [20131017]

2.0版广播支持发布带图和图片批量上传，测试过程发现在Nexus S 2.3系统上多次出现发送失败但是又没有抛异常的情况，后来找到原因了，是natalya库里的HttpRequest里使用的SimpleMultipartEntity处理Multipart类型数据有问题，代码：

```
	......
    ByteArrayOutputStream out = new ByteArrayOutputStream();
    ......
    public void addPart(final String key, final String fileName, final InputStream fin, String type, final boolean isLast){
	......
            out.write("Content-Transfer-Encoding: binaryrnrn".getBytes());
			// 这里会读取InputStream并全部写入到ByteArrayOutputStream中，数据量大时会造成OOM
            final byte[] tmp = new byte[4096];
            int l = 0;
            while ((l = fin.read(tmp)) != -1) {
                out.write(tmp, 0, l);
            }
	......
    }
```

加注释的那一行就是问题所在，上传文件时会一次性读取全部的数据流并写入到ByteArrayOutputStream，超过一定限制时会造成OOM。
[HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) 文档里说明了POST数据时，HttpURLConnection默认是读取全部数据到内存然后再通过网络发送，如果数据量过大建议使用`setFixedLengthStreamingMode(int)`（知道需要POST的数据流的精确长度时）或`setChunkedStreamingMode(int) `（无法计算或不知道需要POST的数据流的精确长度时）:


```
Posting Content

To upload data to a web server, configure the connection for output using setDoOutput(true).For best performance, you should call either `setFixedLengthStreamingMode(int)` when the body length is known in advance, or `setChunkedStreamingMode(int)` when it is not. Otherwise HttpURLConnection will be forced to buffer the complete request body in memory before it is transmitted, wasting (and possibly exhausting) heap and increasing latency.
```


###时区与日期显示问题 [20131017]

现在豆瓣API返回的日期字符串默认都是东八区的本地时间，实际显示时如果不考虑时区，在东八区以外的地区就会有问题，特别是用于和本地时间比较时，所以解析API返回的日期字符串时需要给DateFormat设置时区，这样解析出来的Date才是对的，方法如下：

```
    // API时间的默认时区
    private static final TimeZone TIME_ZONE_CHINA = TimeZone.getTimeZone("GMT+8");
    // 豆瓣API返回的时间格式
    private static final String DATE_FORMAT_STRING = "yyyy-MM-dd HH:mm:ss";
    public static Date parseDate(String dateStr) {
        DateFormat df = new SimpleDateFormat(DATE_FORMAT_STRING);
        df.setTimeZone(TIME_ZONE_CHINA);
        final ParsePosition position = new ParsePosition(0);
        return df.parse(dateStr, position);
    }
```



###用adb备份和恢复应用数据 [20131218]

日常开发过程中有时会碰到app签名不一致需要卸载重新安装的问题，重新安装之后之前的应用配置和数据就都丢失了，有个技巧可以在卸载前备份数据，在重新安装后恢复数据：

```
// 假设包名是 com.douban.shuo
// 备份包名对应app的数据
adb backup com.douban.shuo
// 从备份中恢复数据
adb restore backup.ab
```

注意：使用前提是手机已经解锁，其实adb backup/restore是系统提供的一个备份工具，可以用于整机备份，有兴趣的可以这篇文章：[Full Phone Backup without Unlock or Root](http://forum.xda-developers.com/showthread.php?t=1420351)，至于通过代码的方式备份和恢复应用数据，可以参考官方文档，国内的话由于网络原因估计不太好用：[Data Backup](http://developer.android.com/guide/topics/data/backup.html)



###APK签名检测脚本 [20131218]

写了一个简单的apk签名检测脚本，可以显示签名的详细信息：

```
#!/bin/bash
#jarsigner -verify -verbose -certs $1
INPUT=$1
OUTPUT=apk.tmp
rm -rf ${OUTPUT} 2>&1 > /dev/null
unzip ${INPUT} -d ${OUTPUT} 2>&1 > /dev/null
openssl pkcs7 -inform DER -noout -print_certs -text -in ${OUTPUT}/META-INF/CERT.RSA
rm -rf ${OUTPUT} 2>&1 > /dev/null
```



###切换Android Studio的默认构建类型 [20131218]

Android Studio--View--Tool Windows--Build Variants可以切换，一般默认建议使用release签名+开启debug的构建类型，以广播饿为例，build.gradle有如下配置：

```
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }

        beta {
            signingConfig signingConfigs.release
            debuggable true
        }

        debug {
            packageNameSuffix null
        }
    }
```

Android Studio中设置默认使用beta构建类型，build生成的apk默认使用release签名，但是BuildConfig.DEBUG为true，debug为true便于调试，使用release签名避免了给其他人测试时，或者使用qaci的构建包时签名冲突，当然，也有其它的解决办法，比如所有人使用统一的debug签名，我觉得使用buildTypes比较灵活，还可以自定义很多其它选项，比如修改包名，修改应用名，版本名，添加构建时间等。



###Idea插件QAPlug [20131017]

  IDEA有很多有用的插件，我推荐一个[QAPlug](http://qaplug.com/)，这个插件包含几个独立的模块([Findbugs/PMD/CheckStyle](http://plugins.jetbrains.com/search/index?pr=idea&search=qaplug))，可选择安装，Findbugs是一个代码缺陷检测工具，可以帮助寻找代码中的各种隐患或潜在的性能问题；PMD是一个静态代码分析工具，可以检查复杂的逻辑结构/资源使用/重复代码等问题，帮助提高代码质量；CheckStyle则专注于代码规范，可以检查命名约定/类设计/方法设计/import/空白等。这三个工具都可以指定自定义的规则，推荐每次版本发布前对代码进行一次全面的检查，有些提示是不需要修改的，但是还是能发现一些潜在的问题的，对于Android,还要做的一个检查是Lint，版本发布前对代码做一个全面的检查至少可以避免一些平时因疏忽导致的低级问题。这个插件和IDEA集成，使用起来非常简单，官网页面有详细的说明。


###跳转到第三方应用无法返回的问题 [20131218]

广播的时间线有很多地方可以点击跳转到第三方应用，比如电影/FM/小组/浏览器等 ，测试发现奇怪的问题，跳转到第三方应用后，按返回键没有返回到广播，直接返回到手机桌面了，后来发现是FLAG_ACTIVITY_NEW_TASK的问题，使用Intent跳转到其它应用，如果没有添加Intent.FLAG_ACTIVITY_NEW_TASK标志，按返回键时不能返回到上一个应用，在4.1以上系统上都是如此。所以，建议在使用Intent跳转到第三方应用的地方添加Intent.FLAG_ACTIVITY_NEW_TASK标志（选择图片等功能不需要，这时候他们位于同一个TASK）。

系统的某些功能和应用也存在这个问题，在某个应用处于前台时，如果点击通知栏的一个通知跳转到了其它应用，返回时有可能直接返回了手机桌面，而不是当前应用。



###无法直接启动其它应用的Activity [20131218]

广播的上传通知点击后会弹出一个对话框形式的Activity，有几次点击没反应，log里看到如下的错误信息：

```
12-03 15:52:33.133 611-622/? W/ActivityManager﹕ Permission Denial: starting Intent { flg=0x10000000 cmp=com.douban.shuo/.app.UploadNotifyActivity bnds=[0,102][768,230] (has extras) } from null (pid=-1, uid=10191) not exported from uid 10192
12-03 15:52:33.133 611-622/? W/ActivityManager﹕ Unable to send startActivity intent
java.lang.SecurityException: Permission Denial: starting Intent { flg=0x10000000 cmp=com.douban.shuo/.app.UploadNotifyActivity bnds=[0,102][768,230] (has extras) } from null (pid=-1, uid=10191) not exported from uid 10192
```

而且只有4.4的系统存在这个问题，查资料后发现从4.4版本开始，使用ClassName方式直接调用第三方Activity是不允许的，需要目标Activity在AndroidManifest里面明确声明了android:exported="true" 才可以，需要注意的是，不仅仅是第三方应用，只要是不同的进程，都必须声明为android:exported="true"，Activity才可以被直接调用。



###Kitkat系统WebView的换行问题 [20131218]

4.4开始，webview不会自动断行，需要在html里处理，代码里也可以这样处理：

```
if (MiscUtils.hasKitkat()) {
    settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.TEXT_AUTOSIZING);
    }
```

但是很多网页这样不解决问题，还需要在Html里处理：


```
    <pre style="word-wrap: break-word; white-space: pre-wrap;">
```

@wuzeyi 说吃喝组里是直接在html里写上了word-break:break-all;有用到可以参考。

详细的方法可以看官方的说明：

[http://developer.android.com/guide/webapps/migrating.html#Columns](http://developer.android.com/guide/webapps/migrating.html#Columns)和[https://code.google.com/p/android/issues/detail?id=62378](https://code.google.com/p/android/issues/detail?id=62378)

补充一点，4.2以后的系统，通过JS调用Java函数时，需要添加@JavascriptInterface这个Annotation，否则系统会忽略这个函数。



###Framelayout的margin设置无效的问题 [20131218]
4.0以前的系统FrameLayout的margin设置有时不生效，查资料发现原因是没有设置layout_gravity，这是4.0之前系统个一个BUG，见[https://code.google.com/p/android/issues/detail?id=28057](https://code.google.com/p/android/issues/detail?id=28057)，猜测是因为FrameLayout需要一个定位的锚点，布局文件里加入以下代码即可：

```
// 设置上下margin必须有top，设置左右margin必须有left
// layout_gravity必须有一个值，margin才会生效
android:layout_gravity="top|left"
```

代码里可以这样设置：

```
MarginLayoutParams marginParams = new MarginLayoutParams(this.getLayoutParams());  
marginParams.height = this.getMeasuredHeight();  
marginParams.width = this.getMeasuredWidth();  
marginParams.setMargins(l,t,r,b);  
LayoutParams layoutParams = new LayoutParams(marginParams);  
// 起作用的是这一行
layoutParams.gravity = Gravity.TOP|Gravity.LEFT;  
setLayoutParams(layoutParams);  
```

参考文档：

[http://blog.csdn.net/fengye810130/article/details/9147695](http://blog.csdn.net/fengye810130/article/details/9147695)
[http://stackoverflow.com/questions/5401952/framelayout-margin-not-working](http://stackoverflow.com/questions/5401952/framelayout-margin-not-working)










