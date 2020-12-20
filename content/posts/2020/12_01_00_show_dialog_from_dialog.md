---
title: "ダイアログからダイアログを開く"
description: "画面回転で色々アレするやつの対策"
tags: ["Android","kotlin","DialogFragment"]
date: 2020-12-01T20:45:00+09:00
lastmod: 2020-12-01T20:45:00+09:00
archives:
    - 2020
    - 2020-12
    - 2020-12-01
hide_overview: false
draft: false
---

なんか同じようなこと度々やっては忘れている気がするので記録しておく。

`DialogFragment`を開いて、その操作結果でさらに別のダイアログを開きたくなったときなどのやり方。

## 問題

`AlertDialog`(とか)を直接開いただけでは画面回転で閉じてしまう。そこで`DialogFragment`を作成して、その中で`AlertDialog`を作る。  
こうすると画面回転後も`DialogFragment`が再生成されて操作を続行できるわけだが、ダイアログの操作結果を受けてダイアログの呼び出し元側で色々するようにしているときに`fragmentManager`や`Activity`など再生成されるもの達の参照に問題が発生する。

分かりづらいので具体例。

## 問題ある例

### 呼び出し元

たとえば何らかの`Fragment`とかで、以下のようにしてダイアログを開くことにする。

```kt:HogeFragment(問題あり).kt
fun openFooDialog(fragmentManager: FragmentManager) {
    val dialog = FooDialogFragment.createInstance()

    dialog.setPositiveAction {
        // ダイアログのポジティブボタン押下で別のダイアログ`BarDialogFragment`を開く
        // `FooDialogFragment`表示中に画面回転すると失敗する
        openBarDialog(fragmentManager)
    }

    dialog.show(fragmentManager, DIALOG_TAG_FOO)
}
```

### ダイアログ

ダイアログ自体は次のような感じ。  
`BarDialogFragment`も同じ感じとする。

```kt:FooDialogFragment(問題あり).kt
class FooDialogFragment : DialogFragment() {
    companion object {
        // 引数がある場合に`createInstance()`を経由して`setArguments()`したりしている
        // 統一のため引数有無に関わらずすべての`Fragment`にこれを用意することにしている
        fun createInstance() = FooDialogFragment()
    }

    // ------ //

    // `lazyProvideViewModel`は`ViewModelFactory`をよしなにやるやつ
    // 良い感じに`ViewModel`を生成しているだけなので別にどうでもいい
    private val viewModel by lazyProvideViewModel {
        DialogViewModel()
    }

    // ------ //

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        return AlertDialog.Builder(requireContext())
            .setTitle(R.string.dialog_title_foo)
            .setNegativeButton(R.string.dialog_cancel, null)
            .setPositiveButton(R.string.dialog_ok) { _, _ ->
                viewModel.positiveAction?.invoke()
            }
            .create()
    }

    // ------ //

    /** ViewModelを扱っていいライフサイクルに達したらリスナを設定する */
    fun setPositiveAction(l : (()->Unit)?) = lifecycleScope.launchWhenCreated {
        viewModel.positiveActioin = l
    }

    // ------ //

    class DialogViewModel : ViewModel() {
        var positiveAction : (()->Unit)? = null
    }
}
```

このような実装をしている場合、

```kt
dialog.setPositiveAction {
    // ダイアログのポジティブボタン押下で別のダイアログ`BarDialogFragment`を開く
    // `FooDialogFragment`表示中に画面回転すると失敗する
    openBarDialog(fragmentManager)
}
```

の部分で参照している`fragmentManager`が`openFooDialog()`呼び出し時点のものであるため、画面回転などで破棄されて使用不可能な状態になってしまう。  
同じ理由で`Activity`や`Fragment`への参照もキャプチャするべきではない。

## 修正例

**「操作完了時点の」** `DialogFragment`を必ず結果とあわせて返すようにする。  
その時点で生きている(アタッチしている)`Activity`も呼び出し元`Fragment`もこのインスタンスを通して取得することができる。

### 呼び出し元(修正後)

```kt:HogeFragment.kt
fun openFooDialog(fragmentManager: FragmentManager) {
    val dialog = FooDialogFragment.createInstance()

    dialog.setPositiveAction { f ->
        // ダイアログのポジティブボタン押下で別のダイアログ`BarDialogFragment`を開く
        // `FooDialogFragment`の`parentFragmentManager`なので、
        // `openFooDialog()`に渡された`fragmentManager`に相当するものである(再生成されていたらインスタンスは別である)
        openBarDialog(f.parentFragmentManager)
    }

    dialog.show(fragmentManager, DIALOG_TAG_FOO)
}
```

### ダイアログ(修正後)

```kt:FooDialogFragment.kt
class FooDialogFragment : DialogFragment() {
    companion object {
        // 引数がある場合に`createInstance()`を経由して`setArguments()`したりしている
        // 統一のため引数有無に関わらずすべての`Fragment`にこれを用意することにしている
        fun createInstance() = FooDialogFragment()
    }

    // ------ //

    // `lazyProvideViewModel`は`ViewModelFactory`をよしなにやるやつ
    // 良い感じに`ViewModel`を生成しているだけなので別にどうでもいい
    private val viewModel by lazyProvideViewModel {
        DialogViewModel()
    }

    // ------ //

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        return AlertDialog.Builder(requireContext())
            .setTitle(R.string.dialog_title_foo)
            .setNegativeButton(R.string.dialog_cancel, null)
            .setPositiveButton(R.string.dialog_ok) { _, _ ->
                viewModel.positiveAction?.invoke(this)
            }
            .create()
    }

    // ------ //

    /** ViewModelを扱っていいライフサイクルに達したらリスナを設定する */
    fun setPositiveAction(l : ((FooDialogFragment)->Unit)?) = lifecycleScope.launchWhenCreated {
        viewModel.positiveActioin = l
    }

    // ------ //

    class DialogViewModel : ViewModel() {
        var positiveAction : ((FooDialogFragment)->Unit)? = null
    }
}
```
