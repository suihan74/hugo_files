---
title: "enum値に設定されたアノテーションを取得する方法"
description: ""
tags: ["Kotlin","annotation","enum"]
date: 2021-03-27T22:44:28+09:00
lastmod: 2021-03-27T22:44:28+09:00
archives:
    - 2021
    - 2021-03
    - 2021-03-27
hide_overview: false
draft: false
---

ある列挙型の値に設定されたアノテーションを取得する方法。  
個人的には滅多に使わないけど、必要な時に忘れるのでメモ。

```kt
val anno =
    HogeEnum.VALUE
    .declaringClass
    .getField(HogeEnum.VALUE.name)
    .getAnnotation(TestAnno::class.java)
```

```kt:例のアノテーション.kt
annotation class TestAnno
```

```kt:例のEnum.kt
enum class HogeEnum {
    @TestAnno
    VALUE
}
```
