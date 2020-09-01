---
title: "AndroidでQRコードを生成して画面に表示する"
description: "zxingでQRコード生成"
tags: ["Android", "kotlin", "QRcode"]
date: 2020-02-20T01:48:21+09:00
lastmod: 2020-09-02T02:30:00+09:00
archives:
    - 2020
    - 2020-02
    - 2020-02-20
hide_overview: true
draft: false
---

AndroidアプリでQRコードを生成して画面に表示する方法。今回は読み取りについては書いていない。

## 追記 (2020-07-21)

データバインディングの始め方についての記事へのリンクを追加。

## 追記 (2020-03-31)

lifecycleに関する依存先のバージョンを`2.2.0`にアップデート。  
それに伴い、`ViewModelProvider`を使用したViewModelのインスタンス生成方法を修正。

`ViewModelProviders.of(owner)` → `ViewModelProvider(owner)`

## 追記 (2020-09-02)

あまりにあんまりだったのでサンプルコードを修正した。  
まぁ多少はマシになった。

---

## 準備

ここでは(無駄に)データバインディングを使用しているので、準備が必要ならしておく。

[Android - DataBindingはじめ - すいはんぶろぐ.io](/posts/2020/01_02_00_beginning_of_data_binding/#buildgradle-app)

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

## Model

```kt
package com.suihan74.sample

import android.graphics.Color
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel

/** QRコード生成の元情報 */
data class QRSource(
    /** QR化するデータ */
    val data: String,

    /**
     * QRコードサイズ(dp)
     *
     * margin=0にしてImageViewのサイズいっぱいに拡縮無しで表示するためにここではdpで扱っている
     */
    val size: Int,

    /** 誤り訂正レベル */
    val errorCorrectionLevel: ErrorCorrectionLevel,

    /** 文字コード */
    val charset: String,

    /**
     * マージン(セル数, 4セル以上の余白が必要)
     *
     * レイアウト側で十分な余白を用意してある場合は0とかにしておいた方が制御しやすいかもしれない
     */
    val margin: Int = 4,

    /** 前景色 */
    val foregroundColor: Int = Color.BLACK,

    /** 背景色 */
    val backgroundColor: Int = Color.WHITE
)
```

## ViewModel

```kt
package com.suihan74.sample

import android.content.Context
import android.graphics.Bitmap
import android.graphics.Color
import android.util.Log
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.google.zxing.BarcodeFormat
import com.google.zxing.EncodeHintType
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel
import com.journeyapps.barcodescanner.BarcodeEncoder
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch

class HogeViewModel : ViewModel() {
    /** QRコード化する情報 */
    var qrSource: QRSource? = null
        private set

    /** QRコードのビットマップ */
    val qrBitmap: LiveData<Bitmap?> by lazy {
        MutableLiveData<Bitmap?>(null)
    }

    /** QRコード化する情報をセット */
    fun setQRSource(qrSource: QRSource?, context: Context) {
        this.qrSource = qrSource
        viewModelScope.launch(Dispatchers.Default) {
            (qrBitmap as MutableLiveData<Bitmap?>).postValue(
                generateQRCodeBitmap(qrSource, context)
            )
        }
    }

    /** QRコードのビットマップを生成 */
    private fun generateQRCodeBitmap(qrSource: QRSource?, context: Context) : Bitmap? =
        if (qrSource == null) null
        else try {
            // dpからpxに変換する
            // 他に良いやりようがあるんだろうなという感じはする
            val density = context.resources.displayMetrics.density
            val pxSize = (qrSource.size * density).toInt()

            // 生成に関するパラメータ
            val hints = mapOf(
                // マージン
                EncodeHintType.MARGIN to qrSource.margin,
                // 誤り訂正レベル
                EncodeHintType.ERROR_CORRECTION to qrSource.errorCorrectionLevel,
                // 文字コード
                EncodeHintType.CHARACTER_SET to qrSource.charset
            )

            BarcodeEncoder().encodeBitmap(
                qrSource.data,
                BarcodeFormat.QR_CODE,
                pxSize, pxSize,
                hints
            ).also { encoder ->
                // 黒白以外にしたいならここで適当にQRコードを色付けする
                val pixels = IntArray(pxSize * pxSize)
                encoder.getPixels(pixels, 0, pxSize, 0, 0, pxSize, pxSize)
                for (idx in pixels.indices) {
                    pixels[idx] =
                        if (pixels[idx] == Color.BLACK) qrSource.foregroundColor
                        else qrSource.backgroundColor
                }
                encoder.setPixels(pixels, 0, pxSize, 0, 0, pxSize, pxSize)
            }
        }
        catch (e: Throwable) {
            Log.e("genQRCode", Log.getStackTraceString(e))
            null
        }
}
```

## Activity

```kt
package com.suihan74.sample

import androidx.appcompat.app.AppCompatActivity
import androidx.databinding.DataBindingUtil
import androidx.lifecycle.ViewModelProviders
import com.suihan74.sample.R
import com.suihan74.sample.databinding.ActivityHogeBinding
import kotlinx.android.synthetic.main.activity_hoge.*

class HogeActivity : AppCompatActivity {
    private val viewModel: HogeViewModel by lazy {
        ViewModelProvider(this)[HogeViewModel::class.java]
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

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
package com.suihan74.sample

import android.graphics.Bitmap
import android.widget.ImageView
import androidx.databinding.BindingAdapter

/** ImageViewにBitmapをバインドする */
@BindingAdapter("bitmap")
fun ImageView.setBitmapSource(bitmap: Bitmap?) {
    this.setImageBitmap(bitmap)
}
```

## Layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="vm"
            type="com.suihan74.sample.HogeViewModel" />
    </data>

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageView
            android:id="@+id/QRImageView"
            bitmap="@{vm.qrBitmap}"
            android:contentDescription="@null"
            android:scaleType="center"
            android:layout_width="@dimen/qr_size"
            android:layout_height="@dimen/qr_size" />

...

    </LinearLayout>
</layout>
```

`Bitmap`の生成はコルーチンで行い、生成完了したらpostValue()で渡す。

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

