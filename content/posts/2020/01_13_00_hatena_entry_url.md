---
title: "はてブ公式がTwitterに共有しているコメントページURLがまた変わっていた件"
description: /entry/s/...だったり/entry/https://...だったりentry?url=...だったり
tags: ["memo", "hatena"]
date: 2020-01-13T13:43:46+09:00
lastmod: 2020-01-13T15:11:46+09:00
archives:
    - 2020
    - 2020-01
    - 2020-01-13
draft: false
---

Twitterをぼんやり眺めていたら、1/8頃から[はてブの公式アカウント](https://twitter.com/hatebu)が共有しているコメントページのURLが変わっていたことに気がついたのでメモ。

この記事を書いた直前に一応[Satena](https://play.google.com/store/apps/details?id=com.suihan74.satena)にも変更を反映してある。

{{<AndroidApp_Satena>}}

# 内容

今現在使用されているコメントページURLは次の3パターン存在することになる。  
パターン2,3は今のところ公式ページでは目にしておらず、SNSに投稿されるリンクに使用されている。

例としてGoogleのトップページ https://www.google.com/ に対するコメントページのURLを扱う。

## パターン1

https://b.hatena.ne.jp/entry/s/www.google.com/

要するに、https://b.hatena.ne.jp/entry/ 以下にhttpsなら/s/を加えてスキーム以外の部分を続かせればよい。  
はてブページ内ではこの方式が使用されている。

## パターン2

https://b.hatena.ne.jp/entry/https://www.google.com/

/entry/以下に対象ページのURLをそのまま突っ込む形。少し前からはてブ公式はこれをTwitterに投稿していた。  
パターン1にリダイレクトされる。

## パターン3

https://b.hatena.ne.jp/entry?url=https%3A%2F%2Fwww.google.com%2F

今回増えたやつ。クエリパラメータにエンコードした対象ページURLを指定している。  
他にも参照元がtwitterであるとかエントリのカテゴリが何であるかといったパラメータが付いているが別に付けなくてもページは開く。  
パラメータがないなら対象urlも別にエンコードしてなくても開く。

---

正直なんで最初からパターン3じゃないのか謎ではあるが、これに対応させてめでたしめでたし……と思ったら一点問題が。

## コメントページのコメントページはどれを使って取得すればいいのか

結論としては、こちら側で「エントリURL→コメントページURL」の変換をするときはパターン1を使うのが無難という感じ。  
2階より上に行きたい場合は今調べた限りではこれを使う他ない（？）

### ○: パターン1を再帰する

パターン1にパターン1  
https://b.hatena.ne.jp/entry/s/b.hatena.ne.jp/entry/s/www.google.com/

### ○: パターン3にパターン1のURLを渡す

パターン3にパターン1  
https://b.hatena.ne.jp/entry?url=https%3A%2F%2Fb.hatena.ne.jp%2Fentry%2Fs%2Fwww.google.com%2F

### ×: パターン1にパターン3のURLを渡す

パターン1にパターン3  
https://b.hatena.ne.jp/entry/s/b.hatena.ne.jp/entry?url=https%3A%2F%2Fwww.google.com%2F

### ×: パターン3にパターン3のURLを渡す

以下のどれも駄目

- パターン3のエンコードされていない部分をエンコード  
https://b.hatena.ne.jp/entry?url=https%3A%2F%2Fb.hatena.ne.jp%2Fentry%3Furl%3Dhttps%3A%2F%2Fwww.google.com%2F

- パターン3の全部分をエンコード  
https://b.hatena.ne.jp/entry?url=https%3A%2F%2Fb.hatena.ne.jp%2Fentry%3Furl%3Dhttps%253A%252F%252Fwww.google.com%252F

- エンコードしない  
https://b.hatena.ne.jp/entry?url=https://b.hatena.ne.jp/entry?url=https://www.google.com/

---

どうでもいいけど「パターン」がゲシュタルト崩壊。
