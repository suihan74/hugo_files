---
title: "Firebase CrashlyticsをAndroidアプリに導入する"
description: ""
tags: ["Android", "Kotlin", "Firebase", "Crashlytics"]
date: 2021-02-08T14:59:48+09:00
lastmod: 2021-02-08T14:59:48+09:00
archives:
    - 2021
    - 2021-02
    - 2021-02-08
hide_overview: false
draft: false
---

{{<tweet 1358568317921861632>}}

ということで Firebase Crashlytics を導入しようと公式ドキュメントを見に行くと、「Android」を選択しても何故かiOS用の導入方法しか表示されない。なんでやねん。

{{<tweet 1358568525753765890>}}

というわけでここに導入方法をまとめておく。

## 導入

### Firebase プロジェクトの作成

まず、リンク先の手順に従って Firebase に対象アプリ用のプロジェクトを作成・アプリの追加を行う。

[ステップ 1: Firebase プロジェクトを作成する - Android プロジェクトに Firebase を追加する](https://firebase.google.com/docs/android/setup?hl=ja#create-firebase-project)

[Firebase コンソール](https://console.firebase.google.com/)に新規プロジェクトを作って、作ったプロジェクトのサイドメニュー上にある歯車マーク>「`プロジェクトの設定`」>「`マイアプリ`」(ページ真ん中)>「`アプリを追加`」ボタンから対象アプリを登録する。

`Androidパッケージ名`だけしっかり入力されていれば、あとは空欄でも今回の用途では問題ない。

### app/ディレクトリ直下に google-services.json を配置する

アプリ登録が完了すると、`google-services.json`がダウンロードできるので、これをAndroidプロジェクトのルートにある`app`ディレクトリ直下に配置する。

登録完了画面でDLし忘れても、「`プロジェクトの設定>マイアプリ`」からいつでもDLできる。

.gitignoreにも追加して一応表に出さないようにしておくなどする。

### Crashlytics の有効化

登録が完了したら、「`サイドメニュー/リソースとモニタリング`」の「`Crashlytics`」を選択し、遷移したページの上の方にある「`Crashlyticsを有効化する`」を押す。

プログレスバーが回り出したら、ブラウザはそのままにして Android Studio での作業に移る。

### アプリに Firebase + Crashlytics を導入する

Androidプロジェクトに Firebase を追加する。

```gradle:(project)build.gradle
buildscript {
    repositories {
        google()  // <- 無かったら追加
    }

    dependencies {
        // ...

        // 2行追加
        classpath 'com.google.gms:google-services:4.3.5'
        classpath 'com.google.firebase:firebase-crashlytics-gradle:2.4.1'
    }
}

allprojects {
    // ...

    repositories {
        google()  // <- 無かったら追加
    }
}
```

```gradle:(app)build.gradle
// 2行追加
apply plugin: 'com.google.gms.google-services'
apply plugin: 'com.google.firebase.crashlytics'

android {
    // ...
}

dependencies {
    // ...

    // Firebase
    // Import the Firebase BoM
    implementation platform('com.google.firebase:firebase-bom:26.3.0')
    implementation 'com.google.firebase:firebase-crashlytics-ktx'
    implementation 'com.google.firebase:firebase-analytics-ktx'
}
```

Firebase BoM を導入することで、firebaseの各モジュールは勝手に適切なバージョンに設定してくれる。  
(必須ではないので、BoMを外して各ライブラリのバージョンを自分で指定できなくもない)

## 初回だけわざとクラッシュを発生させる

起動処理以外のどこか適当な場所で、キャッチしない例外送出を行ってアプリをクラッシュさせる。(ボタンを押したらクラッシュするようにする、など)

クラッシュしただけで Crashlytics のレポートに反映されない場合はもう一度アプリを(正常に)起動させる。

ブラウザの Crashlytics の画面がレポート画面に変わったら成功。

---

## デバッグビルドでレポート送信しないようにする

デバッグビルド(または他のビルド構成)で Crashlytics にレポートを送信しないようにするには、アプリレベルの`build.gradle`に次のような設定を記述する。

```gradle:(app)build.gradle
android {
    // ...
    buildTypes {
        release {
            // ...
        }
        debug {
            firebaseCrashlytics {
                mappingFileUploadEnabled false
            }
        }
    }
}
```

## アプリ内で処理した例外をレポート送信する

アプリ内で適切に処理するなり握り潰すなりした例外も明示的に Crashlytics に送信することができる。

```kt
runCatching {
    throw RuntimeException("test")
}.onFailure { e ->
    // 送信
    FirebaseCrashlytics.getInstance().recordException(e)

    // ...
}
```
