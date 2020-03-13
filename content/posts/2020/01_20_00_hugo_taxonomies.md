---
title: "Hugo - Taxonomyを使ってアーカイブページを作る"
description: "アーカイブページを静的構築する方法"
tags: ["hugo", "memo"]
date: 2020-01-20T02:25:00+09:00
lastmod: 2020-03-14T01:35:00+09:00
archives:
    - 2020
    - 2020-01
    - 2020-01-20
draft: false
---

## 追記 (2020/03/14 01:35)

アーカイブ用の日付指定部分のフォーマットを

```yml
archives:
    - 2020
    - 2020/01
    - 2020/01/20
```

のようにしていると、少なくともHugoのバージョン`v0.65.2`ではTaxonomiesを使った記事列挙時にひとつの記事につき`/archives/2020`,`/archives/2020/01`,`archives/2020/01/20`の三つ分が重複して列挙されてしまう問題が発生した。

そのため、次のように年月日アーカイブページが同じディレクトリに並列して配置されるように修正したらうまくいった。

```yml
archives:
    - 2020
    - 2020-01
    - 2020-01-20
```

これでアーカイブリストのURLは`/archives/2020`,`/archives/2020-01/`,`/archives/2020-01-20/`となる。

以上の内容について記事内容に修正を加えた。

---

## 参考

[静的サイトジェネレータHugoを使ったサイト構築（アーカイブ編） &middot; feedtailor Inc. スタッフブログ](http://staff.feedtailor.jp/2016/08/10/hugo_16/)

参考というかほぼそのまま全内容なのだが。。。

## 1. config.tomlに使用するTaxonomyを指定

```toml
[taxonomies]
  tag = "tags"
  archive = "archives"
```

`テンプレートファイル名 = "Taxonomyのキー名"`で指定する。

いまのところ当ブログではカテゴリーを使用していないので除外した。  
`tags`と`categories`はデフォルトでは有効になっているが、`[taxonomies]`を明示的に指定した上でそれらを記述しないと勝手に生成されなくなる。  
必要なら`category = "categories"`とか書いておく。

[Taxonomies | Hugo # Hugo Taxonomy Defaults](https://gohugo.io/content-management/taxonomies/#default-taxonomies)

## 2. レイアウトファイルを用意

>Taxonomy編 で説明したように、以下の順に検索されます。
>
>1. /layouts/taxonomy/SINGULAR.html
>2. /layouts/_default/taxonomy.html
>3. /layouts/_default/list.html
>4. /themes/THEME/layouts/taxonomy/SINGULAR.html
>5. /themes/THEME/layouts/_default/taxonomy.html
>6. /themes/THEME/layouts/_default/list.html
>
>前述の config.toml の設定では左辺が archive でしたので、上の 1 は /layout/taxonomy/archive.html となります。

<http://staff.feedtailor.jp/2016/08/10/hugo_16/>

## 3. 記事のフロントマターに項目を作成

### archetypes/default.md

```yaml
archives:
    - {{ now.Format "2006" }}
    - {{ now.Format "2006-01" }}
    - {{ now.Format "2006-01-02" }}
```

以上を記述しておくことで、`hugo new`した際に自動的にアーカイブ用の情報が挿入される。  
この記事の場合の出力結果↓

### posts/2020/01_20_00_hugo_taxonomies.md

```yaml
---
title: "Hugo - Taxonomyを使ってアーカイブページを作る"
description: "アーカイブページを静的構築する方法"
tags: ["hugo"]
date: 2020-01-20T02:25:00+09:00
lastmod: 2020-01-20T02:25:00+09:00
archives:
    - 2020
    - 2020-01
    - 2020-01-20
draft: false
---
```

これで[/archives/2020/](/archives/2020/)とか[/archives/2020-01/](/archives/2020-01/)とか[/archives/2020-01-20/](/archives/2020-01-20/)とかでアーカイブページを表示することができる。

やったぜ。

`archives`に指定する内容を変更するなど必要に応じて。

この方法で日毎の記事を毎回静的生成してたらいつかすごい数になりそう。(なればいいですね)

---

## 余談

Goはあまり触れていないので日時のフォーマットが慣れないなぁ気持ち悪いなぁと思っていたのだけれども、やっぱりこういうことだったのね。。。→  
[Goのtimeパッケージのリファレンスタイム（2006年1月2日）は何の日？ - Qiita](https://qiita.com/ruiu/items/5936b4c3bd6eb487c182)
