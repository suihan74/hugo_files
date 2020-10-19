---
title: "BindingAdapterが重複作成される警告"
description: "BindingAdapterはCompanionオブジェクトに書いてはいけない"
tags: ["Android","kotlin","DataBinding"]
date: 2020-10-19T14:48:13+09:00
lastmod: 2020-10-19T14:48:13+09:00
archives:
    - 2020
    - 2020-10
    - 2020-10-19
hide_overview: false
draft: false
---

## 参考

今回の内容は全部ここに書いてあることです。

[Defining Android Binding Adapter in Kotlin | by Herman Cheung | Medium](https://medium.com/@thinkpanda_75045/defining-android-binding-adapter-in-kotlin-b08e82116704)

---

## 問題

ビルドログを眺めているとこんな警告が大量に発生していた。

```
w: warning: Binding adapter AK(androidx.recyclerview.widget.RecyclerView, com.suihan74.example.Hoge) already exists for bindItems! Overriding com.suihan74.example.Hoge.Companion#bindItems with com.suihan74.example.Hoge#bindItems
```

ちなみに警告は出るがビルドは成功する。

## 原因

原因は、```companion object```の中に```BindingAdapter```を記述していたこと。  
kotlinの```companion object```は静的なフィールドやメソッドを定義するために有用だが、Javaコードに展開されると次のような状態になる。

### kotlin

```kt
package com.suihan74.example

class Hoge {
    companion object {
        @JvmStatic
        @BindingAdapter("items")
        fun bindItems(view: RecyclerView, items: List<Item>?) {
            // 省略
        }
    }
}
```

### Java

```java
package com.suihan74.example;

public final class Hoge {
    public static final class Companion {
        static void bindItems(RecyclerView view, List<Item> items) {
            // 省略
        }
    }
    public static bindItems(RecyclerView view, List<item> items) {
        Companion.bindItems(view, items);
    }
}
```

```Hoge.bindItems()```でも```hoge.Companion.bindItems()```でもアクセスできるようにふたつの静的メソッドが作成されるようだ。  
最初の警告文通り、これでは同じパラメータのための```BindingAdapter```が重複してしまう。

## 解決法

```companion object```を使わないで普通に書く。奇を衒わない。素直に公式で紹介されてるものを踏襲する。

### 拡張関数として用意する

```kt
@BindingAdapter("items")
fun RecyclerView.setItems(items: List<Item>?) {
    // 省略
}
```

簡単でいいと思う。  
やたら拡張関数生やすのはちょっとと思ったら次の方法。

### シングルトンオブジェクトに書く

```kt
object RecyclerViewBindingAdapters {
    @JvmStatic
    @BindingAdapter("items")
    fun setItems(view: RecyclerView, items: List<Item>?) {
        // 省略
    }
}
```

内部クラスとして適当なクラス内に書いてもいいので、たとえば```RecyclerView```なら各モデル用の```ItemsAdapter```を必ず書いているはずなので、そこに書くとかするとまとまりがあってよろしい感じがする。
