---
title: "タッチイベントの伝播を止める方法"
description: "スクロールできるビューの上にさらにスクロールできるものを追加する場合などに、一番上のビューだけがタッチイベントを処理するようにする方法"
tags: ["Android", "kotlin", "onTouchListener"]
date: 2020-09-26T23:03:54+09:00
lastmod: 2020-09-26T23:03:54+09:00
archives:
    - 2020
    - 2020-09
    - 2020-09-26
hide_overview: false
draft: false
---

## 仮定する状況

たとえば、ドロワーの上に横方向の```RecyclerView```を配置した場合、```RecyclerView```の項目をスクロールしようとするとドロワーが閉じてしまう。  
また他にも、```ViewPager```でタブフラグメントを表示して、その上に同じように横方向にスクロールできるビューを配置したい場合などでも同様。

## 対応策

そのような場合に、操作対象のビューのタッチイベントだけを処理してドロワーやらタブやらの切り替えを防ぐには、次のようにしてタッチイベントの伝播を防ぐことができる。

```kt
targetView.setOnTouchListener { view, ev ->
    // 伝播を防ぐ
    parent.requestDisallowInterceptTouchEvent(true)

    // ビューに対する処理
    // ~~~
    false
}
```

```RecyclerView```とかなら、項目のタッチイベントで同じようにすればよい。
