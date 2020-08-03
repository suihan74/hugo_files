---
title: "入力メソッドの表示に関するTIPS"
description: "ややこしい思いをしたのでとりあえず問題のなくなった状態を記録する。"
tags: ["android", "kotlin", "InputMethod"]
date: 2020-03-26T22:14:43+09:00
lastmod: 2020-08-03T15:10:00+09:00
archives:
    - 2020
    - 2020-03
    - 2020-03-26
hide_overview: false
draft: false
---

## 追記 (2020-08-03)

```Activity.hideSoftInputMethod()```にフォーカス先のビューを渡せるようにした。  
以前の単純に```window.decorView.rootView```を使う方法だと、ルートが```focusable = false```の場合に文字入力先のビューからフォーカスが外れないため。より確実にフォーカスを外すには、画面中の```focusable = true```な適当なViewやViewGroupを渡せばよい。

---

久しぶりに触ってたら「わっかんねぇよ」とブチギレた件。

## 拡張メソッド

表示・非表示の切り替えについて次のような拡張関数を用意した。

```kt
/**
 * キーボードを表示して対象にフォーカスする
 */
fun Activity.showSoftInputMethod(
    targetView: View,
    softInputMode: Int? = WindowManager.LayoutParams.SOFT_INPUT_STATE_VISIBLE
) {
    if (softInputMode != null) {
        window?.setSoftInputMode(softInputMode)
    }

    targetView.requestFocus()
    (getSystemService(Context.INPUT_METHOD_SERVICE) as? InputMethodManager)?.run {
        showSoftInput(targetView, 0)
    }
}

/**
 * キーボードを隠して入力対象のビューをアンフォーカスする
 */
fun Activity.hideSoftInputMethod(focusTarget: View? = null) : Boolean {
    val windowToken = window.decorView.windowToken
    val imm = getSystemService(Context.INPUT_METHOD_SERVICE) as? InputMethodManager
    val result = imm?.hideSoftInputFromWindow(windowToken, InputMethodManager.HIDE_NOT_ALWAYS)
    currentFocus?.clearFocus()
    (focusTarget ?: window.decorView.rootView)?.requestFocus()
    return result ?: false
}
```

削れる部分もありそうだけど、とりあえずこれでちゃんと動く。

使う時は基本的には次のようにする。

```kt
// 表示
requireActivity().showSoftInputMethod(editText)

// 非表示
// mainLayoutはfocusableな(IMEのフォーカス対象ではない)適当なビューでよい
requireActivity().hideSoftInputMethod(mainLayout)
```

`imm.showSoftInput(view, flag)`の`flag`には`InputMethodManager.SHOW_FORCED`とかいかにも強制的に表示できそうな値が渡せるのだが、どうもこれは使用しない方がよさそう。これを渡すとあとで入力メソッドを隠す際にうまくいかなくなる場合があり、非常に混乱した。

## DialogFragment(+AlertDialog)と併用する際の注意

このままこいつを使って「`DialogFragment`オープンと同時に入力メソッドを開いてダイアログ上の`EditText`にフォーカス」しようとすると失敗する。一瞬だけ入力メソッドが開いて即座に閉じるといったことが起こる。  
ここでまた非常に混乱した。入力メソッド死ね死ぬな生きろという気持ちになった。

`AlertDialog.Builder`とかでダイアログ本体を生成してから`dialog.window`に対して次のように設定する必要がある。

```kt
// 以下、AlertDialog.Builder(...).~~~.create()とか.show()とかやった後でできあがったdialogインスタンスに対して行う

// IME表示を維持するための設定
dialog.window?.run {
    clearFlags(WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM)
    setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE)
}
// ダイアログオープンと同時にIME表示
requireActivity().showSoftInputMethod(editText, WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_HIDDEN)
```

先ほど用意した`showSoftInputMethod`の`flag`には必ず`WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_HIDDEN`を指定する。  
この辺ややこしいので`DialogFragment`に専用の拡張メソッドなり何なりを用意しておく方がスマートかもしれない。

以上の指定を行うことで`AlertDialog`上でも問題なく入力メソッドを使用することができる。

### 参考

[Close/hide android soft keyboard - Stack Overflow](https://stackoverflow.com/a/10439581)
