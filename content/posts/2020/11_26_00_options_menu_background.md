---
title: "オプションメニューの背景色を変更する"
description: "アクションバーのオプションメニューの背景色の変え方"
tags: ["Android", "MaterialComponents"]
date: 2020-11-26T21:43:04+09:00
lastmod: 2020-11-26T21:43:04+09:00
archives:
    - 2020
    - 2020-11
    - 2020-11-26
hide_overview: false
draft: false
---

## 問題と解決

「Androidアプリのスタイル指定というか、ここの色を変えるにはどの値を設定するっていうのものすごく分かりづらいし調べづらくない？？？」って割とよく思う。

今回ブチギレたのは  
`アクションバーのオプションメニューポップアップの背景色の変え方（少なくともMaterialComponentsで）`

設定する属性はこれ。

```xml
<resources>
    <style name="AppTheme" parent="Theme.MaterialComponents.Light">
        <!-- オプションメニューの背景色 -->
        <item name="colorSurface">@color/optionMenuBackground</item>
    </style>
</resources>
```

## 備考

「android option menu background」とかで検索すると次のスタイルを指定する解答がやけにヒットする。(古い情報)

```xml
<!-- オプションメニューの"各項目の"背景色 -->
<item name="android:itemBackground">@color/optionMenuBackground</item>
```

この方法でも確かに"メニュー項目の背景色は"変わるのだが、次のスクリーンショットのように上下の余白に意図しない色が残ってしまい完全ではない。

{{<img src="itemBackground.png" zoom=".5" title="androidd:itemBackgroundを設定">}}

また画像からは分からないのだが、タッチ時のアニメーションも再生されなくなる（多分上書きする前にデフォルトでセットされているのが`?selectableItemBackground`とかなんだと思う）

## 参考

[android - Remove top and bottom black borders from AutoCompleteTextView - Stack Overflow](https://stackoverflow.com/questions/61310493/remove-top-and-bottom-black-borders-from-autocompletetextview)
