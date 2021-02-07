---
title: "古めのAndroidバージョンをサポートしたままJava 8+ APIを使う方法"
description: "今さらcore library desugaring知った話"
tags: ["Android","Kotlin","Java","Gradle","java.time"]
date: 2021-02-08T00:30:00+09:00
lastmod: 2021-02-08T00:30:00+09:00
archives:
    - 2021
    - 2021-02
    - 2021-02-08
hide_overview: false
draft: false
---

## AndroidのAPIレベルとJavaバージョン

Androidの最小APIレベルを低めに設定している場合、そのままでは Java 8 言語機能を利用できず、Java 7 言語機能を使用するために使用できる言語機能に制限がかかる場合がある。

たとえば、日時情報を扱う`java.time`に含まれる`LocalDateTime`とか`ZonedDateTime`とかのクラスは最小APIレベル26以上に設定しなければそのままでは使用できない。

この場合[ThreeTen Android Backport](https://github.com/JakeWharton/ThreeTenABP)などのバックポートライブラリを使用するなどして対応することができるが、用途をAndroidに限らないライブラリを作って使いたい場合などに話がややこしくなる。

じゃあAndroid専用ではない[ThreeTen backport](https://github.com/ThreeTen/threetenbp)を使用すればいいかというと、これはこれでAndroid上で動かす場合にタイムゾーン情報の扱いに関して極めて非効率なところがあるという[^*]。

## Android Gradle プラグイン 4 以上での解決方法

`Android Gradle プラグイン 4.0.0`以上を使用している場合、次のように`build.gradle(appレベル)`を記述することで最小APIレベルが小さくても Java 8 の言語機能を使用することができるようになる。

```gradle:(app)build.gradle
android {
    ~~ 他省略 ~~

    defaultConfig {
        // minSdkVersionが20以下なら必要
        multiDexEnabled true
    }

    compileOptions {
        // Flag to enable support for the new language APIs
        coreLibraryDesugaringEnabled true

        // Sets Java compatibility to Java 8
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.1.1'

    ~~ 他省略 ~~
}
```

[Java 8 言語機能と API を使用する | Android デベロッパー | Android Developers](https://developer.android.com/studio/write/java8-support#library-desugaring)

[desugar で使用可能な Java 8+ API | Android デベロッパー | Android Developers](https://developer.android.com/studio/write/java8-support-table)

`Android Gradle プラグイン 4.0.0`がリリースされたのが2020年4月なので、ほぼ一年気づいていなかったというアンテナの低さを露呈しているわけだが。

---

## 余談 kotlinx-datetime

さらにKotlin使っていてJVMに限らないマルチプラットフォーム対応で日時情報を扱いたい場合、Kotlin 1.4.0以降なら次のようなライブラリが開発されているらしい。  
これもminSDKVersionが小さいAndroidアプリに組み込む場合先述のcore library desugaringが必要となる。

{{<github "Kotlin/kotlinx-datetime">}}

JSやNativeを考慮せず単にJVM上で動けばいいだけのものなら中身はただ`java.time`っぽいので、特別何か嬉しいことがある訳ではなさそうだが、一応。

---

[^*]: https://github.com/JakeWharton/ThreeTenABP#why-not-use-threetenbp
