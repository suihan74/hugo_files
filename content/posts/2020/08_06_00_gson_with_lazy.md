---
title: "lazyプロパティを持ったデータクラスをJson化する"
description: "Gsonなどでlazyプロパティ持ったデータクラスを扱う方法"
tags: ["kotlin", "gson", "lazy"]
date: 2020-08-06T14:50:03+09:00
lastmod: 2020-08-06T14:50:03+09:00
archives:
    - 2020
    - 2020-08
    - 2020-08-06
hide_overview: false
draft: false
---

{{<tweet 1290979210228428803>}}

たまに忘れてやってしまうやつ。

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
