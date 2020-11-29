---
title: "BindingAdapterに関するいくつかのこと"
description: "シンプルな方法でビューに値をセットする。"
tags: ["android", "kotlin", "DataBinding"]
date: 2020-03-07T20:30:49+09:00
lastmod: 2020-11-30T01:30:00+09:00
archives:
    - 2020
    - 2020-03
    - 2020-03-07
draft: false
---

## 追記 (2020-11-30)

双方向バインディングのやり方を書いてなかったので追記。

[#双方向バインディング](#双方向バインディング)

### 拡張関数かオブジェクトかの選択

以下2通りの方法でバインディングアダプタを記述することができるが、基本はオブジェクトの方を使った方がいいと思う。  
コード側でも共用したい場合に限り拡張関数の方を使う感じで。

#### 拡張関数として用意する

```kt
@BindingAdapter("items")
fun RecyclerView.setItems(items: List<Item>?) {
    // 省略
}
```

#### オブジェクトとして用意する

```kt
object RecyclerViewBindingAdapters {
    @JvmStatic
    @BindingAdapter("items")
    fun setItems(view: RecyclerView, items: List<Item>?) {
        // 省略
    }
}
```

---

## DataBinding

データバインディングを使うと色々嬉しいですねという話。導入は前書いた。

[Android - DataBindingはじめ - すいはんぶろぐ.io](/posts/2020/01_02_00_beginning_of_data_binding/)

前記事で`BindingAdapter`を使う方法を書いてなかったのでここに記録しておく。

以下の内容は自分がやったことのみ記述していますので、詳細な情報は一次情報にあたってください。

[バインディング アダプター &nbsp;|&nbsp; Android デベロッパー &nbsp;|&nbsp; Android Developers](https://developer.android.com/topic/libraries/data-binding/binding-adapters)

## BindingAdapter

例えば「真偽値をVisibilityに変換して指定する」例。

### hoge.kt

```kt
@BindingAdapter("android:visibility")
fun View.setBoolToVisibility(b: Boolean?) {
    this.visibility =
        if (b == true) View.VISIBLE
        else View.GONE
}
```

kotlinの場合、指定する対象にこのような拡張関数を用意するだけでいい。

バインディングアダプタは次のように名前と値の型が符合する属性値を指定した場合にのみ実行される。  
アダプタにする拡張関数の引数型は必ずしもnullableである必要はない。しかし、その場合`null`が渡された途端に実行時エラーになるので注意がいる。

### layout.xml

```xml
<!-- これであたかもandroid:visibilityに直接真偽値をセットしているように書ける -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="@{vm.hogeBool}"
/>
```

なお、`vm.hogeBool`が`LiveData<Boolean>`とかの場合でもこの書き方でいい(`"@{vm.hogeBool.value}"`とかにしない)

## 複数の属性を扱う

上記の例では`android:visibility`だけをセットしていたが、「複数の属性をすべてセットしないといけないようにする」こともできる。

たとえば「`ImageView`に画像ソースのURL文字列を渡したいが、値がnullの場合は非nullなデフォルト値を使用する」みたいな例。

```kt
@BindingAdapter("src", "srcDefault")
fun ImageView.setSource(src: String?, defaultSrc: String) {
    Glide.with(context)
        .load(src ?: defaultSrc)
        .into(this)
}
```

単純に、`@BindingAdapter`に渡す属性名を増やせばいい。

なおこの方法だと「`requireAll = true`」、つまり指定したすべての属性値に値がちゃんと指定されていないとコンパイルエラーになる。  
あってもなくてもいい属性がある場合は次のようにできる。

```kt
@BindingAdapter(value = ["src", "srcDefault"], requireAll = false)
fun ImageView.setSource(src: String?, defaultSrc: String?) {
    val imgSrc = src ?: defaultSrc ?: return
    Glide.with(context)
        .load(imgSrc)
        .into(this)
}
```

(属性の組み合わせ分だけアダプタを宣言してもいいが、冗長にはなる)

```kt
@BindingAdapter("src")
fun ImageView.setSource(src: String?) {
    if (src == null) return
    Glide.with(context)
        .load(src)
        .into(this)
}

@BindingAdapter("src", "srcDefault")
fun ImageView.setSource(src: String?, defaultSrc: String?) {
    val imgSrc = src ?: defaultSrc ?: return
    Glide.with(context)
        .load(imgSrc)
        .into(this)
}
```

## 属性名の名前空間

バインディングアダプタの`value`に指定する属性名はここまで`@BindingAdapter("hoge")`のように記述してきたが、このときこのバインディングアダプタは「属性値の型が一致する ***"android:"以外の***すべての名前空間のhoge」にマッチする。  
(`"app:hoge"`、`"hoge"`...)

`"android:"`にマッチさせる為には明示的に`@BindingAdapter("android:hoge")`と書く必要がある。

## 値の変更を処理する

属性値が`oldValue`から`newValue`に変更されたときその両方の値を使って何かすることもできる。

```kt
@BindingAdapter("hoge")
fun View.setHoge(oldValue: Hoge?, newValue: Hoge?) {
    ...
}
```

### 複数属性の場合

```kt
@BindingAdapter("a", "b")
fun View.setHoge(oldA: Hoge?, oldB: Hoge?, newA: Hoge?, newB: Hoge?) {
    ...
}
```

## 双方向バインディング

`Slider`の現在値(`android:value`)を双方向バインドする例。

```kt

object SliderBindingAdapters {

    /** Model --> View */
    @JvmStatic
    @BindingAdapter("android:value")
    fun bindSliderValue(slider: Slider, value: Float?) {
        if (value != null && slider.value != value) {
            slider.value = value
        }
    }

    /** View --> Model */
    @JvmStatic
    @InverseBindingAdapter(attribute = "android:value")
    fun bindSliderValueInverse(slider: Slider) : Float {
        return slider.value
    }

    /**
     * `android:value`が更新されたことを検知するリスナを登録して、
     * 値変更時に`bindSliderValueInverse`を呼び出す
     */
    @JvmStatic
    @BindingAdapter("android:valueAttrChanged")
    fun bindListeners(slider: Slider, valueAttrChanged: InverseBindingListener?) {
        slider.addOnChangeListener { _, _, _ ->
            valueAttrChanged?.onChange()
        }
    }
}
```

`InverseBindingAdapter`と、リスナをセットするための`BindingAdapter`を書くのが肝。

バインドするリスナの属性名はデフォルトでは`(値バインド対象属性名+)AttrChanged`となる。(今回の場合`android:value`の変更を監視するから`android:valueAttrChanged`)

任意の属性名を設定したい場合は`@InverseBindingAdapter`アノテーションの引数に`event="~~~"`という形で渡す。

`bindListeners`で設定しているリスナの中身は、要は値が更新されたことだけ伝えられればいいのでただ単純に`valueAttrChanged?.onChange()`を呼んでいる。(`InverseBindingListener`が呼ばれる)
