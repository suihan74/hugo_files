---
title: "AndroidでQRコードを生成して画面に表示する"
description: "zxingでQRコード生成"
tags: ["Android", "kotlin", "QRcode"]
date: 2020-02-20T01:48:21+09:00
lastmod: 2020-03-31T17:50:00+09:00
archives:
    - 2020
    - 2020-02
    - 2020-02-20
hide_overview: true
draft: false
---

AndroidアプリでQRコードを生成して画面に表示する方法。今回は読み取りについては書いていない。

## 追記 (2020-03-31)

lifecycleに関する依存先のバージョンを`2.2.0`にアップデート。  
それに伴い、`ViewModelProvider`を使用したViewModelのインスタンス生成方法を修正。

`ViewModelProviders.of(owner)` → `ViewModelProvider(owner)`


## build.gradle (app)

[journeyapps/zxing-android-embedded](https://github.com/journeyapps/zxing-android-embedded)というライブラリを使うことにした。

```gradle
dependencies {
...
    // QR code
    implementation 'com.journeyapps:zxing-android-embedded:4.1.0'
}
```

READMEに色々書いてあるが、QRコードの読み込みはしないで単純にQRコードを生成するだけならそんなに色々やる必要はなさそう。

## ViewModel

```kt
class HogeViewModel : ViewModel() {
    /** QRコード化する文字列 */
    val qrData by lazy {
        MutableLiveData<String>("https://foo.bar.baz/")
    }

    /** QRコードの色 */
    val qrForegroundColor by lazy {
        MutableLiveData<Int>(Color.BLACK)
    }

    /** QRコードの背景色 */
    val qrBackgroundColor by lazy {
        MutableLiveData<Int>(Color.WHITE)
    }

    /** コルーチンスコープ */
    val coroutineScope get() = this.viewModelScope
    // DataBinding経由で渡すためにプロパティとして用意し直している。冗長感ある
}
```

## Activity

```kt
class HogeActivity : AppCompatActivity {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val viewModel = ViewModelProvider(this)[HogeViewModel::class.java]
        DataBindingUtil.setContentView<ActivityHogeBinding>(
            this,
            R.layout.activity_hoge
        ).apply {
            lifecycleOwner = this@HogeActivity
            vm = viewModel
        }
    }
}
```

## BindingAdapter

```kt
/** ImageViewにQRコードを描画するBindingAdapter */
@BindingAdapter("app:qrSrc", "app:background", "app:foreground", "app:coroutineScope")
fun AppCompatImageView.generateQRCode(data: String?, backgroundColor: Int?, foregroundColor: Int?, coroutineScope: CoroutineScope?) {
    if (data.isNullOrBlank()) return
    if (backgroundColor == null) return
    if (foregroundColor == null) return
    if (coroutineScope == null) return

    val pxSize = context.resources.getDimension(R.dimen.qr_size).toInt()

    coroutineScope.launch(Dispatchers.Default) {
        val bitmap = generateQRCodeBitmap(
            data,
            backgroundColor,
            foregroundColor,
            pxSize
        ) ?: return@launch

        withContext(Dispatchers.Main) {
            setImageBitmap(bitmap)
        }
    }
}

/** QRコードのBitmapを生成 */
private fun generateQRCodeBitmap(data: String, backgroundColor: Int, foregroundColor: Int, pxSize: Int) : Bitmap? {
    return try {
        val hints = mapOf(
            // マージン指定
            EncodeHintType.MARGIN to 0,
            // 誤り訂正レベルを指定
            EncodeHintType.ERROR_CORRECTION to ErrorCorrectionLevel.M
        )

        val barcodeEncoder = BarcodeEncoder()
        barcodeEncoder.encodeBitmap(data, BarcodeFormat.QR_CODE, pxSize, pxSize, hints).apply {
            // QRコードを色付けする
            val pixels = IntArray(pxSize * pxSize)
            getPixels(pixels, 0, pxSize, 0, 0, pxSize, pxSize)
            pixels.indices.forEach { idx ->
                pixels[idx] =
                    if (pixels[idx] == Color.BLACK) foregroundColor
                    else backgroundColor
            }
            setPixels(pixels, 0, pxSize, 0, 0, pxSize, pxSize)
        }
    }
    catch (e: Throwable) {
        Log.e("QR error", Log.getStackTraceString(e))
        null
    }
}
```

`Bitmap`の生成はコルーチンで行い、生成完了したらUIスレッドで`ImageView`に渡す。

bindingを経由した拡張関数へのコルーチンスコープの渡し方に無理矢理感を感じる。ViewModelごと直接渡してしまった方がシンプルでいいかもしれない。

### EncodeHintType

`barcodeEncoder.encodeBitmap()`に`hints`を渡すことで色々指定できる。

指定できる項目は以下。QRコード以外の二次元コードに関する設定値についての説明は適当(というかよく知らないのでコメント直訳)

- `ERROR_CORRECTION` …… 誤り訂正レベルの指定。  
  - QRコードのエンコーディングでは`ErrorCorrectionLevel.L,M,Q,H`のどれかを指定できる。  
  訂正率をより高くすれば汚れや破損に強くなるが、誤り訂正に必要な情報量が増える。バージョンを指定する場合、エンコード対象のデータ量と誤り訂正に必要なデータ量を考慮する必要がある。

  |ErrorCorrectionLevel|誤り訂正率(%)|
  | :---:| :---: |
  |L|7|
  |M|15|
  |Q|25|
  |H|30|

  - Aztecコードの場合は1~99のパーセンテージを整数値で与えるらしい

  - PDF417コードの場合は0~8の整数値を与えるらしい

- `CHARACTER_SET` …… dataの文字コード (デフォルト: "ISO-8859-1")  
  URLをエンコードしたい場合はデフォルトでとくに問題にならないように思えるが、日本語文字列などをエンコードしたい場合は指定する必要がある。

- `MARGIN` …… マージンサイズの指定 (デフォルト: 4)  
  QRコードは周囲4セル以上のマージンが必要らしいが、レイアウト側で別途マージンを指定している場合には0にしておいた方が扱いやすいように思える。

- `QR_VERSION` …… QRコードのバージョン (デフォルト: データサイズと訂正率に応じて変化？)  
  1~40で指定できる。バージョンごとにセル数が決まっており、バージョンが大きいほどよりセル数が多くなる(表現できる情報量が増える)  
  参考: <https://www.qrcode.com/about/version.html>

- `DATA_MATRIX_SHAPE` …… 正方形か長方形かということを指定するらしい  
  通常のQRコードの場合とくに指定しないか`FORCE_SQUARE`でいい  
  - `SynbolShapeHint.FORCE_NONE` …… 指定なし
  - `SynbolShapeHint.FORCE_SQUARE` …… 正方形
  - `SynbolShapeHint.FORCE_RECTANGLE` …… 長方形

以下QRコード**以外**に対するもの

- `PDF417_COMPACT` …… PDF417コードでコンパクトモードを使用するか (true or false)

- `PDF417_COMPACTION` …… PDF417コードでのコンパクト化する際のモード

- `PDF417_DIMENSIONS` …… 行数・列数の最大最小値(Dimensions型)

- `AZTEC_LAYERS` …… Aztecコードのレイヤー数
  - (-1,-2,-3,-4) …… コンパクトAztecコード
  - 0 …… 最小レイヤー数になるようにする (デフォルト)
  - (1,2,...,32) …… 通常の(非コンパクトな)Aztecコード

- `GS1_FORMAT` …… GS1標準のデータエンコードを強制するか (true or false)  
  GS1QRコードについてはこの辺？ <https://www.dsri.jp/standard/2d-symbol/gs1-qr.html>

## レイアウト

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="vm"
            type="com.suihan74.hoge.HogeViewModel" />
    </data>

    <LinearLayout
        android:orientation="vertical"
        android:background="@{vm.backgroundColor}"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.appcompat.widget.AppCompatImageView
            app:qrSrc="@{vm.qrData}"
            app:background="@{vm.qrBackgroundColor}"
            app:foreground="@{vm.qrForegroundColor}"
            app:coroutineScope="@{vm.coroutineScope}"
            android:scaleType="center"
            android:layout_width="@dimen/qr_size"
            android:layout_height="@dimen/qr_size"
            />

...

    </LinearLayout>
</layout>
```
