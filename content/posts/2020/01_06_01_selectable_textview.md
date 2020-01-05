---
title: "LinkMovementMethodを設定したTextViewの文字列選択を解除するとクラッシュする"
description: LinkMovementMethodを設定したTextViewを文字列選択して解除するとクラッシュする問題をdispatchTouchEvent()を弄ることで回避する。
tags: ["android", "kotlin", "view"]
date: 2020-01-06T02:49:08+09:00
lastmod: 2020-01-06T02:49:08+09:00
draft: false
---

# 問題

以下の条件の`TextView`を用意する。

- `movementMethod`に`LinkMovementMethod`を設定する。
- `android:textIsSelectable="true"`を設定する。

そして以下の手順で操作を行うと、アプリがクラッシュする。

1. 該当`TextView`の文字列を選択する。
2. `TextView`上の適当な領域をタップして文字列選択を解除する。
3. 破滅する。

ちなみに、  
`LinkMovementMethod`の`return super.onTouchEvent(...)`の部分にブレークポイントを置いてから文字列選択解除すると発生しなくてなんやねんってなった。

また、解除時のタップは単押し長押しに関わらずとにかく`MotionEvent.ACTION_DOWN`がきた時点で駄目っぽかった。

## 様子

![様子](/images/2020/01_06_01_01.gif "様子アニメーション")

## ログ

```java
java.lang.IllegalArgumentException: Invalid offset: -1. Valid range is [0, 16]
    at android.text.method.WordIterator.checkOffsetIsValid(WordIterator.java:380)
    at android.text.method.WordIterator.isBoundary(WordIterator.java:101)
    at android.widget.Editor$SelectionStartHandleView.positionAtCursorOffset(Editor.java:4260)
    at android.widget.Editor$HandleView.updatePosition(Editor.java:3708)
    at android.widget.Editor$PositionListener.onPreDraw(Editor.java:2507)
    at android.view.ViewTreeObserver.dispatchOnPreDraw(ViewTreeObserver.java:944)
    at android.view.ViewRootImpl.performTraversals(ViewRootImpl.java:2055)
    at android.view.ViewRootImpl.doTraversal(ViewRootImpl.java:1107)
    at android.view.ViewRootImpl$TraversalRunnable.run(ViewRootImpl.java:6013)
    at android.view.Choreographer$CallbackRecord.run(Choreographer.java:858)
    at android.view.Choreographer.doCallbacks(Choreographer.java:670)
    at android.view.Choreographer.doFrame(Choreographer.java:606)
    at android.view.Choreographer$FrameDisplayEventReceiver.run(Choreographer.java:844)
    at android.os.Handler.handleCallback(Handler.java:739)
    at android.os.Handler.dispatchMessage(Handler.java:95)
    at android.os.Looper.loop(Looper.java:148)
    at android.app.ActivityThread.main(ActivityThread.java:5417)
    at java.lang.reflect.Method.invoke(Native Method)
    at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:726)
    at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:616)
```

「オフセット: -1を選択しようとしてますね」じゃねぇんじゃ。

コールバックとかで呼ばれてエラー発生個所までの間にユーザーコード出てこないやつ割と困る。

# 解決方法

[IllegalArgumentException while selecting text in Android TextView - Stack Overflow](https://stackoverflow.com/questions/33821008/illegalargumentexception-while-selecting-text-in-android-textview/43290390#43290390)

阪本さんアイコンの人が対処方法を見つけていたので集合知すごいという感じだ。

次のようなカスタムビューを作成して、`TextView`の代わりに使用する

## SelectableTextView.kt

```kt
package com.suihan74.utilities

import android.content.Context
import android.util.AttributeSet
import android.view.MotionEvent
import androidx.appcompat.widget.AppCompatTextView

class SelectableTextView : AppCompatTextView {
    constructor(context: Context, attributeSet: AttributeSet? = null) :
            super(context, attributeSet)

    constructor(context: Context, attributeSet: AttributeSet?, defStyleAttr: Int) :
            super(context, attributeSet, defStyleAttr)

    override fun dispatchTouchEvent(event: MotionEvent?): Boolean {
        if (selectionStart != selectionEnd && event?.actionMasked == MotionEvent.ACTION_DOWN) {
            text = text
        }
        return super.dispatchTouchEvent(event)
    }
}
```

`text = text`の部分が肝心のようで、これだけ見ると何やってんのこの人って感じだけれども、とにかく一度`setText(...)`を呼ぶことで内部状態がクリアされるのか何なのかで「カーソル位置が-1」みたいなよくわからない状態は回避される。

`setText(...)`の中身を深掘りするのはまた今度暇と元気があったらというかなんというか。する必要はとくに感じないわけで。

# 余談

リンク先で「`TextView`じゃなくて`AppCompatTextView`使った方がいいよ」と言ってる人がいたので、二者の違いをよく知らなかったから調べた。

[AppCompatTextView - Android Developers](https://developer.android.com/reference/android/support/v7/widget/AppCompatTextView)

古いバージョンの端末で動かしている場合にbackground tintingやらauto sizingやらが働かないので、その場合には明示的に`AppCompatTextView`を使用する必要があるみたいなことが書いてある。

[【v7 appcompat library を読む】 レイアウト XML のインフレート時に各種 view が compatible widget に変換される仕組み - ひだまりソケットは壊れない](https://vividcode.hatenablog.com/entry/android-app/library/v7-appcompat-compatible-widget-inflation)

また、`AppCompatActivity`などで使用する場合においてはレイアウトファイル側で`TextView`と記述していてもinflate時に自動的に`AppCompatTextView`に置換してくれるらしいので、とくに明示的に使用する必要はない。

ただし、`LayoutInflater`でのinflateを介さない他のすべての場合では明示的に使用する必要がある。  
つまり今回のように`TextView`を継承したカスタムビューを作成するなどの場合には`TextView`ではなく`AppCompatTextView`を継承する必要があるということらしい。  
これは他の`AppCompatナンタラ`なViewすべてに共通か。
