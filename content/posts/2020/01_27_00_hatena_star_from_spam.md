---
title: "Satenaで最近多いスパムからのスター通知を一部無視できるようにした"
description: ""
tags: ["Hatena", "Satena", "memo"]
date: 2020-01-27T00:42:28+09:00
lastmod: 2020-01-27T00:42:28+09:00
archives:
    - 2020
    - 2020/01
    - 2020/01/27
draft: false
---

[「存在しないブコメ」にはてなスターを付けるのはスパム行為なのか？](https://anond.hatelabo.jp/20200125213356)

この件。

最近モチベーション下がっていたこともあり「とりあえずネタひとつみっけ」ということで対応してみた。

<div class="appreach"><img src="https://lh3.googleusercontent.com/8s4Fzo7AmnoNOT-pbsRoBSYbmBFgfS98l0Qatr1-aHYCRUJlHwab6jB1rijGC1_FYA=s128" alt="Satena - はてなブックマーククライアント, はてブビューア" class="appreach__icon"><div class="appreach__detail"><p class="appreach__name">Satena - はてなブックマーククライアント, はてブビューア</p><p class="appreach__info"><span class="appreach__developper">すいはん</span><span class="appreach__price">無料</span><span class="appreach__posted">posted with<a href="https://mama-hack.com/app-reach/" title="アプリーチ" target="_blank" rel="nofollow">アプリーチ</a></span></p></div><div class="appreach__links"><a href="https://play.google.com/store/apps/details?id=com.suihan74.satena" rel="nofollow" class="appreach__gplink"><img src="https://nabettu.github.io/appreach/img/gplay_ja.png"></a></div></div>

## 「あなたへのお知らせ」を取得するAPI

はてなのページ右上にある「あなたへのお知らせ」は、ログイン情報を付加(cookieにrkを挿入する)して <https://www.hatena.ne.jp/notify/api/pull> をGETすることでJSONで取得することができる。

通常の中身がどんなものか詳細は省くとして、今回の件のスパムからきた通知は次のようになる。

```json
{
    "created": 1579831251,
    "object": [
        {
            "color": "yellow",
            "user": "bibibi260"
        }
    ],
    "verb": "star",
    "subject": "https://b.hatena.ne.jp/suihan74/2020124#bookmark-4680412377476457090",
    "metadata": {
        "subject_title": "はてなブックマーク - 2020124に関するsuihan74のブックマーク"
    },
    "modified": 1579831251,
    "user_name": "suihan74"
}
```

このスターが付けられたブクマは**無言ブックマーク**である。

冒頭の増田でも言及されているように、**無言ブクマの場合**に限り `subject_title` を正規表現なりで判定してやれば良さそう。ということで(雑に)やった。

```kt
package com.suihan74.satena.scenes.entries.notices

import com.suihan74.HatenaLib.Notice

private val spamRegex = Regex("""はてなブックマーク\s*-\s*\d+に関する.+のブックマーク""")

/** スパムからのスターの特徴に当てはまるか確認する */
fun Notice.checkFromSpam() =
    spamRegex.matches(this.metadata?.subjectTitle ?: "")

```

**コメント付き**の場合、`subject_title` 部分に自分のブコメが入る。（なお、入らない場合もある。はっきり検証していないが、最初に無言ブクマをして後からコメントを書き直した場合？あとで分かったら追記する）

そのため先述の方法では判別できないので、さてどうしたものかと思っている。

スパムユーザーはプロフィールとか見れば一発でそれと分かりそうな感じなのだけど、取得した通知分全員のプロフィールを取ってくるのも無駄な感じするしなあ。
