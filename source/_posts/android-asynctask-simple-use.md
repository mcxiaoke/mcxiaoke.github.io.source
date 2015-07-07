Title: Android AsyncTask简单用法
Date: 2010-10-12 10:02
Author: mcxiaoke
Category: Android
Tags: Android, AsyncTask
Slug: android-asynctask-simple-use

android.os.AsyncTask  

三个泛型参数：  

* Param 任务执行器需要的数据类型  
* Progress 后台计算中使用的进度单位数据类型  
* Result 后台计算返回结果的数据类型  

有些参数是可以设置为不使用的，只要传递为Void型即可，比如 `AsyncTask<Void,Void, Void>`

四个步骤：  

* onPreExecute() 执行预处理，它运行于UI线程，可以为后台任务做一些准备工作，比如绘制一个进度条控件。  
* doInBackground(Params...) 后台进程执行的具体计算在这里实现，doInBackground(Params...)是AsyncTask的关键，此方法必须重载。在这个方法内可以使用publishProgress(Progress...)改变当前的进度值。  
* onProgressUpdate(Progress...) 运行于UI线程。如果在doInBackground(Params...)中
使用了publishProgress(Progress...)，就会触发这个方法。在这里可以对进度条控件根据进度值做出具体的响应。  
* onPostExecute(Result) 运行于UI线程，可以对后台任务的结果做出处理，结果就是doInBackground(Params...)的返回值。此方法也要经常重载，如果Result为null表明后台任务没有完成(被取消或者出现异常)。

这4个方法都不能手动调用。而且除了doInBackground(Params...)方法，其余3个方法都是被UI线程所调用的，所以要求：  
1. AsyncTask的实例必须在UI thread中创建；  
2. AsyncTask.execute方法必须在UI thread中调用；

Task只能被执行一次，多次调用时将会出现异常,而且是不能手动停止。

代码示例：

```
import android.app.Activity;
import android.os.AsyncTask;
import android.os.Bundle;
import android.util.Log;
import android.widget.TextView;

/**
 * @author mcxiaoke
 * @date 2010.10.12
 * @description AsyncTask Test
 */
public class AsyncTaskTest extends Activity {
    TextView tv;
    final String TAG = "AsyncTaskTest";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        tv = (TextView) findViewById(R.id.label);
        new MyTask().execute(6, 12, 7);

    }

    class MyTask extends AsyncTask<Integer, Integer, Integer>

    {

        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            Log.d(TAG, "onPreExecute()");
        }

        @Override
        protected Integer doInBackground(Integer... params) {
            Log.d(TAG, "doInBackground()");
            int p = 0;
            for (int index = 0; index < params.length;
                 index++) {
                int num = params[index];
                for (int j = 0; j < num;
                     j++) {
                    if (num - j <= 0) {
                        break;
                    }
                    p++;
                    publishProgress(p);
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            return p;
        }

        @Override
        protected void onProgressUpdate(Integer... progress) {
            Log.d(TAG, "onProgressUpdate()");
            tv.append("nProgress: " + progress[0]);
        }

        @Override
        protected void onPostExecute(Integer result) {
            Log.d(TAG, "onPostExecute()");
            tv.append("nFinished. Result: " + result);
        }

        @Override
        protected void onCancelled() {
            super.onCancelled();
            Log.d(TAG, "onCancelled()");
        }
    }

}

```

