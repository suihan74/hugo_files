---
title: "WebViewで画像リンクを長押ししたときにもリンク先URLを取得する方法"
description: "hitTestResultを使用するだけだと画像URLしか得られないので、ひと手間必要というお話"
tags: ["Android", "kotlin", "WebView"]
date: 2020-10-26T20:14:44+09:00
lastmod: 2020-10-26T20:14:44+09:00
archives:
    - 2020
    - 2020-10
    - 2020-10-26
hide_overview: false
draft: false
---

## 前提

Androidの```WebView```では、次のようにして表示しているページのリンク部分や画像を長押ししたときの処理を任意に設定できる。

```kt
webView.setOnLongClickListner {
    val hitTestResult = webView.hitTestResult
    when (hitTestResult.type) {
        WebView.HitTestResult.SRC_ANCHOR_TYPE -> {
            // リンクテキストの場合
            val linkUrl: String? = hitTestResult.extra
        }

        WebView.HitTestResult.IMAGE_TYPE -> {
            // 画像の場合
            val imageUrl: String? = hitTestResult.extra
        }

        WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE -> {
            // 画像リンクの場合
            val imageUrl: String? = hitTestResult.extra
        }
    }
}
```

上記コードにおいて、変数```linkUrl```は「リンク先のURL」、```imageUrl```は「長押しした対象画像のURL」を指す。  
ここでは値が```nullable```であることを明示するため、一応変数型を明記している。

「リンクテキストの場合」「画像の場合」はとくに問題なく自然な感じにURLが扱えるのだが、「画像リンクの場合」は```hitTestResult```から得られる情報だけではリンク先URLが得られない。

## 画像リンクのリンク先アドレスを取得する

そこで、次のようにしてフォーカスしているリンクの部分の情報を得ることができるので、これを使ってリンク先URLを取得する。

```kt
webView.setOnLongClickListner {
    val hitTestResult = webView.hitTestResult
    when (hitTestResult.type) {
        // 他省略

        WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE -> {
            // 画像リンクの場合
            val message = Handler().obtainMessage()
            wv.requestFocusNodeHref(message)

            val linkUrl: String? = message.data.getString("url")

            // hitTestResultから得ればいいが一応
            val imageUrl: String? = message.data.getString("src")
        }
    }
}
```

これでリンク先URL、画像URL両方が取得できた。

## 余談

なお、上記コード中の```message.data```からはここでは次の三つの情報が得られる。

```kt
// リンク先URL
message.data.getString("url")

// 画像URL(画像リンクの場合。リンクテキストではnullになる)
message.data.getString("src")

// リンク部分の文字列(リンクテキストの場合。画像リンクではnullになる)
message.data.getString("title")
```
