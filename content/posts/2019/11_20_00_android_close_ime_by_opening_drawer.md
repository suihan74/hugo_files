---
title: "Androidアプリ開発Tips – Drawer展開時にIMEを閉じる"
description: "Drawer展開を検知したら入力メソッドを閉じるようにする"
tags: ["Android", "kotlin", "migrated_from_previous_blog"]
date: 2019-11-20T01:42:00+09:00
lastmod: 2020-02-01T03:10:11+09:00
archives:
    - 2019
    - 2019/11
    - 2019/11/20
draft: false
hide_overview: true
---

## おことわり

この記事は旧ブログから引っ張ってきたものの再掲です。

## 当時の内容

`EditText`とか`SearchView`とかにフォーカスが当たっていてIMEで文字入力中にドロワーを表示状態にしたとき、IMEは自動的に閉じませんよというお話。

<img src="/images/2019/11_20_00_from.gif" title="改修前" width="25%"/>

これを

<img src="/images/2019/11_20_00_to.gif" title="改修後" width="25%"/>

こうじゃ。

まずはIMEの非表示処理を毎回書くのは面倒くさいので、Activity辺りを拡張する。

```kt
/**
 * キーボードを隠して入力対象のビューをアンフォーカスする
 */
fun Activity.hideSoftInputMethod() = currentFocus?.let { focusedView ->
    focusedView.clearFocus()
    val im = getSystemService(Context.INPUT_METHOD_SERVICE) as? InputMethodManager
    im?.hideSoftInputFromWindow(
        focusedView.windowToken,
        InputMethodManager.HIDE_NOT_ALWAYS
    )
} ?: false
```

非表示処理が問題なく行われたらtrueが返ってくるようにした。何もフォーカスされてなかったりサービスの取得に失敗したりする（するのか？）とfalseが返る。

ちなみに、focusedView.clearFocus()を実行しないとIMEは隠れてもViewにフォーカスは当たりっぱなしなので、意図した戻るボタンの挙動を得る前に一回余計に戻るボタンを押す必要が生じてしまう。

あとはこの処理をドロワーが開いたときに呼んでやればよい。

```kt
// ActivityのonCreate()とかに書く
mDrawer = findViewById(R.id.drawer_layout)
val drawerToggle = object : ActionBarDrawerToggle(this, mDrawer, toolbar,
    R.string.drawer_open,
    R.string.drawer_close
) {
    override fun onDrawerOpened(drawerView: View) {
        super.onDrawerOpened(drawerView)
        hideSoftInputMethod()
    }
}
mDrawer.addDrawerListener(drawerToggle)
```

なお、この方法だと「 ドロワー が開き終わったとき」にIMEが閉じる。「開き始めたとき」に閉じたかったら`onDrawerStateChanged()`辺りをアレすればできそう（だが今のところウチではやってないので詳しくは知らん）
