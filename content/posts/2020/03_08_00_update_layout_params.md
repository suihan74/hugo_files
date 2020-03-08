---
title: "updateLayoutParams<T>使おうの巻"
description: "layoutParamsの更新方法"
tags: ["android", "kotlin"]
date: 2020-03-08T19:16:29+09:00
lastmod: 2020-03-08T19:16:29+09:00
archives:
    - 2020
    - 2020-03
    - 2020-03-08
draft: false
---

## 概要

Viewの`layoutParams`の更新は再代入が必要なので、とくに何も考えずにこういう風に書いていた。

```kt
// ツールバーをコンテンツのスクロールに合わせて隠したりする設定をコード側でする例
toolbar.layoutParams = (toolbar.layoutParams as? AppBarLayout.LayoutParams)?.apply {
    scrollFlags = AppBarLayout.LayoutParams.SCROLL_FLAG_SCROLL or AppBarLayout.LayoutParams.SCROLL_FLAG_ENTER_ALWAYS
}
```

## updateLayoutParams

ちゃんと更新用の拡張関数が用意されているのでこれを使うようにする。

-> [List of KTX extensions | Android Developers](https://developer.android.com/kotlin/ktx/extensions-list#for_androidviewview)  
androidxに含まれるものっぽい。

```kt
// ツールバーをコンテンツのスクロールに合わせて隠したりする設定をコード側でする例
toolbar.updateLayoutParams<AppBarLayout.LayoutParams> {
    scrollFlags = AppBarLayout.LayoutParams.SCROLL_FLAG_SCROLL or AppBarLayout.LayoutParams.SCROLL_FLAG_ENTER_ALWAYS
}
```

どちらを使うにしてもどうということはないけれど、少し簡潔になって嬉しい。

## 実装

ちなみにこの拡張関数の中身はこんなのなので、どちらにしろやっていることは本当に変わらない。

```kt
/**
 * Executes [block] with a typed version of the View's layoutParams and reassigns the
 * layoutParams with the updated version.
 *
 * @see View.getLayoutParams
 * @see View.setLayoutParams
 **/
@JvmName("updateLayoutParamsTyped")
inline fun <reified T : ViewGroup.LayoutParams> View.updateLayoutParams(block: T.() -> Unit) {
    val params = layoutParams as T
    block(params)
    layoutParams = params
}
```
