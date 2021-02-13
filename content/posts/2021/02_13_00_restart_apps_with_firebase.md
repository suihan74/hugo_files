---
title: "メインプロセス以外でFirebaseサービスの使用を避ける"
description: "Firebaseを使用しているアプリをプログラム側から再起動するときなどに必要な処理"
tags: ["Android","Kotlin","Firebase"]
date: 2021-02-13T14:37:58+09:00
lastmod: 2021-02-13T14:37:58+09:00
archives:
    - 2021
    - 2021-02
    - 2021-02-13
hide_overview: false
draft: false
---

## 例) アプリの再起動

Android Q 以降を対象にしているアプリでは、アプリ再起動時に`AlarmManager`を使った方法は使用できないため、  
たとえば次のように別プロセスで起動した再起動処理専用の`RestartActivity`からメインプロセスを終了した後に再び起動する、というような処理を行うことで実現できる。

### RestartActivity

```kt:RestartActivity.kt
class RestartActivity : Activity() {
    companion object {
        const val EXTRA_MAIN_PID = "RestartActivity.EXTRA_MAIN_PID"

        fun createIntent(context: Context) = Intent().apply {
            setClass(context, RestartActivity::class.java)
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
            putExtra(EXTRA_MAIN_PID, Process.myPid())
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // メインプロセス終了
        val mainPid = intent.getIntExtra(EXTRA_MAIN_PID, -1)
        Process.killProcess(mainPid)

        // メインアクティビティ再起動
        val restartIntent = Intent(applicationContext, SplashActivity::class.java).apply {
            action = Intent.ACTION_MAIN
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
        }
        applicationContext.startActivity(restartIntent)

        // RestartActivity終了
        finish()
        Process.killProcess(Process.myPid())
    }
}
```

```xml:AndroisManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.suihan74.hoge">

    <application
            android:name=".HogeApplication"
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:label="@string/app_name"
            android:supportsRtl="true"
            android:theme="@style/AppTheme.Light">

        <!-- 他省略 -->

        <activity
                android:name=".RestartActivity"
                android:excludeFromRecents="true"
                android:exported="false"
                android:launchMode="singleInstance"
                android:process=":restart_process"
                android:theme="@android:style/Theme.Translucent.NoTitleBar"/>
    </application>

</manifest>
```

### 使用箇所

```kt
val intent = RestartActivity.createIntent(context)
startActivity(intent)
```

### 参考

[Android Q でアプリを強制的に再起動する方法 - Qiita](https://qiita.com/Shiozawa/items/85f078ed57aed46f6b69)

## Firebase

しかし、これと Crashlytics など Firebase のサービスを併用すると、`RestartActivity`が実行されるプロセスでのFirebaseサービスのインスタンス取得に失敗する。

```kt:HogeApplication.kt
class HogeApplication : Application() {
    // ...

    override fun onCreate() {
        super.onCreate()

        // デバッグビルドでクラッシュレポートを送信しない
        FirebaseCrashlytics.getInstance()       // <-- ここで例外発生
            .setCrashlyticsCollectionEnabled(
                BuildConfig.DEBUG.not()
            )
    }
}
```

Logcatを見ると次のようなエラーメッセージ。

```text
E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.suihan74.satena:restart_process, PID: 27168
    java.lang.RuntimeException: Unable to create application com.suihan74.satena.SatenaApplication: java.lang.IllegalStateException: Default FirebaseApp is not initialized in this process com.suihan74.satena:restart_process. Make sure to call FirebaseApp.initializeApp(Context) first.
        at android.app.ActivityThread.handleBindApplication(ActivityThread.java:5876)
        at android.app.ActivityThread.access$1100(ActivityThread.java:199)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1650)
        at android.os.Handler.dispatchMessage(Handler.java:106)
        at android.os.Looper.loop(Looper.java:193)
        at android.app.ActivityThread.main(ActivityThread.java:6669)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:858)
Caused by: java.lang.IllegalStateException: Default FirebaseApp is not initialized in this process com.suihan74.satena:restart_process. Make sure to call FirebaseApp.initializeApp(Context) first.
        at com.google.firebase.FirebaseApp.getInstance(FirebaseApp.java:183)
        at com.google.firebase.crashlytics.FirebaseCrashlytics.getInstance(FirebaseCrashlytics.java:229)
        at com.suihan74.satena.SatenaApplication.onCreate(SatenaApplication.kt:133)
        at android.app.Instrumentation.callApplicationOnCreate(Instrumentation.java:1154)
        at android.app.ActivityThread.handleBindApplication(ActivityThread.java:5871)
```

`Make sure to call FirebaseApp.initializeApp(Context) first.`

Firebase は基本的にはメインプロセス以外での使用をサポートしておらず、別プロセスで使用する場合には明示的に初期化処理を行う必要がある。

## メインプロセスで実行されているか確認する

今回のようにただ単純に再起動するだけのために別プロセスを作成する場合、`Application`内で通常行う初期化処理は基本的には不要であるので、行わないようにすれば良い。

また、何らかの他の用途で別プロセスを使用する場合、Firebase の機能を使用する可能性があるのであれば別プロセスの場合だけ明示的に`FirebaseApp.initializeApp(Context)`を実行するようにする。

どちらにしても、「生成された`Application`がメインプロセスで作られたか否か」を判定する機能が必要になるので、次のようなメソッドなり拡張関数なりを用意する。

```kt:HogeApplication.kt
/**
 * メインプロセスで実行されているかを確認する
 *
 * @throws NullPointerException `ActivityManager`が取得できない
 */
private fun isMainProcess(context: Context) : Boolean {
    val manager = context.getSystemService<ActivityManager>()!!
    val pid = android.os.Process.myPid()
    return manager.runningAppProcesses.firstOrNull {
        it.processName == BuildConfig.APPLICATION_ID && it.pid == pid
    } != null
}
```

たとえば「メインプロセス以外では不必要な初期化処理を行わない」のであれば、`onCreate()`メソッドで次のように使用する。

```kt:HogeApplication.kt
override fun onCreate() {
    super.onCreate()

    if (!isMainProcess(this)) {
        return
    }

    // デバッグビルドでクラッシュレポートを送信しない
    FirebaseCrashlytics.getInstance()
        .setCrashlyticsCollectionEnabled(
            BuildConfig.DEBUG.not()
        )
}
```
