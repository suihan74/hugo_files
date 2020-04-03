---
title: "HugoでTwitterの投稿を引用表示するために必要な設定"
description: そのままTwitterの引用コードをコピーしても表示できないっぽいという話
tags: ["hugo"]
date: 2019-12-29T03:05:26+09:00
lastmod: 2020-04-03T12:45:00+09:00
archives:
    - 2019
    - 2019-12
    - 2019-12-29
hide_overview: true
draft: false
---

# 追記 (2020-04-03)

ショートコードを使用すればこの記事の内容を行う必要はない。申し訳ない。

```md
{{</*tweet 1210955996861882370*/>}}
```

↓

{{<tweet 1210955996861882370>}}

## 参考

[Shortcodes | Hugo](https://gohugo.io/content-management/shortcodes/#tweet)

---

# 必要な設定

https://gohugo.io/news/0.60.0-relnotes/

`config.toml` に以下を追記する必要がある。

```toml
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
```

# 記事

その上で記事マークダウンの該当箇所では、Twitter公式の「ツイートを埋め込む」から得られた次のようなコードを記述する。

```html
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">この十年で良かったアニメ10選書こうとして、10個に絞るのも難しいし何度も観て好きなはずなのに感想らしい感想が出てこないのも多くてなんというか何</p>&mdash; すいはん (@suihan742) <a href="https://twitter.com/suihan742/status/1210955996861882370?ref_src=twsrc%5Etfw">December 28, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
```

# 結果
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">この十年で良かったアニメ10選書こうとして、10個に絞るのも難しいし何度も観て好きなはずなのに感想らしい感想が出てこないのも多くてなんというか何</p>&mdash; すいはん (@suihan742) <a href="https://twitter.com/suihan742/status/1210955996861882370?ref_src=twsrc%5Etfw">December 28, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
