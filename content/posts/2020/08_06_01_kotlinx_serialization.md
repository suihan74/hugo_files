---
title: "kotlinx.serializationことはじめ"
description: "kotlin用のJsonシリアライザを使ってみる"
tags: ["kotlin", "kotlinx", "serialization"]
date: 2020-08-06T21:40:00+09:00
archives:
    - 2020
    - 2020-08
    - 2020-08-06
hide_overview: false
draft: false
---

## この記事の情報

```kotlinx.serialization 1.0.1```について書かれています。  
アップデートによって内容が古くなる可能性があります。

公式のドキュメントがかなりしっかり書いてあるので、そっちを読んだ方がいいとは思います。

[kotlinx.serialization/docs at master · Kotlin/kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization/tree/master/docs)

## 追記 (2021-01-24)

バージョン`1.0.1`時点での内容に更新。

---

## 依存関係の設定

必要なところだけ。とりあえずAndroidアプリプロジェクトの想定で。

```gradle:[project]build.gradle
buildscript {
    ext.kotlin_version = '1.4.21'
    repositories {
        jcenter()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-serialization:$kotlin_version"
    }
}
```

`serialization 1.0.1`では`kotlin 1.4.0`以上である必要がある。

```gradle:[app]build.gradle
apply plugin: 'kotlinx-serialization'
// ほか省略

android {
    // 省略
}

dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.0.1'

    // ほか省略
}
```

```:proguard-rules.pro
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt # core serialization annotations

# kotlinx-serialization-json specific. Add this if you have java.lang.NoClassDefFoundError kotlinx.serialization.json.JsonObjectSerializer
-keepclassmembers class kotlinx.serialization.json.** {
    *** Companion;
}
-keepclasseswithmembers class kotlinx.serialization.json.** {
    kotlinx.serialization.KSerializer serializer(...);
}

# Change here com.yourcompany.yourpackage
-keep,includedescriptorclasses class com.yourcompany.yourpackage.**$$serializer { *; } # <-- change package name to your app's
-keepclassmembers class com.yourcompany.yourpackage.** { # <-- change package name to your app's
    *** Companion;
}
-keepclasseswithmembers class com.yourcompany.yourpackage.** { # <-- change package name to your app's
    kotlinx.serialization.KSerializer serializer(...);
}
```

- ```# <-- change package name to your app's``` : 自分のアプリのパッケージ名に書き換えるのを忘れずに

---

## データクラス例

```kt:Hoge.kt
@Serializable
data class Hoge(
    @SerialName("hogegege")
    val str : String,

    val num : Int,

    @Serializable(with = BooleanAsBinarySerializer::class)
    val b : Boolean,

    @Serializable(with = ZonedDateTimeSerializer::class)
    val date : ZonedDateTime,

    @Serializable(with = RectSerializer::class)
    val rect : Rect,

    val nullableStr : String? = null
) {
    val message : String by lazy {
        str + num + b + date + (nullableStr ?: "null")
    }
}
```

- `@SerialName("")` : Jsonでのキー名が異なる場合に指定する  
  Gsonなどのように「すべてのキー名をJsonではスネークケースで扱う」とかは自動でできないっぽいので、現状いちいちこいつを設定する必要がありそう。

- `@Serializable(with = xxSerializer::class)` : シリアライザをプロパティ側で指定する

- `@Contextual` : Jsonインスタンス側でシリアライザを指定する  
    `@Contextual`と`@Serializable`をどちらも指定した場合、`@Serializable`のシリアライザ指定が優先される。

- プリミティブ値 : シリアライザを明示的に指定しない場合はアノテーションをつける必要はない

- デフォルト値 : そのままではデフォルト値の場合はシリアライズ時に出力されない  
    デフォルト値を出力するためには、`Json { encodeDefaults = true }`と指定した`Json`インスタンスを使用する

- lazyプロパティ : Gsonなどと違い、明示的に回避する必要はない

## ユーザー定義のシリアライザ例

### 非プリミティブなクラス用のシリアライザ

プリミティブ型の値(文字列、数値、真偽値など)やそれらだけをプロパティにもつデータクラスは`@Serializable`アノテーションを指定する以外には特別な処理を必要としない。  
一方、ユーザーが作成したクラスや、他のモジュールによって用意されたクラスの内容をjson文字列にシリアライズする際には`KSerializer<T>`を継承したシリアライザを用意する必要がある。

#### プリミティブな値と相互変換可能な場合

```kt:ZonedDateTimeSerializer.kt
/** ZonedDateTime用シリアライザ */
class ZonedDateTimeSerializer : KSerializer<ZonedDateTime> {
    override val descriptor: SerialDescriptor by lazy {
        PrimitiveSerialDescriptor(
            ZonedDateTimeSerializer::class.qualifiedName!!,
            PrimitiveKind.STRING
        )
    }

    private val formatter = DateTimeFormatter.ofPattern("uuuu-MM-dd'T'HH:mm:ssXXX")

    override fun serialize(encoder: Encoder, value: ZonedDateTime) {
        encoder.encodeString(value.format(formatter))
    }

    override fun deserialize(decoder: Decoder): ZonedDateTime {
        return ZonedDateTime.parse(decoder.decodeString(), formatter)
    }
}
```

`ZonedDateTime`の内容は`Long`型の数値と相互に可換である。このようにプリミティブ値で表現可能なデータクラスは`PrimitiveSerialDescriptor`を使用して変換できる。

`PrimitiveSerialDescriptor`の第一引数には一意な名前を指定する。シリアライザ自身の名前を渡しておけばまず大丈夫と思われる。  
第二引数の`PrimitiveKind`指定は、何型にエンコードして出力するかという指定だ。(ちなみに、仮に間違った指定をしてもとくにエラーなどにはならず動きはする)

#### 直接プリミティブ値に変換できない場合

例として、外部モジュールから提供される次のような`Rect`クラスを扱いたいとする。`Rect`クラスの4つのプロパティは何らかの1つのプリミティブ値にまとめて表現することはできないとする。

```kt:Rect.kt
data class Rect(
    val left : Int,
    val top : Int,
    val right : Int,
    val bottom : Int
) {
    init {
        require(left <= right)
        require(top <= bottom)
    }
}
```

この場合、使用するデスクリプタは`PrimitiveSerialDescriptor`ではなく、次例のように`buildClassSerialDescriptor {...}`を使用してデータ構造を扱うデスクリプタを作成する。

```kt:RectSerializer.kt
class RectSerializer : KSerializer<Rect> {
    private enum class Element(val selector: (Rect)->Int) {
        LEFT({ it.left }),
        TOP({ it.top }),
        RIGHT({ it.right }),
        BOTTOM({ it.bottom })
    }

    override val descriptor: SerialDescriptor
        get() = buildClassSerialDescriptor(RectSerializer::class.qualifiedName!!) {
            Element.values().forEach {
                element<Int>(it.name.toLowerCase())
            }
        }

    override fun serialize(encoder: Encoder, value: Rect) {
        encoder.encodeStructure(descriptor) {
            Element.values().forEachIndexed { index, elem ->
                encodeIntElement(descriptor, index, elem.selector(value))
            }
        }
    }

    override fun deserialize(decoder: Decoder): Rect {
        return decoder.decodeStructure(descriptor) {
            val elements = IntArray(Element.values().size)
            while (true) {
                when (val index = decodeElementIndex(descriptor)) {
                    CompositeDecoder.DECODE_DONE -> break
                    else -> {
                        if (0 <= index && index < elements.size) {
                            elements[index] = decodeIntElement(descriptor, index)
                        }
                        else error("Unexpected index: $index")
                    }
                }
            }
            Rect(elements[0], elements[1], elements[2], elements[3])
        }
    }
}
```

これによって`{"left":0,"top":1,"right":2,"bottom":3}`というような形で出力されるようになる。

ここまでやるなら、この方法用のベースクラスを用意して`Element`列挙体の部分だけ差し替えるように処理の共通化もできそうだが。

シンプルにべたっと書いた感じは公式リファレンスを参照。

[Hand-written composite serializer](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/serializers.md#hand-written-composite-serializers)

### プリミティブ型に別の様式のものを用意する

```kt:BooleanAsBinarySerializer.kt
/** 真偽値を0と1で表現するやつ */
class BooleanAsBinarySerializer : KSerializer<Boolean> {
    override val descriptor: SerialDescriptor by lazy {
        PrimitiveSerialDescriptor(
            BooleanBinarySerializer::class.qualifiedName!!,
            PrimitiveKind.INT
        )
    }

    override fun serialize(encoder: Encoder, value: Boolean) {
        encoder.encodeInt(if (value) 1 else 0)
    }

    override fun deserialize(decoder: Decoder): Boolean {
        return decoder.decodeInt() == 1
    }
}
```

真偽値などのプリミティブ型の値に対しては基本的にはシリアライザを用意する必要はない。ただし、なんらかの事情でjson側での表現方法を変更する必要がある場合にはこのように新しいシリアライザを用意して、`@Serializable`アノテーションで指定することができる。

---

### シリアライズ・デシリアライズしてみる

```kt:example.kt
fun main() {
    val hoge = Hoge(
        str = "hello world.",
        num = 1234,
        b = true,
        date = ZonedDateTime.now(),
        rect = Rect(0, 1, 2, 3)
    )

    val encoded = Json.encodeToString(hoge)
    println("1: $encoded")

    val decoded = Json.decodeFromString<Hoge>(encoded)
    println("2: ${decoded.message}")

    val list = listOf(hoge, hoge)
    val encodedList = Json.encodeToString(list)

    println("3: $encodedList")
}
```

実行結果の出力▼

```:出力
1: {"hogegege":"hello world.","num":1234,"b":1,"date":"2021-01-24T17:22:49+09:00","rect":{"left":0,"top":1,"right":2,"bottom":3}}
2: hello world.1234true2021-01-24T17:22:49+09:00null
3: [{"hogegege":"hello world.","num":1234,"b":1,"date":"2021-01-24T17:22:49+09:00","rect":{"left":0,"top":1,"right":2,"bottom":3}},{"hogegege":"hello world.","num":1234,"b":1,"date":"2021-01-24T17:22:49+09:00","rect":{"left":0,"top":1,"right":2,"bottom":3}}]
```

json文字列化するには`Json.encodeToString(value)`、json文字列からデータクラスを生成するには`Json.decodeFromString<T>(str)`を使う。  
シンプルな用途では基本的にこれだけでよい。

---

## ほか

公式リファレンスがすごい文量なので、いくつか使ったことがある部分だけ適当にピックアップしていく。

### データクラスに存在しないフィールドを無視する

[Ignoring unknown keys](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/json.md#ignoring-unknown-keys)

```kt
@Serializable
data class Project(val name: String)

fun main() {
    val formatter = Json { ignoreUnknownKeys = true }
    val data = formatter.decodeFromString<Project>("""
        {"name":"kotlinx.serialization","language":"Kotlin"}
    """)
    println(data)
}
```

`language`フィールドは`Project`クラスには存在しないが、json文字列には存在する。このような場合、`ignoredUnknownKeys = true`を指定した新しい`Json`インスタンスを生成したものを使用することでプログラム上では`language`フィールドを無視することができる。  
(指定しない場合`JsonDecodingException`が発生する)

### シリアライザを`Json`インスタンス側で設定する

プロパティに`@Contextual`を指定している場合、そのプロパティを変換するのに使用するシリアライザは`Json`インスタンス生成時に指定する必要がある。

```kt
@Serializable
data class Hoge(
    @Contextual
    val data : ZonedDateTime
)

fun main() {
    val formatter = Json {
        serializersModule = SerializersModule {
            contextual(ZonedDataTime::class, ZonedDataTimeSerializer())
        }
    }
}
```

あるクラスに対するシリアライザが常に一種類でかつ何度もその型のプロパティを記述する場合や、値が異なるフォーマットで記録されているがデータ内容自体は同じ複数のjsonを扱う場合(あるか？)などには面倒が減っていいかもしれない。  
基本的には`@Serializable(with = xxSerializer::class)`を毎回きっちり指定する方が混乱を招かなくて済むと思う。

### ポリモーフィズムへの対応

[Polymorphism](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/polymorphism.md)

#### 継承元を`sealed class`にする場合

`sealed class`を使用する場合、`Json`インスタンスに追加の指定を行うことなくポリモーフィックな値を扱うことができる。

```kt
@Serializable
sealed class Base(val id : Int)

@Serializable
class DerivedA(val msg: String) : Base(0)

@Serializable
class DerivedB(val msg: String) : Base(1)

@Serializable
class Exam(val data: Base)

fun main() {
    val exam = Exam(
        data = DerivedA("hage")
    )

    val str = Json.encodeToString(exam)
    println(str)

    val decoded = Json.decodeFromString<Exam>(str)
    println((decoded.data as DerivedA).msg)
}
```

#### モジュールに継承構造を教える場合

`sealed class`を使用したくない場合は、次のように継承構造を教えた`Json`インスタンスを使用することで取り扱いが可能になる。

```kt
@Serializable
abstract class Base(val id : Int)

@Serializable
class DerivedA(val msg: String) : Base(0)

@Serializable
class DerivedB(val msg: String) : Base(1)

@Serializable
class Exam(val data: Base)

fun main() {
    val formatter = Json {
        serializersModule = SerializersModule {
            polymorphic(Base::class) {
                subclass(DerivedA::class)
                subclass(DerivedB::class)
            }
        }
    }

    val exam = Exam(
        data = DerivedA("hage")
    )

    val str = formatter.encodeToString(exam)
    println(str)

    val decoded = formatter.decodeFromString<Exam>(str)
    println((decoded.data as DerivedA).msg)
}
```

実行結果の出力▼

```
{"data":{"type":"com.suihan74.sandbox.DerivedA","id":0,"msg":"hage"}}
hage
```

どちらの方法でも、実体が何型であるかを記録するための`type`フィールドが自動的に挿入されるので、対象のデータクラスに`type`という名前のプロパティを持たせているとエラーになる。`@SerialName("~~")`で保存時に改名するか違うプロパティ名を使用する必要がある。

また、この`type`に記録される内容(デフォルトではクラス名)は次のようにして`@SerialName("~~")`で明示的に指定することができる。

```kt
@Serializable
@SerialName("derived_a")
class DerivedA(val msg: String) : Base(0)
```

実行結果の出力▼

```
{"data":{"type":"derived_a","id":0,"msg":"hage"}}
```
