---
title: "Kotlin 1.5.20でHiltを使う際に必要なこと(2021.06.28時点)"
description: "現時点だとなんか色々バグっててめんどい"
tags: ["Kotlin","Android","Hilt"]
date: 2021-06-28T22:37:31+09:00
lastmod: 2021-06-28T22:37:31+09:00
archives:
    - 2021
    - 2021-06
    - 2021-06-28
hide_overview: false
draft: false
---

先日リリースされた[Kotlin 1.5.20](https://blog.jetbrains.com/kotlin/2021/06/kotlin-1-5-20-released/)だが、既存のプロジェクトに何も考えず突っ込んだらビルドでズッコケたので対処方法のメモ。

2021年6月28日時点の内容なので、いまに要らない情報になるんじゃないかと思います。

---

## Android Gradle PluginとHiltのバージョン

`com.android.tools.build:gradle:7.1.0-alpha02`と`com.google.dagger:hilt-android-gradle-plugin:2.37`を使用する。`com.android.tools.build:gradle:7.0.0-beta04`でやろうとすると`java.lang.NoSuchMethodError`で失敗する。

[Gradle crashes with: Hilt  + AGP 4.2.0-beta04 · Issue #2337 · google/dagger](https://github.com/google/dagger/issues/2337)

```gradle:(project)build.gradle
buildscript {
    ext.kotlin_version = "1.5.20"
    repositories {
        google()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.1.0-alpha02'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-serialization:$kotlin_version"
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.37'
    }
}
```

## build.gradle(:app)の追記

[Support for Kotlin 1.5.20 · Issue #2684 · google/dagger](https://github.com/google/dagger/issues/2684)によると、Hilt Gradle Pluginによって本来自動的に設定されるはずのオプションがKotlin 1.5.20のバグによって設定されないため、手動で記述しておく必要があるらしい。

```gradle:(app)build.gradle
android {
    ...
    kapt {
        javacOptions {
            // These options are normally set automatically via the Hilt Gradle plugin, but we
            // set them manually to workaround a bug in the Kotlin 1.5.20
            option("-Adagger.fastInit=ENABLED")
            option("-Adagger.hilt.android.internal.disableAndroidSuperclassValidation=true")
        }
    }
}
```
