---
title: "ダイアログアクティビティの表示位置を変更する方法"
description: "ダイアログアクティビティにコード側でgravityを設定する方法"
tags: ["Android","Kotlin","activity"]
date: 2021-05-30T12:45:00+09:00
lastmod: 2021-05-30T12:45:00+09:00
archives:
    - 2021
    - 2021-05
    - 2021-05-30
hide_overview: false
draft: false
---

ダイアログ化した`Activity`を表示する際、とくに指定が無ければダイアログは画面中央に表示される（はず）。

{{<img src="default.png" zoom=".5" title="gravity指定なし">}}

いつの間にかすっかり巨大化してしまったスマホの画面であるが、片手操作では画面中央に表示されるダイアログのボタンに指が届かないなどの不満が（少なくとも自分には）あるので「ダイアログの表示位置を画面下部とかにできたらなァ」ということで今回やってみた。

```kt:HogeDialogActivity.kt
class HogeDialogActivity : AppComapActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setDialogVerticalGravity(Gravity.BOTTOM)
        ~~~ いろいろ ~~~
    }

    private fun setDialogVerticalGravity(gravity: Int) {
        window.attributes.gravity =
            (gravity and Gravity.VERTICAL_GRAVITY_MASK) or Gravity.CENTER_HORIZONTAL
    }
}
```

{{<img src="bottom.png" zoom=".5" title="Gravity.BOTTOM">}}

{{<img src="top.png" zoom=".5" title="Gravity.TOP">}}

`Gravity.CENTER_HORIZONTAL`の部分を変えれば横位置も変えられるのかもしれないが、今回は試していない。
