---
title: "コンストラクタ引数有りのViewModelを簡単に作成する"
description: "NewInstanceFactoryを継承したFactoryをいちいち用意しないようにする一方法"
tags: ["Android", "kotlin", "ViewModel"]
date: 2020-09-14T16:49:08+09:00
lastmod: 2020-12-27T12:50:00+09:00
archives:
    - 2020
    - 2020-09
    - 2020-09-14
hide_overview: false
draft: false
---

## 追記 (2020-12-27)

`ViewModelOwner#lazyProvideViewModel`を追加。[▼](#lazyprovideviewmodelを追加-2020-12-27)

---

## 前提1

```ViewModel```の最もシンプルな作り方は多分こんな感じだろう。

```kt
class HogeViewModel : ViewModel() {
    // LiveDataなど色々
}
```

```kt
class HogeActivity {
    private val viewModel : HogeViewModel by lazy {
        ViewModelProvider(this)[HogeViewModel::class.java]
    }
    // 他省略
}
```

```ViewModelProvider```は```ViewModelStoreOwner```と```ViewModelProvider.Factory```を引数にとり、後者を省略すると無引数コンストラクタで```ViewModel```が生成される。  
後半の```[]```の中身は、```(省略可能な)キー名```と```生成するViewModelのクラス```を指定する。```キー```は```owner```が同型の```ViewModel```を複数持つ場合にそれぞれを識別するために使う。

## 前提2

以上の方法では```ViewModel```が引数有りコンストラクタを用意することができないため、外部から初期値を与えたい場合などは対象のプロパティをミュータブルにせざるを得ない問題がある。  
きっちりイミュータブルにしておきたい場合は、次のようにインスタンス生成用の```Factory```を用意する。

```kt:普通にプライマリコンストラクタでプロパティ宣言する例.kt
class HogeViewModel(
    private val hogeArg : Hoge
) : ViewModel() {
    // ------ //
    class Factory(
        private val hogeArg : Hoge
    ) : ViewModelProvider.NewInstanceFactory() {
        @Suppress("unchecked_cast")
        override fun <T : ViewModel?> create(modelClass: Class<T>): T {
            return HogeViewModel(hogeArg) as T
        }
    }
}
```

```kt:引数付きのViewModel生成例.kt
class HogeActivity {
    private val viewModel : HogeViewModel by lazy {
        val hoge = Hoge()
        val factory = HogeViewModel.Factory(hoge)
        ViewModelProvider(this, factory)[HogeViewModel::class.java]
    }
    // 他省略
}
```

```HogeActivity```側で```HogeViewModel```生成時に```Factory```を渡すことで、```Factory#create()```メソッドによってインスタンスが生成される。

## 簡単化

新しい```ViewModel```を作るごとにこの```Factory```を毎回書くのは少々面倒であるし無駄に冗長なので、```Factory```を作る関数と、作った```Factory```を使って```ViewModel```インスタンスを生成する関数を作って簡単にする。

```kt:ViewModelFactoryUtil.kt
package com.suihan74.utilities

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.ViewModelStoreOwner
import kotlin.reflect.KClass

class ViewModelFactory<ViewModelT : ViewModel>(
    private val creator: () -> ViewModelT,
    private val kClass: KClass<ViewModelT>
) : ViewModelProvider.NewInstanceFactory() {

    @Suppress("unchecked_cast")
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return creator.invoke() as T
    }

    /** ViewModelを作成・取得する */
    fun provide(owner: ViewModelStoreOwner, key: String? = null) : ViewModelT =
        if (key == null) ViewModelProvider(owner, this)[kClass.java]
        else ViewModelProvider(owner, this)[key, kClass.java]
}

/**
 * ViewModel作成時に用いるNewInstanceFactoryを生成する
 */
inline fun <reified ViewModelT : ViewModel> createViewModelFactory(
    noinline creator: ()->ViewModelT
) = ViewModelFactory(creator, ViewModelT::class)

/**
 * ViewModelを作成・取得する
 */
inline fun <reified ViewModelT : ViewModel> provideViewModel(
    owner: ViewModelStoreOwner,
    noinline creator: ()->ViewModelT
) = createViewModelFactory(creator).provide(owner)

/**
 * ViewModelを作成・取得する
 */
inline fun <reified ViewModelT : ViewModel> provideViewModel(
    owner: ViewModelStoreOwner,
    key: String?,
    noinline creator: ()->ViewModelT
) = createViewModelFactory(creator).provide(owner, key)

// ------ //

/**
 * ViewModelを作成・取得する(lazy版)
 */
inline fun <reified ViewModelT : ViewModel> ViewModelStoreOwner.lazyProvideViewModel(
    noinline creator: ()->ViewModelT
) = lazy { provideViewModel(this, creator) }

/**
 * ViewModelを作成・取得する(lazy版)
 */
inline fun <reified ViewModelT : ViewModel> ViewModelStoreOwner.lazyProvideViewModel(
    key: String?,
    noinline creator: ()->ViewModelT
) = lazy { provideViewModel(this, key, creator) }
```

以上の内容のファイルを用意しておいて、次のように使う。

```kt:HogeViewModel.kt
class HogeViewModel(
    private val hoge: Hoge
) : ViewModel() {
    // 省略
}
```

```kt:HogeActivity.kt
class HogeActivity {
    private val viewModel : HogeViewModel by lazy {
        provideViewModel(this) {
            val hoge = Hoge()
            HogeViewModel(hoge)
        }
    }
    // 他省略
}
```

```キー```を使う場合は```provideViewModel(this, key) { // --- // }```とする。

これで少しはすっきりしたというものです。  
よかったですね。

## lazyProvideViewModelを追加 (2020-12-27)

```kt
private val viewModel by lazy {
    provideViewModel(this) {
        HogeViewModel()
    }
}
```

をもう少し簡単に書くために`lazyProvideViewModel`関数を追加した。

```kt
private val viewModel by lazyProvideViewModel {
    HogeViewModel()
}
```

使用できる`ViewModelStoreOwner`が`this`限定である(他の`owner`を渡せない)という制限がある点には注意。  
これは`Fragment`のプロパティ初期化などで`Activity`などを`owner`に指定すると`Fragment`インスタンス生成時にはまだアタッチ前なので失敗する問題があるため、このようにしている。
