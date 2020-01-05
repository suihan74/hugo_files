---
title: "タイトルテキストがMarqueeするToolbarを作る"
description: 2020年にもなって文字列をmarqueeさせたくなったので作る
tags: ["android", "kotlin", "view"]
date: 2020-01-06T00:18:01+09:00
lastmod: 2020-01-06T00:18:01+09:00
draft: false
---

# やりたいこと

こういうの。

![やりたいやつ](/images/2020/01_06_00_01.gif "様子アニメーション")

ツールバーのtitle部分の文字列が横に流れ続けている。


# カスタムビューの作成

`Toolbar`を継承した次のようなカスタムビューを作成する。

## MarqueeToolbar.kt

```kt
package com.suihan74.utilities

import android.content.Context
import android.text.TextUtils
import android.util.AttributeSet
import android.widget.TextView
import androidx.appcompat.widget.Toolbar

class MarqueeToolbar : Toolbar {
    constructor(context: Context, attributeSet: AttributeSet? = null) :
            super(context, attributeSet)

    constructor(context: Context, attributeSet: AttributeSet?, defStyleAttr: Int) :
            super(context, attributeSet, defStyleAttr)

    /** タイトルの設定が完了しているか否か */
    private var reflected: Boolean = false
    /** タイトル部分のTextView */
    private var titleTextView: TextView? = null

    override fun setTitle(resId: Int) {
        if (!reflected) {
            reflected = reflectTitle()
        }
        super.setTitle(resId)
        selectTitle()
    }

    override fun setTitle(title: CharSequence?) {
        if (!reflected) {
            reflected = reflectTitle()
        }
        super.setTitle(title)
        selectTitle()
    }

    /** Viewが生成されたときに呼ばれる */
    override fun onWindowFocusChanged(hasWindowFocus: Boolean) {
        super.onWindowFocusChanged(hasWindowFocus)
        if (hasWindowFocus && !reflected) {
            reflected = reflectTitle()
            selectTitle()
        }
    }

    /** タイトル部分のTextViewを設定 */
    private fun reflectTitle() =
        try {
            val field = Toolbar::class.java.getDeclaredField("mTitleTextView").apply {
                isAccessible = true
            }
            titleTextView = (field.get(this) as? TextView)?.apply {
                ellipsize = TextUtils.TruncateAt.MARQUEE
                marqueeRepeatLimit = -1   // forever
            }

            titleTextView != null
        }
        catch (e: Throwable) {
            e.printStackTrace()
            false
        }

    /** タイトル部分を選択 */
    private fun selectTitle() {
        titleTextView?.isSelected = true
    }
}
```

参考:  
[A Marquee-able Android Toolbar. | InsanityOnABun / MarqueeToolbar.java | GitHub Gist](https://gist.github.com/InsanityOnABun/95c0757f2f527cc50e39)

# 説明

要するに、`Toolbar`のprivateフィールドであるタイトル部分の`TextView`をリフレクションで強引にぶっこ抜いてきてmarquee用の設定を追加しているというわけ。（`reflectTitle()`部分）

なお、`onWindowFocusChanged()`をオーバーライドしているのは何らかの方法[^1]でViewが生成完了して画面に表示されるまでの間に`reflectTitle()`が実行されないと`setTitle()`が呼ばれるまでアニメーションが始まらないため。  
（`Activity.onCreate()`内で`toolbar.title = "長い文字列"`を記述しても`View`のライフサイクル的な問題で`reflectTitle()`内の`field.get(this)`に失敗する）

[^1]: 何らかの方法→[ViewのonStart/onPause - Qiita](https://qiita.com/wasnot/items/06375957a325ba3bc2fa)

# 使用方法

## activity_hoge.xml

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbar_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

        <com.suihan74.utilities.MarqueeToolbar
                android:id="@+id/toolbar"
                app:layout_scrollFlags="enterAlways|scroll"
                android:layout_width="match_parent"
                android:layout_height="match_parent"/>

    </com.google.android.material.appbar.AppBarLayout>

    ...

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

# 余談

「サブタイトルもmarqueeさせたいぜ」という欲張りさんは同じようにして`MarqueeToolbar`で`mSubtitleTextView`をぶっこ抜いて設定すればいいんだと思う。
