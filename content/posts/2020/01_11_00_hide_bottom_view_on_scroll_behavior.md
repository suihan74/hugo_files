---
title: "HideBottomViewOnScrollBehaviorの状態を外部から変更する"
description: タブを切り替えたときに強制的に再表示する場合などに使用。
tags: ["android", "kotlin", "view"]
date: 2020-01-11T04:30:41+09:00
lastmod: 2020-01-11T04:30:41+09:00
archives:
    - 2020
    - 2020-01
    - 2020-01-11
draft: false
---

### 関連記事
[スクロールで画面上下端のツールバーやボタンを表示/非表示](/posts/2020/01_02_02_show_views_with_scrolling)

# 前提

- `HideBottomViewOnScrollBehavior`を使用することでコンテンツのスクロールにあわせて画面下部の表示物を表示したり隠したりできる。
- protectedメソッド`slideDown(child)`が呼ばれることで画面下端より下にビューを移動して隠す。`slideUp(child)`で元の位置に戻して再表示する。

# 問題

たとえばコンテンツ部分にタブを表示していて、そのコンテンツ部分には`RecyclerView`が配置されているとする。  
移動前のタブでスクロールによって画面下部のビュー(以下`BottomView`)を隠したあとでタブを移動する。  
もし移動先のタブにスクロールできるものが無かった場合、そのタブを前面に表示している限りは`BottomView`を再表示することができなくなってしまう。

# 解決方法

タブが変更されたら`BottomView`を強制的に再表示するようにする。  
そのためには、`HideBottomViewOnScrollBehavior`をオーバーライドして`slideDown(...)`、`slideUp(...)`を外から呼べるようにする必要がある。

## 様子

![様子アニメーション](/images/2020/01_11_00_00.gif "画面下部FAB部分がタブの切り替えと同時に再表示される。")

画面下部FAB部分がタブの切り替えと同時に再表示される。  
このFAB部分には以下の`ExtendedHideBottomViewOnScrollBehavior`(相当のもの)を`layoutParams.behavior`に設定している。

## ExtendedHideBottomViewOnScrollBehavior.kt
```kt
package com.suihan74.utilities

import android.content.Context
import android.util.AttributeSet
import android.view.View
import com.google.android.material.behavior.HideBottomViewOnScrollBehavior

class ExtendedHideBottomViewOnScrollBehavior<V : View>(
    context: Context?,
    attrs: AttributeSet?
) : HideBottomViewOnScrollBehavior<V>(context, attrs) {

    public override fun slideDown(child: V) {
        super.slideDown(child)
    }

    public override fun slideUp(child: V) {
        super.slideUp(child)
    }
}
```

## HogeActivity.kt
```kt
...
tab_layout.apply {
    setupWithViewPager(viewPager)
    addOnTabSelectedListener(object : TabLayout.OnTabSelectedListener {
        override fun onTabSelected(tab: TabLayout.Tab?) {
            // タブを切り替えたらBottomViewを再表示する
            showButtons()
        }
        override fun onTabUnselected(p0: TabLayout.Tab?) { ... }
        override fun onTabReselected(tab: TabLayout.Tab?) { ... }
    })
}
...

private fun showButtons() {
        val behavior = (bottom_view.layoutParams as? CoordinatorLayout.LayoutParams)
        ?.behavior as? ExtendedHideBottomViewOnScrollBehavior
        ?: return
    behavior.slideUp(buttons_layout)
}

```

---

参考  
[android - How to show/hide AppBottomBar programmatically? - Stack Overflow](https://stackoverflow.com/questions/51456312/how-to-show-hide-appbottombar-programmatically)
