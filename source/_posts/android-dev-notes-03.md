title: Android开发杂记（三）
date: 2014-06-30 14:00
author: mcxiaoke
categories: 
- Android
tags: 
- Android
- Tips
#slug: android-dev-notes-03
---
这是另一部分的Android开发笔记整理

###如何完全不显示一个Activity的UI
可以使用 `android:theme="@android:style/Theme.NoDisplay"` 但是有一个注意事项，见
[how to completely get rid of an activity's GUI](http://stackoverflow.com/questions/4551868/how-to-completely-get-rid-of-an-activitys-gui-avoid-a-black-screen)

###如何移除ActionBar底部的阴影
自定义ActionBar的Style，使用 `<item name="android:windowContentOverlay">@null</item>` 如果是使用ActionBarSherlock或ActionBarCompat，还需要添加 `<item name="windowContentOverlay">@null</item>`，参考：[How can I have a drop shadow on my ActionBar](http://stackoverflow.com/questions/11448679/how-can-i-have-a-drop-shadow-on-my-actionbar-actionbarsherlock)

###使用Gradle时私密存储签名密钥
参考这个Gist：[build.gradle](https://gist.github.com/mcxiaoke/8450376)

###透明的Activity退出后，没有真正finish的问题
具体见 [这里的分析](http://blog.sina.com.cn/s/blog_601cbd070100npf8.html)
android:windowShowWallpaper = true的作用，这并不是在后面显示桌面，是配置Activity的背景为桌面背景

###Background和Seletor必须使用真实的Drawable

否则，有些三星和摩托的机子上会没有背景，显示纯黑色，定义在colors.xml里的伪drawble不行

```xml
<color name="mail_published_time_color">#bcbcbc</color>
<drawable name="ab_bg_black">#aa191919</drawable>
```


必须是真实的图片drawable或者定义好的shape

###合并多个git仓库，保留commit记录的方法
详情见[How to import existing GIT repository into another](http://stackoverflow.com/questions/1683531/how-to-import-existing-git-repository-into-another)和[合并已存在的git仓库](https://github.com/deercoder/Linux/blob/master/Git/git_merge_local_repos.md)

###WebView浏览位置的保存和恢复

```
    // 保存位置
    public float savePosition() {
        LogUtils.v(TAG, "savePosition()");
        float top = getTop();
        float contentHeight = getContentHeight();
        float scrollY = getScrollY();
        mSavedPosition = (scrollY - top) / contentHeight;
        return mSavedPosition;
    }
    
    // 恢复位置
    public float restorePosition() {
        LogUtils.v(TAG, "restorePosition()");
        if (mSavedPosition > 0f) {
            float top = getTop();
            float contentHeight = getContentHeight();
            float height = contentHeight - top;
            float positionInViewPort = height * mSavedPosition;
            int positionY = Math.round(top + positionInViewPort);
            scrollTo(0, positionY);
        }
        return mSavedPosition;
    }
    
```

###使用拨号键盘的SecretCode功能

Android的拨号键盘有一些特殊的定义键，可以启动自定义的Intent，用法：

```
<receiver android:name=".receiver.DiagnoserReceiver">
    <intent-filter>
        <action android:name="android.provider.Telephony.SECRET_CODE"/>
        <data android:scheme="android_secret_code" android:host="111222"/>
    </intent-filter>
</receiver>
```

参考资料：
[Create a secret doorway to your app](http://udinic.wordpress.com/2013/05/17/create-a-secret-doorway-to-your-app/)
[Secret_Star_Codes](https://code.google.com/p/android-roms/wiki/Secret_Star_Codes)


###阻止点击DrawLayout时事件传递到下一层  
方法是给Drawlayout添加一个OnClickListener
参考资料：
[How do I keep DrawerLayout from passing touch events to the underlying view](http://stackoverflow.com/questions/18811973/android-how-do-i-keep-drawerlayout-from-passing-touch-events-to-the-underlying)

###没有root的情况下如何adb pull /data/data/package/下的数据
下面是一个查看应用数据库的例子脚本：

```
PACKAGE_NAME=com.your.package
DB_NAME=data.db
rm -rf ${DB_NAME}
adb shell "run-as ${PACKAGE_NAME} chmod 666 /data/data/${PACKAGE_NAME}/databases/${DB_NAME}"
adb pull /data/data/${PACKAGE_NAME}/databases/${DB_NAME} /tmp/
adb shell "run-as ${PACKAGE_NAME} chmod 600 /data/data/${PACKAGE_NAME}/databases/${DB_NAME}"
sqlite3 /tmp/${DB_NAME
```

分析见：
 [android adb, retrieve database using run-as](http://stackoverflow.com/questions/18471780/android-adb-retrieve-database-using-run-as)
[Access Android app data without root](http://blog.shvetsov.com/2013/02/access-android-app-data-without-root.html)

###快速获取电池电量的方法

```
    public static Intent getBatteryStatus(Context context) {
        Context appContext = context.getApplicationContext();
        return appContext.registerReceiver(null,
                new IntentFilter(Intent.ACTION_BATTERY_CHANGED));
    }

    public static String getBatteryInfo(Context context, Intent batteryIntent) {
        int status = batteryIntent.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
        boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                status == BatteryManager.BATTERY_STATUS_FULL;
        int chargePlug = batteryIntent.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
        boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
        boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;

        int level = batteryIntent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);
        int scale = batteryIntent.getIntExtra(BatteryManager.EXTRA_SCALE, -1);

        float batteryPct = level / (float) scale;
        return "Battery Info: isCharging=" + isCharging
                + " usbCharge=" + usbCharge + " acCharge=" + acCharge
                + " batteryPct=" + batteryPct;
    }
```

分析见：[Get battery level before broadcast receiver responds for Intent.ACTION_BATTERY_CHANGED](http://stackoverflow.com/questions/3661464/get-battery-level-before-broadcast-receiver-responds-for-intent-action-battery-c)

###ActionBar的Title是否可以点击的问题
4.2之前和之后这个有变化，4.2之前只有Icon可以点击，如果没有Icon，Title就无法点击，4.2之后是Title和Icon一起作为点击区域
分析见：[Action Bar icon as up enabled not the title](http://stackoverflow.com/questions/16209963/action-bar-icon-as-up-enabled-not-the-title/16216966#16216966) 

###Webview滚动时背景闪烁的问题
因是渲染帧数不够
如果使用软件渲染，看下文：[Strange webview black blinking when scrolling](http://stackoverflow.com/questions/17315815/strange-webview-black-blinking-when-scrolling)，或者启用硬件加速


###Java中使用try catch的性能问题
使用try catch并没有额外的性能损耗，只有异常真正发生时才会有性能损耗，详细分析见：[Should java try blocks be scoped as tightly as possible](http://stackoverflow.com/questions/2633834/should-java-try-blocks-be-scoped-as-tightly-as-possible)



