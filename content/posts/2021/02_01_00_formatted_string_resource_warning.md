---
title: "書式引数が複数ある文字列リソースでの順序指定"
description: "書式引数に添字をつけないとビルド時に警告が発生するやつ"
tags: ["Android","xml","resources"]
date: 2021-02-01T21:22:42+09:00
lastmod: 2021-02-01T21:22:42+09:00
archives:
    - 2021
    - 2021-02
    - 2021-02-01
hide_overview: false
draft: false
---

文字列リソースには書式文字列を使用することができる。

```xml:values/strings.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hoge">hello! name: %s</string>
</resources>
```

```kt:書式文字列挿入例.kt
val str = context.getString(R.string.hoge, "suihan")
```

▼実行結果  
`hello! name: suihan`

このもっとも簡単な例のように書式引数が一つだけのときはこれで問題ないが、書式引数が複数ある場合に次のようにすると、ビルド時に「`Multiple substitutions specified in non-positional format; did you mean to add the formatted="false" attribute?`」というような警告文が表示される。

```xml:values/strings.xml
<string name="hoge">hello! firstName: %s, lastName: %s</string>
```

```kt:書式文字列挿入例.kt
val str = context.getString(R.string.hoge, "taro", "tanaka")
```

▼実行結果  
`hello! firstName: taro, lastName: tanaka`

これは要するに`Context#getString`の各引数がそれぞれどの書式引数に割り当てられるかということをハッキリさせろということであり、指定しなければ警告とともにとりあえず左から順番に割り当てられていく。

そうしたとき、名前のように「姓→名」で表示する場合と「名→姓」で表示する場合が言語によって異なるような文字列において問題が発生するので、次のように文字列リソース側で割り当てる引数の添え字（1始まり）を指定する。

```xml:values-en/strings.xml
<string name="hoge">hello! firstName: %1$s, lastName: %2$s</string>
```

```xml:values-ja/strings.xml
<string name="hoge">こんにちは! 姓: %2$s, 名: %1$s</string>
```

```kt:書式文字列挿入例.kt
val str = context.getString(R.string.hoge, "taro", "tanaka")
```

▼実行結果  
英語の場合 -> `hello! firstName: taro, lastName: tanaka`  
日本語の場合 -> `こんにちは! 姓: tanaka, 名: taro`
