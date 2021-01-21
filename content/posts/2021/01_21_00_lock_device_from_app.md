---
title: "アプリから端末をロックして消灯する方法"
description: "端末管理アプリとして登録することでアプリから端末ロックを使用する"
tags: ["Android","Kotlin"]
date: 2021-01-21T21:30:00+09:00
lastmod: 2021-01-21T21:30:00+09:00
archives:
    - 2021
    - 2021-01
    - 2021-01-21
hide_overview: false
draft: false
---

## 前提

現在、この記事で説明する端末管理機能を使用する以外の方法でアプリから直接端末をロックしスリープ状態にする方法は存在しないっぽい(カジュアルに実行できてしまうと危険なため)  
それでも何らかの事情でアプリから端末ロック&消灯を行いたい場合は、以下に説明するようにアプリを端末管理アプリとしてユーザーに許可してもらう必要がある。

## 端末管理アプリ

アプリが「端末管理アプリ」として振舞うことをユーザーに許可してもらうことで、端末を強制的にロックするとか端末初期化とかパスワード再設定とかといった(より危険な)部分をいじくる機能をアプリに付与することができる。

この端末管理機能の大部分はAndroid 9で非推奨となり、Android 10以降では実行しようとしても`SecurityException`が発生するようになっている。  

[Device admin deprecation | Android Enterprise | Google Developers](https://developers.google.com/android/work/device-admin-deprecation)

ただし今回行うプログラム側からの端末ロックと、他には端末データ消去やパスワードリセットなどは紛失時の情報保護などが主目的ではあるのだろうが、引き続きサポートされるというようなことがリンク先には書いてある。

## アプリを端末管理アプリとして扱えるようにする

二点用意する。

- 使用する端末管理機能を記述したxmlファイル

- `DeviceAdminReceiver`を継承したレシーバ

### アプリが使用する端末管理機能の宣言

アプリが使用する機能のみを記述する。`<force-lock />`のほか、`<wipe-data />`や`<reset-password />`が使用できる。(他にもあるが先述の通り現在は使用できない)

```xml:res/xml/device_admin.xml
<?xml version="1.0" encoding="utf-8"?>
<device-admin>
    <uses-policies>
        <force-lock />
    </uses-policies>
</device-admin>
```

### 端末管理機能の有効状態変化を受け取るレシーバ

```kt:receiver/MyDeviceAdmingReceiver.kt
class MyDeviceAdmingReceiver : DeviceAdminReceiver() {
    override fun onEnabled(context: Context, intent: Intent) {
        super.onEnabled(context, intent)
        // 有効化された
    }

    override fun onDisabled(context: Context, intent: Intent) {
        super.onDisabled(context, intent)
        // 無効化された
    }
}
```

他にも色々あるメソッドをオーバーライドすることで管理アプリとしての使用可否や各種状態の変化を受け取ることができるが、今回は特に必要ないので割愛する。  
`onEnabled`と`onDisabled`についても、機能使用時に端末管理アプリとして登録されているかどうかを明示的にチェックする場合はそれほど必要でもないので、ぶっちゃけ中身は空でもいいような気もする。

マニフェストファイルにレシーバを記述する。

```xml:AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.suihan74.example">

    <application
        android:allowBackup="true"
        android:name=".Application"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:fullBackupContent="true"
        android:theme="@style/Theme.HogeTheme" >

        <!-- ほか省略 -->

        <receiver
            android:name=".receiver.MyDeviceAdminReceiver"
            android:label="@string/app_name"
            android:description="@string/device_admin_desc"
            android:permission="android.permission.BIND_DEVICE_ADMIN">
            <meta-data android:name="android.app.device_admin"
                android:resource="@xml/device_admin" />
            <intent-filter>
                <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
            </intent-filter>
        </receiver>
    </application>
</manifest>

```

`android:label`と`android:description`はシステムの「端末管理アプリ」設定画面で表示されるタイトルと説明文である。

これでアプリがシステムの「端末管理アプリ」設定画面に候補として表示されるようになる。  
レシーバを使用して有効状態を受け取ってもいいが、次に説明するようにアプリ内で状態を直接確認することもできる。

## アプリからシステムの「端末管理アプリ」設定画面を開いて許可を求める

アプリが端末管理アプリ化されていることを前提とした振る舞いをする場合、端末管理機能を使用する前に(起動直後のスプラッシュ画面などで)システムの設定画面に遷移してユーザーに許可を求める必要がある。

今回の例では単にアプリ起動時に確認するようにしているが、機能使用時に毎回確認するなど必要に応じて書き換えればいいと思う。

### アプリからシステムの「端末管理アプリ」設定画面を開く

```kt:SplashActivity.kt
/** 起動時処理用のアクティビティ */
class SplashActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // ~~ 必要なら画面を用意 ~~ //

        if (requestDeviceAdmin()) {
            // アプリ本体のアクティビティを開始
            launchContentsActivity()
        }
    }

    /**
     * 端末管理アプリとして未許可状態なら、ユーザーに許可してもらう
     */
    private fun requestDeviceAdmin() : Boolean {
        val dpm = getSystemService(Service.DEVICE_POLICY_SERVICE) as DevicePolicyManager
        val componentName = ComponentName(this, MyDeviceAdminReceiver::class.java)

        return dpm.isAdminActive(componentName).onFalse {
            // システムの「端末管理アプリ」設定画面を開く
            val intent = Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN).also {
                it.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN, componentName)
            }
            startActivityForResult(intent, RequestCode.DEVICE_POLICY_MANAGER.ordinal)
        }
    }
}
```

`Boolean#onFalse`はこちらで勝手に用意した「`false`時にブロックを実行してから`false`を返す」拡張関数。

`DevicePolicyManager#isAdminActive(ComponentName)`でアプリが端末管理アプリとして利用可能かどうかを確認し、許可されていない状態ならアプリ用の使用許可画面に直接遷移するようにしている。

`ComponentName`は用意した`MyDeviceAdminReceiver`を使用して生成したものを使う必要がある。

## アプリから端末をロックする

以上をもってようやくアプリ側から端末ロックを行えるようになる。  
どうやらロックと同時に画面も消灯される。

```kt
/** 強制的にロックして消灯する */
private fun sleep(context: Context) {
    try {
        val dpm = context.getSystemService(Service.DEVICE_POLICY_SERVICE) as DevicePolicyManager
        dpm.lockNow()
    }
    catch (e: Throwable) {
        Log.e("DevicePolicyManager", Log.getStackTraceString(e))
    }
}
```

下手をすると「画面が点くたびに即座にロックが実行されてしまう」とかそういう状況を作れてしまうので、元から悪意があるとかそういう事でないならこの機能の使用には注意が必要である。  
