---
title: "AlertDialogにスタイルが適用されなくて無邪気におかしいなとか思った話"
description: android接頭辞のつく属性はサポートライブラリのスタイルには使わない。
tags: ["android"]
date: 2020-01-02T22:15:23+09:00
lastmod: 2020-01-02T22:15:23+09:00
archives:
    - 2020
    - 2020-01
    - 2020-01-02
draft: false
---

android:接頭辞のつく属性はサポートライブラリのスタイルには使わない。

以上。

# 発生した問題

1. AlertDialogを使うときに、うっかり`androidx.appcompat.app.AlertDialog`と`android.app.AlertDialog`を混在させていた。

2. そのどちらでも共通のスタイルを使用していた。

3. アプリで使用するダイアログで下部ボタン（Positive, Negative, Neutral）の色がちゃんとスタイル適用されるものとされないものがあった。

# 原因

> 注: サポート ライブラリの属性名は、android: 接頭辞を使用しません。これは Android フレームワークの属性にのみ使用します。

[スタイルとテーマ | Android Developers](https://developer.android.com/guide/topics/ui/look-and-feel/themes.html#CustomizeTheme)

はい切腹しますありがとうございました。

# 修正

1. すべてのAlertDialogインポート部分を`androidx.appcompat.app.AlertDialog`に統一。

2. ボタンスタイルを次のように指定。
```xml
<item name="buttonBarPositiveButtonStyle">@style/PositiveButtonStyle</item>
<item name="buttonBarNegativeButtonStyle">@style/NegativeButtonStyle</item>
<item name="buttonBarNeutralButtonStyle">@style/NeutralButtonStyle</item>
```
