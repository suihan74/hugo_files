---
title: "kotlinx.serializationことはじめ"
description: "kotlin用のJsonシリアライザを使ってみる"
tags: ["kotlin", "kotlinx", "serialization"]
date: 2020-08-06T21:40:00+09:00
lastmod: 2020-08-06T21:40:00+09:00
archives:
    - 2020
    - 2020-08
    - 2020-08-06
hide_overview: false
draft: false
---

## この記事の情報

```kotlinx.serialization 0.20.0```について書かれています。  
アップデートによって内容が古くなる可能性が高いです。

## 依存関係の設定

必要なところだけ

### build.gradle (Project)

```gradle
buildscript {
    ext.kotlin_version = '1.3.72'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:4.0.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-serialization:$kotlin_version"
    }
}
```

### build.gradle (app)

```gradle
apply plugin: 'kotlinx-serialization'
// ほか省略

android {
    // 省略
}


dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-serialization-runtime:0.20.0'

    // ほか省略
}
```

### proguard-rules&#046;pro

```txt
##---------------Begin: proguard configuration for kotlinx.serialization  ----------

-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.SerializationKt
-keep,includedescriptorclasses class com.yourcompany.yourpackage.**$$serializer { *; } # <-- change package name to your app's
-keepclassmembers class com.yourcompany.yourpackage.** { # <-- change package name to your app's
    *** Companion;
}
-keepclasseswithmembers class com.yourcompany.yourpackage.** { # <-- change package name to your app's
    kotlinx.serialization.KSerializer serializer(...);
}
##---------------End: proguard configuration for kotlinx.serialization  ----------
```

- ```# <-- change package name to your app's``` : 自分のアプリのパッケージ名に書き換えるのを忘れずに

---

## データクラス例

```kt
@Serializable
data class Hoge(
    @SerialName("strstr")
    val str : String,
    val num : Int,

    @ContextualSerialization
    val b : Boolean,

    @Serializable(with = LocalDateTimeSerializer::class)
    val date: LocalDateTime,

    val nullableStr : String? = null
) {
    val message : String by lazy {
        str + num + b + date + (nullableStr ?: "null")
    }
}
```

- ```@SerialName("")``` : Jsonでのキー名が異なる場合に指定する  
  Gsonなどのように「すべてのキー名をJsonではスネークケースで扱う」とかは自動でできないっぽいので、現状いちいちこいつを設定する必要がありそう。

- ```@ContextualSerialization``` : 実際にシリアライズ・デシリアライズを行う際に処理方法(シリアライザ)を決定する

- ```@Serializable(with = xxSerializer::class)``` : シリアライザをプロパティ側で指定する

- nullable : 無くてもいいプロパティには```@Optional```を付けるらしいが、```nullable```にはべつに必要ないっぽい

- lazyプロパティ : Gsonなどと違い、明示的に回避する必要はない

## ユーザー定義のシリアライザ例

### シリアライザを用意しないとそもそも扱えないデータ型のためのシリアライザ

```kt
/** LocalDateTime用シリアライザ */
@Serializer(forClass = LocalDateTime::class)
object LocalDateTimeSerializer : KSerializer<LocalDateTime> {
    private val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS")

    override val descriptor : SerialDescriptor =
        PrimitiveDescriptor("LocalDateTime", PrimitiveKind.STRING)

    override fun serialize(encoder: Encoder, obj: LocalDateTime) {
        encoder.encodeString(obj.format(formatter))
    }

    override fun deserialize(decoder: Decoder) : LocalDateTime =
        LocalDateTime.parse(decoder.decodeString(), formatter)
}
```

### 元からシリアライザが用意されている型に別のものを用意する

```kt
/** 真偽値が何故か0と1で扱われているやつが渡されてくるので変換するやつ */
@Serializer(forClass = Boolean::class)
object UserBooleanSerializer : KSerializer<Boolean> {
    override val descriptor : SerialDescriptor =
        PrimitiveDescriptor("BooleanInt", PrimitiveKind.INT)

    override fun serialize(encoder: Encoder, obj: Boolean) {
        encoder.encodeInt(if (obj) 1 else 0)
    }

    override fun deserialize(decoder: Decoder) : Boolean =
        decoder.decodeInt() == 1
}
```

- descriptorに既存の物と被らないように名前を与える必要がある  
  (```PrimitiveDescriptor("Boolean", PrimitiveKind.INT)```ではダメ)

---

### シリアライズ・デシリアライズしてみる

```kt
fun example() {
    // サンプル用データ
    val hoge = Hoge("hello world.", 1234, true, LocalDateTime.now())

    // 型ごとのシリアライザを一括で指定する
    val context = serializersModuleOf(mapOf(
        Boolean::class to BooleanSerializer,
        LocalDateTime::class to LocalDateTimeSerializer
    ))
    // この記事の場合ではLocalDateTimeSerializerはプロパティ側で指定されているのでここでは必要ないが、複数の型・シリアライザペアをcontextに登録する例として無駄に書いている

    val json = Json(JsonConfiguration.Stable)

    // ■シリアライズ
    val jsonData = json.stringify(Hoge.serializer(), hoge)

    println(jsonData)
    // ==> {"strstr":"hello world.","num":1234,"b":1,"date":"2020-08-06 21:02:29.571","nullableStr":null}

    // ■デシリアライズ
    val restored = json.parse(Hoge.serializer(), jsonData)

    println(restored.message)
    // ==> hello world.1234true2020-08-06T21:02:29.571null

    // ■リストの扱い方
    val jsonListData = json.stringify(Hoge.serializer().list, listOf(hoge, hoge))

    println(jsonListData)
    // ==> [{"hogegege":"hello world.","num":1234,"b":1,"date":"2020-08-06 21:32:50.745","nullableStr":null},{"hogegege":"hello world.","num":1234,"b":1,"date":"2020-08-06 21:32:50.745","nullableStr":null}]
}
```

- ```serializersModuleOf()```で型に対応するシリアライザを指定できる。  
  これはプロパティ側で```@ContextualSerialization```が指定されている場合に使用される。(ここで指定していてもアノテーションを忘れるとシリアライザが差し変わらない)

- ```Hoge.serializer().list``` : リストを扱うときはこうする。マップなら```map```  
  パッケージは```kotlinx.serialization.builtins.*```のものをインポートすることに注意。

- 結果をnullableにする場合 : ```Hoge.serializer().nullable```
