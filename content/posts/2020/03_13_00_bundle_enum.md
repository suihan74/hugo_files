---
title: "BundleでEnumを扱うやつ"
description: "コストとかはとくに考えないでBundle.putEnum()とかやってみるやつ。"
tags: ["android", "kotlin"]
date: 2020-03-13T18:50:56+09:00
lastmod: 2020-03-13T18:50:56+09:00
archives:
    - 2020
    - 2020-03
    - 2020-03-13
hide_overview: false
draft: false
---

## やりたいこと

Activity/Fragmentの`arguments`やら`savedInstanceState`やらは`Bundle`クラスを使ってデータをやりとりする。

列挙型の値を渡したいときは次のようにする(のが多分一番簡単)。

```kt
// 入れるとき
arguments.putInt(ARG_KEY, hogeEnum.ordinal)

// 出すとき
val hogeEnum = HogeEnum.values().get(arguments.getInt(ARG_KEY))
```

そんなわけで、他の値の出し入れと同じように次のように書きたくなる(なった)。

```kt
// 入れるとき
arguments.putEnum(ARG_KEY, hogeEnum)

// 出すとき
val hogeEnum = arguments.getEnum<HogeEnum>(ARG_KEY, defaultHoge)
```

## 書いた

### Bundle.kt

```kt
package com.suihan74.utilities

import android.os.Bundle
import kotlin.reflect.KClass

/** Enum<T>::classから直接valuesを取得する */
@Suppress("UNCHECKED_CAST")
inline fun <reified T : Enum<T>> KClass<T>.getEnumConstants() =
    Class.forName(T::class.qualifiedName!!).enumConstants as Array<out T>

// --------- //

/** BundleにEnumをセットする */
inline fun <reified T : Enum<T>> Bundle.putEnum(key: String, value: T) {
    putInt(key, value.ordinal)
}

/** BundleからEnumを取得する(失敗時null) */
inline fun <reified T : Enum<T>> Bundle.getEnum(key: String) : T? =
    try {
        (get(key) as? Int)?.let { ordinal ->
            T::class.getEnumConstants().getOrNull(ordinal)
        }
    }
    catch (e: Throwable) {
        null
    }

/** BundleからEnumを取得する(失敗時デフォルト値) */
inline fun <reified T : Enum<T>> Bundle.getEnum(key: String, defaultValue: T) : T =
    getEnum<T>(key) ?: defaultValue

// --------- //

/** BundleにEnumをセットする(ordinal以外を使用) */
inline fun <reified T : Enum<T>> Bundle.putEnum(key: String, value: T, selector: (T)->Int) {
    putInt(key, selector(value))
}

/** BundleからEnumを取得する(ordinal以外を使用, 失敗時null) */
inline fun <reified T : Enum<T>> Bundle.selectEnum(key: String, selector: (T)->Int) : T? =
    try {
        (get(key) as? Int)?.let { intValue ->
            T::class.getEnumConstants().firstOrNull { selector(it) == intValue }
        }
    }
    catch (e: Throwable) {
        null
    }

/** BundleからEnumを取得する(ordinal以外を使用, 失敗時デフォルト値) */
inline fun <reified T : Enum<T>> Bundle.selectEnum(key: String, defaultValue: T, selector: (T)->Int) : T =
    selectEnum(key, selector) ?: defaultValue
```

## 使い方

二種類の方法を用意してみた。

### ordinalを使用する

通常とくにこれで問題ない。  
ただしbundleに保存される値はenum値の宣言された順序に依存するので、外部に値を保存する場合などはその点には注意が必要かもしれない。

```kt
// 入れるとき
arguments.putEnum(ARG_KEY, hogeEnum)

// 出すとき(見つからなかったらnull)
val hogeEnum = arguments.getEnum<HogeEnum>(ARG_KEY)!!

// 出すとき(見つからなかったらデフォルト値)
val hogeNum = arguments.getEnum<HogeEnum>(ARG_KEY, defaultHoge)
```

### 任意のプロパティを使用する

Enumには任意の値を持たせられる。

```kt

enum HogeEnum(
    val prop : Int
) {
    A(100),
    B(200),
    C(300)
}
```

これを使用して識別する方法を用意した。  
使用できるのは今回Int値だけにした。面倒くさいから。

```kt
// 入れるとき
arguments.putEnum(ARG_KEY, hogeEnum) { it.prop }

// 出すとき(見つからなかったらnull)
val hogeEnum = arguments.selectEnum<HogeEnum>(ARG_KEY) { it.prop }!!

// 出すとき(見つからなかったらデフォルト値)
val hogeNum = arguments.selectEnum<HogeEnum>(ARG_KEY, defaultHoge) { it.prop }
```

## 結論

やったぜ。
