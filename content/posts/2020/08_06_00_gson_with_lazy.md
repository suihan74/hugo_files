---
title: "lazyプロパティを持ったデータクラスをJson化する"
description: "Gsonなどでlazyプロパティ持ったデータクラスを扱う方法"
tags: ["kotlin", "gson", "lazy"]
date: 2020-08-06T14:50:03+09:00
lastmod: 2020-08-06T18:00:00+09:00
archives:
    - 2020
    - 2020-08
    - 2020-08-06
hide_overview: false
draft: false
---

{{<tweet 1290979210228428803>}}

たまに忘れてやってしまうやつ。

## 追記 (2020-08-06)

### -> [Androidアプリ開発で発生したさらなる問題](#androidアプリ開発で発生したさらなる問題)

---

## 誤り

```kt
data class Hoge(
    val str : String,
    val num : Int
) {
    val message : String by lazy {
        str + num
    }
}
```

```kt
fun incorrectExample() {
    val hoge = Hoge("hoge", 1234)
    val gson = Gson()

    val json = gson.toJson(hoge)

    val deserialized = gson.fromJson<Hoge>(json, Hoge::class.java)

    System.out.println(desrialized.message)

    // ==> ヌルポ
}
```

non-null型だろうがお構いなしにヌルポ。

## 正しい書き方

```kt
data class Hoge(
    val str : String,
    val num : Int
) {
    constructor() : this("", 0)

    @delegate:Transient
    val message : String by lazy {
        str + num
    }
}
```

```kt
fun correctExample() {
    val hoge = Hoge("hoge", 1234)
    val gson = Gson()

    val json = gson.toJson(hoge)

    val deserialized = gson.fromJson<Hoge>(json, Hoge::class.java)

    System.out.println(desrialized.message)

    // ==> "hoge1234"
}
```

やったぜ。

## 原因

Gson(とかその類のもの)はデシリアライズの際にまず無引数のコンストラクタを探し、存在するならそれを使用する。存在しない場合はUnSafeなあの手この手の悪事を働いてとにかく指定されたクラスのインスタンスを無理矢理作る。  
そして作ったインスタンスに対してフィールドにJsonから変換した値を突っ込んでいく。

そういうわけで、lazyなプロパティを使用していようがいまいがgson.fromJson()から返された段階で既に中身が初期化されている扱いになってしまうため、遅延初期化が実行されずnullになってしまう。

なので、要点は以下の二点。

- 無引数のコンストラクタを用意する

- @delegate:Transientアノテーション付けてlazyプロパティのシリアライズを回避する

## 余談

無引数コンストラクタをデータクラスの外部に晒したくなかったら、privateでも大丈夫っぽい。

---

## Androidアプリ開発で発生したさらなる問題

この記事の方法を適用したデータクラスをさらに```Serializable```にして、  
```Bundle#putSerializable()```,```Bundle#getSerializable()```を使用してシリアライズ・デシリアライズしようとすると依然としてlazyプロパティが```null```になる模様。

なので、予めオブジェクトをJson化して```Bundle#putString(key,value)```で突っ込む・```Bundle#putString(key)```で取り出す方法を取るのが余計な頭使わなくて楽そう。  
ひとまずこんな感じで拡張関数書いた。

```kt
/** Bundle#putObject(), Bundle#getObject()で使用するGsonインスタンス */
val Bundle.gson : Gson by lazy {
    GsonBuilder()
        .serializeNulls()
        .setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES)
        .registerTypeAdapter(Boolean::class.java, BooleanDeserializer())
        .create()
}

/** シリアライズしたオブジェクトをBundleに渡す */
fun Bundle.putObject(key: String, value: Any?) {
    putString(key, this.gson.toJson(value))
}

/** keyに対応する文字列をT型にデシリアライズして返す */
inline fun <reified T> Bundle.getObject(key: String) : T? {
    val json = getString(key) ?: return null
    return try {
        this.gson.fromJson<T>(json, object : TypeToken<T>() {}.type)
    }
    catch (e: Throwable) {
        null
    }
}

// ------ //

/** Intentのデータ授受用 */
fun Intent.putObjectExtra(key: String, value: Any?) {
    this.putExtras((this.extras ?: Bundle()).apply {
        putObject(key, value)
    })
}

/** Intentのデータ授受用 */
inline fun <reified T> Intent.getObjectExtra(key: String) : T? {
    return this.extras?.getObject(key)
}
```

Gson使っているけど、kotlinx.serializationでもなんでも。
