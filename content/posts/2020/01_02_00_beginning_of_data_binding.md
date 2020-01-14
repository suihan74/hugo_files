---
title: "Android - DataBindingはじめ"
description: いまさらAndroidでDataBinding触れはじめてみた浅い記事
tags: ["android", "kotlin", "DataBinding", "ViewModel", "LiveData"]
date: 2020-01-02T17:04:24+09:00
lastmod: 2020-01-02T17:04:24+09:00
draft: false
---

最近ずっとSatenaのActivity/Fragmentのコード側を作り直す作業をしていて、ViewModel + LiveDataは割と活用してきて"前よりは"いい感じになってきているのだが、  
DataBindingに関しては以前UWPアプリ作るとき（中途半端に）触れて以来、MVVMというかデータバインディングやったらやったでそれはそれで色々と面倒なことがあるイメージがあったので積極的に取り入れてなかったのだけど、楽できそうな部分ではやっていこうかと唐突に思った。

今さら感強い上にまだ詳しく使い込んでいないので、  
とりあえず導入的なものと、  
主に忘れそうな部分についていくつかメモを書く。

# 導入

## build.gradle (app)

ViewModelやらDataBindingを使用するのに必要な設定・依存関係を追加する。

```gradle
android {
    ...
    // DataBinding利用に必要
    dataBinding {
        enabled = true
    }
    ...
}

dependencies {
    ...
    // ViewModel and LiveData
    def lifecycle_version = "2.1.0"
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"
    testImplementation "androidx.arch.core:core-testing:$lifecycle_version"
    ...
}
```

## activity_hoge.xml

以前のレイアウトの内容を`<layout>`で囲い、`<data> ~ </data>`にバインドに必要な情報を記述する。  
ここでは、コード側で用意してバインドする`ViewModel`のオブジェクトをレイアウトファイル内では`vm`として扱う。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- bindするデータ -->
    <data>
        <variable
                name="vm"
                type="com.suihan74.hoge.HogeViewModel" />
    </data>

    <!-- 以下表示部分 -->
    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:id="@+id/hoge_layout"
        android:background="?attr/panelBackground"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        ...

    </androidx.coordinatorlayout.widget.CoordinatorLayout>
</layout>
```

## HogeActivity.kt

ActivityHogeBindingはレイアウトまで書いてビルドすると自動的に生成される。  
`ViewDataBinding`を継承した`"アッパーキャメルなレイアウトファイル名 + Binding"`という名前のクラスができるので、これを`DataBindingUtil.setContentView<T>(...)`の型引数に与えて呼ぶとバインドが開始される。

```kt
class HogeActivity : AppCompatActivity() {
    private lateinit var viewModel: HogeViewModel
    private lateinit var binding: ActivityHogeBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // ViewModelの生成/取得
        // ViewModel用のFactoryを用意しておけばViewModelを生成する際コンストラクタに値を渡すことができる
        val factory = HogeViewModel.Factory(
            ...
        )
        viewModel = ViewModelProviders.of(this, factory)[HogeViewModel::class.java]

        // データバインド
        binding = DataBindingUtil.setContentView<ActivityHogeBinding>(
            this,
            R.layout.activity_hoge
        ).apply {
            lifecycleOwner = this@BookmarkPostActivity
            vm = viewModel
        }
    }

    ...
}
```

バインドするデータに`LiveData`を使用する場合、作成した`binding`に適切な`lifecycleOwner`を渡す必要がある。これを行わないと`LiveData`の値変更がレイアウトにうまく通知されない。

`ViewModel`を使うことで`savedInstanceState`はせいぜい初回起動か復元後かを判別する程度の用途しか無くなりましたとさ。良さ。

# データの変更をViewに反映する

たとえば次のようなViewModelを用意するとする。  
`LiveData<T>`は`LiveData<T>.value`が変更された際に生きている監視者に変更を通知するもの。`LiveData<T>`を継承して好きなように作ることもできるが、ここでは面倒なので`value`を外から変更できる`MutableLiveData<T>`を使うことにする。

## HogeViewModel.kt

```kt
class HogeViewModel(
    private val repository : HogeRepository
) : ViewModel() {
    val text by lazy {
        MutableLiveData<String>("")
    }

    ...

    // DIできるよ的な例
    class Factory(
        private val repository : HogeRepository
    ) : ViewModelProvider.NewInstanceFactory() {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel?> create(modelClass: Class<T>) =
            HogeViewModel(repository) as T
    }
}
```

これを先ほどのレイアウトファイルに`TextView`を追加して、データバインディングで表示するようにするには次のように`TextView`を記述する。

```xml
<TextView
        android:text="@{vm.text}"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
```

`@{...}`でViewModel→Viewの単方向データバインディング。

これで`viewModel.text`が変更されたときに`TextView`の表示も変更されるようになる。楽。


# Viewの変更をデータに反映する

`EditText`の入力テキストを先ほどの`HogeViewModel.text`に反映する例。

```xml
<EditText
        android:text="@={vm.text}"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
```

`@={...}`でViewModel⇔Viewの双方向データバインディング。

他にもチェックボックスやトグルボタンの状態管理など。


# 真偽値をバインドしてVisibilityを変更する例

楽そうだと思ったのはこれ。

```xml
...
<data>
    <import type="android.view.View"/>
    ...
</data>

...

<TextView
        android:text="@{vm.text}"
        android:visibility="@{vm.display ? View.VISIBLE : View.GONE}"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
```

バインディング式にView.VISIBLEなどを記述する場合、`<data>~</data>`にて`View`をインポートする必要がある。

どうでもいいけど、最近kotlinばかり書いていて三項演算子懐かしいってなった。

[レイアウトとバインディング式 | Android Developers](https://developer.android.com/topic/libraries/data-binding/expressions?hl=ja)

もっと凝ったことをする場合は、バインディングアダプターを用意する方法をとれば良さそう。

[バインディングアダプター | Android Developers](https://developer.android.com/topic/libraries/data-binding/binding-adapters.html?hl=ja)

# 双方向バインディングでConverterを介して値をやりとりする

たとえば適当な列挙型`HogeEnum`があるとして、これを`Spinner`に表示して選択させたいときなどに以下のようなシングルトンを用意しておいて、レイアウトから呼ぶことができる。

```kt
object HogeEnumConverter {
    @JvmStatic
    @InverseMethod("toHogeEnum")
    fun toInt(item: HogeEnum?) = item?.ordinal ?: 0

    @JvmStatic
    fun toHogeEnum(ordinal: Int) = HogeEnum.values()[ordinal]
}
```

```xml
<layout ...>
    <data>
        <import type="com.suihan74.hoge.HogeEnumConverter"/>
        <variable
                name="vm"
                type="com.suihan74.hoge.HogeViewModel" />
    </data>

    ...

    <Spinner
        android:id="@+id/spinner"
        android:entries="@array/hogeEnums"
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:selectedItemPosition="@={HogeEnumConverter.toInt(vm.selectedHogeEnum)}" />

    ...
</layout>
```

[DataBindingのInverseMethodの使い方 - Kenji Abe - Medium](https://medium.com/@star_zero/databindingのinversemethodの使い方-63f4effa49a0)

# 手をつけていないこと

以上のように特にシンプルな部分に関してはデータバインディング使っておけば基本的には楽そうである。

まだ詳しく知らないのだけども、`RecyclerView`と合わせて使うのは面倒臭そうだなぁと思ったり、イベントの処理はコード側に書けばいいんじゃないかなぁとか思ったりしている。

「全ての画面をばっちりMVVM的な感じに書き直してやるぜ」という気分には今さらあまりなれないものの、変に複雑なことになったりハマったりしないで使えそうなところではやっていこうかなという感じになった。  
よかったですね。