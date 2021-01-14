---
title: "文字列リソースファイル内で文字列を連結する方法"
description: "リソースファイルで内部エンティティが使える話"
tags: ["Android", "xml", "resources"]
date: 2021-01-14T23:06:05+09:00
lastmod: 2021-01-14T23:06:05+09:00
archives:
    - 2021
    - 2021-01
    - 2021-01-14
hide_overview: false
draft: false
---

## 前提

次のようにアプリ名を`@string/app_name`として文字列リソースに用意しているとする。

```xml:values/strings.xml
<resources>
    <string name="app_name">HogeApp</string>
</resources>
```

別の文字列リソースの部分文字列として`@string/app_name`を含めたい場合、  
通常であれば、たとえば次のようにしてコード側やデータバインドで値を挿入した結果の文字列を生成する必要がある。

```xml:res/values/strings.xml
<resources>
    <string name="app_name">HogeApp</string>
    <string name="author">suihan</string>
    <!-- 「アプリ名: HogeApp」と表示したい(雑な例だが) -->
    <string name="hoge">アプリ名: %s, 作者: %s</string>
</resources>
```

```kt:コードで文字列を作成する例.kt
val str = String.format(
    context.getString(R.string.hoge),
    context.getString(R.string.app_name),
    context.getString(R.string.author)
)
```

```xml:データバインディングで文字列を挿入する例.xml
<TextView
    android:text="@{@string/hoge(@string/app_name,@string/author)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
/>
```

今回はそうではなく、`string.xml`内で複数の文字列リソースで部分文字列を共有する方法について記述する。

## 内部エンティティ宣言を使用する方法

次のようにDTD内に記述したエンティティは`&名前;`の形で定数的に各文字列リソースに挿入することができる。

これはAndroidアプリ開発に限った話ではなく、xmlのエンティティ宣言という要素らしい。  
Android Developersのリファレンスにはこの方法に関する記述は見つからなかったのだが、これは単にリソースファイルがxmlだから当たり前すぎて書いていないのか、推奨されない何らかの理由があるのかその辺はよくわからない。

```xml:res/values/strings.xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE resources [
    <!ENTITY appName "HogeApp">
    <!ENTITY author "suihan">
]>

<resources>
    <string name="app_name">&appName;</string>
    <string name="author">&author;</string>
    <!-- 「アプリ名: HogeApp, 作者: suihan」と表示したい(雑な例だが) -->
    <string name="hoge">アプリ名: &appName;, 作者: &author;</string>
</resources>
```

ちなみにエンティティ名入力時にはコード補完が普通に効く。

参考

[Reference one string from another string in strings.xml? - Stack Overflow](https://stackoverflow.com/questions/4746058/reference-one-string-from-another-string-in-strings-xml)

## 注意点

- この方法を使用する場合、AndroidStudioで`Ctrl+Numpad-`などでリソース名を折り畳むとエンティティの挿入部分は空文字列になってしまう。

- エンティティ宣言を外部ファイル化する方法については(そこまでする必要を感じなかったので)試していないからここには書いていないので参考先を見てほしいのだが、どうも問題がある(あった)らしいので現在のAndroid Studioで実際できるかどうかはわからない。

---

こういうGradleプラグインを作ってる人もいるらしい。

[LikeTheSalad / android-string-reference](https://github.com/LikeTheSalad/Android-string-reference)
