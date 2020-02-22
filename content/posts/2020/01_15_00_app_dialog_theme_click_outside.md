---
title: "ダイアログ化したActivityの外側タップを検知する"
description: "AppDialogThemeが設定されたActivityの外側をタップするとActivityが閉じるが、その前に何らかの処理を追加する方法"
tags: ["android", "kotlin", "activity"]
date: 2020-01-15T01:30:40+09:00
lastmod: 2020-01-17T01:30:00+09:00
archives:
    - 2020
    - 2020-01
    - 2020-01-15
draft: false
---

## 追記 (2020-01-17 01:30)

記述例の`DialogActivity`を表示するために`android:theme`に設定する値がプロジェクト内で自分で用意したスタイルになっていたので修正。

修正前: `@style/AppDialogTheme`  
↓  
修正後: `@style/Theme.AppCompat.Dialog`

---

# Activityをダイアログ表示する

`android:theme`に`@style/Theme.AppCompat.Dialog`を設定した`Activity`はダイアログとして表示される。(以下`DialogActivity`と呼称することにする)

次のは[Satena](https://play.google.com/store/apps/details?id=com.suihan74.satena)のブクマ投稿画面を抜粋。

## [AndroidManifest.xml](https://github.com/suihan74/Satena/blob/master/app/src/main/AndroidManifest.xml)
```xml {linenos=table, linenostart=62}
<activity
        android:name=".scenes.post2.BookmarkPostActivity"
        android:theme="@style/Theme.AppCompat.Dialog">
    ...
</activity>
```

## 様子

![様子](/images/2020/01_15_00_00.png "Activityは最前面にダイアログ表示されている。")

`Activity`は最前面にダイアログ表示されている。


# ダイアログ外側をタップで閉じる

`DialogActivity`の外側は暗くなっていて、この部分をタップすると`DialogActivity`が閉じて、その下に表示されている画面に処理が戻る。

`DialogActivity`が何らかの結果を呼び出し元に返すことを期待している場合などに、この「外側タップしてからダイアログが閉じるまでの間」に処理を追加したい、というのが今回の内容である。

### 関連記事
[結果を返すActivityを呼ぶ](/posts/2020/01_07_00_start_activity_for_result/)


# 閉じる前に処理を追加する

`onTouchEvent(...)`はダイアログ外側のタップにも反応する。  
タップされた位置から`DialogActivity`の外側であることを判別するようにするのが楽そうな解決策だった。

## DialogActivity.kt

```kt
override fun onTouchEvent(event: MotionEvent?): Boolean {
    // Activityの外側をタップして閉じる際に、結果を渡しておく
    if (event?.action == MotionEvent.ACTION_DOWN && isOutOfBounds(event)) {
        val intent = Intent().apply {
            putExtra(RESULT_HOGE, viewModel.hoge.value)
        }
        setResult(RESULT_CANCELED, intent)
    }
    return super.onTouchEvent(event)
}

/** Activityの外側をタップしたかを判別する */
private fun isOutOfBounds(event: MotionEvent): Boolean {
    val x = event.x.toInt()
    val y = event.y.toInt()
    val dialogBounds = Rect()
    window.decorView.getHitRect(dialogBounds)
    return !dialogBounds.contains(x, y)
}
```

ちなみに`MotionEvent.ACTION_UP`だと先にダイアログが閉じてしまって駄目だった。

### 参考
[android - How to dismiss the dialog with click on outside of the dialog? - Stack Overflow](https://stackoverflow.com/a/24435357)

---

# 余談1

`DialogActivity`の外側タップを検知する方法を調べていると、よくヒットしたのは別の方法だった。(上記の参考先StackOverflowのページでも以下の方法とそれで発生する問題について言及されている)

1. `onCreate()`などで次のようなフラグをセットする。
{{<highlight kotlin>}}
window.addFlags(WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL)
window.addFlags(WindowManager.LayoutParams.FLAG_WATCH_OUTSIDE_TOUCH)
{{</highlight>}}

2. `onTouchEvent()`で`MotionEvent.ACTION_OUTSIDE`を検知する。
{{<highlight kotlin>}}
override fun onTouchEvent(event: MotionEvent?): Boolean {
    if (event?.action == MotionEvent.ACTION_OUTSIDE) {
        ...
    }
    return super.onTouchEvent(event)
}
{{</highlight>}}

この方法を用いると確かに外側タップは検出できるのだが、どういうわけかタップが下にある画面にも伝播してしまう問題があった。  
フラグの設定方法や組み合わせやら`onTouchEvent()`の戻り値やらを色々変えて試してみたが、  
`MotionEvent.ACTION_OUTSIDE`を検出する方法をとった上でタップが伝播する問題を回避する方法はわからなかった。

詳しく調べまわったりはそれほどしていないので実際のところ回避方法があるかはわからないが、無かったところで今回記事の方法で要件は満足しているのでとくに問題もないような気がする。


# 余談2

外側の暗いところをタップしてもダイアログを閉じないようにするには`onCreate(...)`などで次のメソッドを実行すれば良い。

```kt
setFinishOnTouchOutside(false)
```
