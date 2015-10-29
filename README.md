# leakcanarySample_androidStudio

工程包括：

0. LeakCanary库代码
1. LeakCanaryDemo示例代码



使用步骤：

 0. 将LeakCanary import 入自己的工程

 1. 添加依赖： 

	```compile project(':leakcanary')```
	
 2. 在Application中进行配置
	
	```
	public class ExampleApplication extends Application {

	  ......
	  //在自己的Application中添加如下代码
    public static RefWatcher getRefWatcher(Context context) {
        ExampleApplication application = (ExampleApplication) context
                .getApplicationContext();
        return application.refWatcher;
    }

	  //在自己的Application中添加如下代码
    private RefWatcher refWatcher;

    @Override
    public void onCreate() {
        super.onCreate();
        ......
        	//在自己的Application中添加如下代码
        refWatcher = LeakCanary.install(this);
        ......
    }

    .....
}

	```
	
 3. 在Activity中进行配置

```
public class MainActivity extends AppCompatActivity {

   	......
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
			//在自己的应用初始Activity中加入如下两行代码
        RefWatcher refWatcher = ExampleApplication.getRefWatcher(this);
        refWatcher.watch(this);

        textView = (TextView) findViewById(R.id.tv);
        textView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startAsyncTask();
            }
        });

    }

    private void async() {

        startAsyncTask();
    }

    private void startAsyncTask() {
        // This async task is an anonymous class and therefore has a hidden reference to the outer
        // class MainActivity. If the activity gets destroyed before the task finishes (e.g. rotation),
        // the activity instance will leak.
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                // Do some slow work in background
                SystemClock.sleep(20000);
                return null;
            }
        }.execute();
    }


}

```

 4. 在AndroidMainfest.xml 中进行配置,添加如下代码

```
        <service
            android:name="com.squareup.leakcanary.internal.HeapAnalyzerService"
            android:enabled="false"
            android:process=":leakcanary" />
        <service
            android:name="com.squareup.leakcanary.DisplayLeakService"
            android:enabled="false" />

        <activity
            android:name="com.squareup.leakcanary.internal.DisplayLeakActivity"
            android:enabled="false"
            android:icon="@drawable/__leak_canary_icon"
            android:label="@string/__leak_canary_display_activity_label"
            android:taskAffinity="com.squareup.leakcanary"
            android:theme="@style/__LeakCanary.Base" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>


```

