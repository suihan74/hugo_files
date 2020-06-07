---
title: "IntentChooserの項目をカスタムして強制的に表示させる一例"
description: "共有先アプリの選択肢をアプリ側で編集する方法"
tags: ["android", "kotlin", "intent"]
date: 2020-05-03T14:11:01+09:00
lastmod: 2020-05-03T14:11:01+09:00
archives:
    - 2020
    - 2020-05
    - 2020-05-03
hide_overview: false
draft: false
---

## 前提

Androidアプリでは`Intent`を使用してアプリ間で情報をやりとりできる。  
今回の記事では「URLを開く方法をユーザーに選ばせたい」というシナリオを前提とする。

この`Intent`だが、データ形式に対してデフォルトの共有方法をユーザーが設定することもできる。  
たとえば、「URLはデフォルトでChromeで開くようにする」「"https\://b.hatena.ne.jp/entry/"から始まるURL(以下「ブコメページのURL」と呼称する)の場合はデフォルトで[Satena](https://play.google.com/store/apps/details?id=com.suihan74.satena)で」という風に設定すると、毎回いちいち「どのアプリでこのURL開きますか？」を選択する必要はなくなるわけだ。

## この記事に書いたこと

以下二点を同時に満たす実装例。

- IntentChooserを強制的に表示する  
  (複数の選択肢がある場合、デフォルト設定の有無に関わらずchooserを表示する)

- IntentChooserに表示するアプリ項目を呼び出し元から操作する

## 何故そんなことしたいのか

[Satena](https://play.google.com/store/apps/details?id=com.suihan74.satena)ではURLに対して「ページを外部ブラウザで開く」というコマンドを用意しているのだが、これには当然ながら`Intent`を利用している。  
そして同時に「ブコメページのURLを[Satena](https://play.google.com/store/apps/details?id=com.suihan74.satena)で開く」ことができるようにしてある。(後述の[intent-filter例](#satenaのintent-filter例)参照)

さて、では[Satena](https://play.google.com/store/apps/details?id=com.suihan74.satena)アプリ内でブコメページのURLを「外部ブラウザで開く」した場合、どうなるだろう。  
「ブコメページのURLをSatenaで開く」がデフォルト動作として設定されている場合、Chromeなどの外部ブラウザが開かれることはなく、Satenaのブコメページ用Activityが新たに開かれてしまう。なんだか書いてあるのと挙動が違わね？ というようなことになる。

### Satenaのintent-filter例

#### AndroidManifest.xml

```xml
<intent-filter android:label="Satenaで見る">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="https" android:host="b.hatena.ne.jp" android:pathPrefix="/entry"/>
</intent-filter>
```

---

## コード

「ページを外部ブラウザで開く」の実装内容抜粋

```kt
val packageManager = context.packageManager
val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))

if (url.startsWith("https://b.hatena.ne.jp/entry/")) {
    // 強制的にchooserを開くための処理

    // ブコメページ以外のURLを使用することで「Satenaで開く」以外のURLを開く純粋な方法を収集する
    val dummyIntent = Intent(Intent.ACTION_VIEW, Uri.parse("https://dummy"))

    val intentActivities = packageManager.queryIntentActivities(dummyIntent, PackageManager.MATCH_ALL)
    val bookmarksActivities = packageManager.queryIntentActivities(intent, PackageManager.MATCH_ALL)

    // 生成するintentリストを弄ればchooserの項目を操作できる
    val intents = bookmarksActivities.plus(intentActivities)
        .distinctBy { it.activityInfo.name }
        .map { Intent(intent).apply { setPackage(it.activityInfo.packageName) } }

    check(intents.isNotEmpty()) { "cannot resolve intent for browsing the website: $url" }

    val chooser = Intent.createChooser(Intent(), "Choose a browser").apply {
        putExtra(Intent.EXTRA_INITIAL_INTENTS, intents.toTypedArray())
    }
    context.startActivity(chooser)
}
else {
    // それ以外のURLの場合はデフォルトで開く
    checkNotNull(intent.resolveActivity(packageManager)) { "cannot resolve intent for browsing the website: $url" }
    context.startActivity(intent)
}
```

### 参考

- [別のアプリにユーザーを送信する | Android デベロッパー | Android Developers](https://developer.android.com/training/basics/intents/sending?hl=ja)

- [Android でアプリから URL を強制的にブラウザで開く - Qiita](https://qiita.com/hota911/items/df669e2179b6a2342f48)
