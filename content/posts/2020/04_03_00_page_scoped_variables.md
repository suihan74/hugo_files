---
title: "Hugoの記事内で変数を使用できるようにする"
description: "各記事のMarkdown内で変数を利用できるようにするショートコードを作成する"
tags: ["Hugo", "shortcode", "markdown"]
date: 2020-04-03T12:13:58+09:00
lastmod: 2020-04-03T12:13:58+09:00
archives:
    - 2020
    - 2020-04
    - 2020-04-03
hide_overview: false
draft: false
---

## やりたいこと

当サイトは[Hugo](https://gohugo.io/)を使用して作成している。  
各記事はMarkdownで記述するのだが、その中でたとえば次のようにHugoのコードを書いたとしても当然それが実行されることはない。

```md
<!-- 以下はMarkdownで書いてもただの文字列として表示される -->

<!-- 変数をセットして -->
{{ $key := "value" }}

<!-- 取得したい -->
{{ $key }}
```

(実際にやってみたところ)

> {{ $key := "value" }}  
> {{ $key }}

しかし、たとえば[この記事](/posts/2019/06_10_00_suihan_twit_2)のように「更新のたびに記事内のすべての最新バージョン文字列を書き換えなければならない」といった需要があるときに、コード使えれば楽なのになぁといった気持ちになる。

そこで、ページ内でだけ利用できる変数を扱うショートコードを用意した。

## Shortcodes

### layouts/shortcodes/set.html

```md
<!-- set key value で変数をセット -->
{{- .Page.Scratch.Set (index .Params 0) (index .Params 1) -}}
```

### layouts/shortcodes/get.html

```md
<!-- get key で変数を文字列として取得 -->
{{- .Page.Scratch.Get (index .Params 0) -}}
```

それぞれのコードを`{{ ~~~ }}`ではなく`{{- ~~~ -}}`で囲っているのは、hugoの実行パラメータに`--minify`が指定されていない場合に置き換えた文字列前後に余分な空白ができるためである。パラメータが指定されている場合は別にどちらでもとくに問題ないはず。  
(執筆中のリアルタイム確認ではminifyしていないので少し気になっただけ)

以上ふたつのファイルを配置することで、各記事Markdown内で次のように変数が利用できるようになる。

```md
<!-- 変数をセットする -->
{{</*set key value*/>}}

<!-- 変数を取得(文字列として表示)する -->
変数keyは「{{</*get key*/>}}」です。
```

(実際にやってみたところ)

{{<set key value>}}
>変数keyは「{{<get key>}}」です。

やったぜ。

既にテンプレートなどでページ内のScratchを使用していて名前衝突が心配になる場合は、上記ショートコードのキー指定部分にユニークなprefixなりsuffixなりを付け加えるようにすれば良いのだと思う。

---

## 余談

ショートコードを実行しないで記事中に文字列として表示したい場合は、次のように書くらしい。今この記事を書いていて知った。

```md
{{</*/*hoge*/*/>}}
```

### 参考

[How is the Hugo Doc site showing shortcodes in code blocks? - HUGO](https://discourse.gohugo.io/t/how-is-the-hugo-doc-site-showing-shortcodes-in-code-blocks/9074)
