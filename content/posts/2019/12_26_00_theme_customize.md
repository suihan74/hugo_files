---
title: "Hugoテーマのカスタマイズ箇所メモ"
description: このサイトのテーマの改造に関するメモ
tags: ["Hugo", "html"]
date: 2019-12-26T01:09:42+09:00
archives:
    - 2019
    - 2019-12
    - 2019-12-26
hide_overview: true
draft: false
---

どこをどう変えたか、どうやって変えたか……etcを忘れそうなので変更の都度メモ。  
記事の下にいくほど古く、現在では変更してしまった内容も含まれます。

## テーマ

- 適用しているテーマ  
  [suihan74/github-style](https://github.com/suihan74/github-style)

- 改造前のテーマ  
  - [MeiK2333/github-style](https://github.com/MeiK2333/github-style)

  - [サンプルページ](https://themes.gohugo.io//theme/github-style/)

マージ作業が面倒なので`MeiK2333/github-style`の最新の状態はほぼ追従していません。ので、本家ではとっくに直っている問題がそのままだったり、現時点では内部的にはだいぶ別物になっているかも。

---

### 追記 (2021-02-16)

#### 関連記事を表示するようにした

各投稿ページ下部(記事本文とコメント投稿欄の間)に関連(と思われる)記事を表示するようにした。

[43a8a04](https://github.com/suihan74/github-style/commit/43a8a04b11b55bef8b9fb92032a63a90ff34b010)

---

### 追記 (2020-12-20 20:30)

#### シンタックスハイライトにタイトルを挿入できるようにした

[HugoのコードブロックにQiitaのようなTitleをつける | AABrain](https://aakira.app/blog/2018/12/code-block-title/)

参考……というかほぼそのまま使わせてもらいました。  
(タイトル部分のスタイルは適当に修正した > [dbc6945](https://github.com/suihan74/github-style/blob/dbc694577b234ecbbf7c59ab71e2e59aee0b8abc/static/css/syntax-highlight.css#L84))

```kt:example.kt
/** こんな感じ */
fun main(args : Array<String>) {
    println("hello world")
}
```

````md:タイトル付きシンタックスハイライトのマークダウンでの書き方.md
```kt:title.kt
~~~
```
````

のように書くと、`言語:タイトル`のタイトルの部分が表示される。タイトル部分には言語ごとの正しい拡張子が必須っぽい。

#### タグ部分の表示を`MeiK2333/github-style`のものに追従した

[modify styles for tags and timestamp · suihan74/github-style@7e90814](https://github.com/suihan74/github-style/commit/7e908145e43f3e4bd7182a0b0902e2a3da1fb4bc)

ついでにトップページ・投稿一覧ページでのタイムスタンプの相対表記やめた。

画面幅が狭い場合にタグ文字列の途中で改行されないように`display: inline-block`を指定した。

#### 空白を含むタグのリンクが誤っていたのを修正した

[fix tags url · suihan74/github-style@66ebceb](https://github.com/suihan74/github-style/commit/66ebcebfb7b35beebf882acf85ab5f3c6da71bf2)

文字列をURL用に変換するための関数`urlize`を使用し忘れていたので、タグ文字列に空白が含まれる場合に誤ったリンクが生成されていたので修正した。

---

### 追記 (2020-12-01 22:20)

#### 月別アクティビティが年を考慮していない問題を修正

[contribution not properly displayed for past years #45](https://github.com/MeiK2333/github-style/issues/45)

#### グラフタイルをクリックしたときに表示されるアクティビティが年を考慮していない問題を修正

[fix generating contributions · suihan74/github-style@26ee9ff](https://github.com/suihan74/github-style/commit/26ee9ff5e8530b5cf7fb1de77f168ff29f1fa277)

[modify not to use yearStr · suihan74/github-style@08ad04](https://github.com/suihan74/github-style/commit/08ad04beeddb06517b22741d4811d4e98b90f26d)

タイルをクリックしたときの挙動自体がこちらで勝手につけた機能なので、好きに直すなど。  
年を表す変数だけ文字列として扱われていたので、これも数値に直した。

---

### 追記 (2020/11/13 18:00)

#### 画像をポップアップするjQueryプラグイン導入した

[Lightbox2](https://lokeshdhakar.com/projects/lightbox2/)

今まで同タブで単純に画像を開いていたのだが、アレな感じがしたのでページ内でポップアップするようにした。

ショートコード`img`を書き換えた。  
(つまりこれを使用していない古い記事ではポップアップされずにページ遷移する)

---

### 追記 (2020/08/01 19:00)

#### fork元マージした

今までは最初にforkして以来勝手にやっていたのだが、  
fork元の方でグラフとかタイムラインとかがいい感じに実装されていたので、頂戴した。感謝。

貢献しないのに何故forkしたという話ではあるが、改造していると思いのほか「自分にとってはこの変更でいいのだが、プルリク投げて「おれのやつの方がいいだろ！！１」って他所まで影響与えるような改修ではないな……」みたいな気持ちになることが多かったのだよな。  
今回は幾つか直せるところ見つけたから送ってみたけど。

#### 幾つかの簡単な修正をPullRequest投げた

- 画面が狭い場合、タイムラインに表示される項目の日付部分が下方向にズレる問題を修正  
  [fix an issue with position of activity items · suihan74/github-style@f146b07](https://github.com/suihan74/github-style/commit/f146b079d38864942d727230c8d6852dc52b601a)

- 進捗グラフにマウスオーバーしたときに表示されるメッセージ中の'contribution'を'post'に置き換えた  
  [replace 'contribution' with 'post' on hover messages · suihan74/github-style@dc55047](https://github.com/suihan74/github-style/commit/dc5504790b61e055cf59e77edd828dca7bd4c660)

- 名詞の複数形への対応 ('1 post', '2 posts'...など)  
  [fix to consider plural nouns · suihan74/github-style@9ac05a2](https://github.com/suihan74/github-style/commit/9ac05a2a74d6e777b1be482dd1a082fba96015a6)

#### 「Show more activity」ボタンを追加した

最初から表示されている情報は直近2ヶ月分だけにするようにした。

「Show more activity」ボタンを押すと、ボタンを隠して残りの全項目を表示するようにした。  
GitHubでは「押すたびにちょっとずつ増える」だが、作る側も見る側も面倒そうなので再現しないことにした。

#### 進捗グラフ(草のやつ)のタイルを押した時の挙動を作成した

タイルを押すと、そのタイルが属する月のアクティビティだけをタイムラインに表示するようにした。  
これもGitHubそのままというわけにはいかなかったが、まぁこれで十分でしょうという感じもする。

[add 'show more activity' button and behavior of selecting a rect · suihan74/github-style@4c4de3a](https://github.com/suihan74/github-style/commit/4c4de3a86d5a49a7390ddb258163207e884f7818)

#### 記事タイトルの前に著者名を表示しないようにした

すいはんしかいませんしおすし。  
文責ということならどこかに書いておいた方がいいかもしれないが。

#### スタイルシートを最近のGitHubのものに追従した

全体的に角が丸くなった。  
この作業において、何点か修正が必要だった。

[replace stylesheets with newer · suihan74/github-style@81b909e](https://github.com/suihan74/github-style/commit/81b909eb27df914a0e71944bc761f45d2d44b669)

---

### 追記 (2020/03/06 18:00)

#### [/layouts/partials/user-profile.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/user-profile.html)に関する変更

1. 「follow」ボタンを削除した

2. 利用しているサービス一覧のタイトルを`"Organizations"`から`"Services"`に変更した

3. スマホなど幅狭画面で簡略化されるプロフィール部分にサービス一覧のアイコンを表示するようにした  
    -> [remove 'follow/unfollow' button and modify to show service icons when the page width is narrow](https://github.com/suihan74/github-style/commit/cf6e33882a70761aae72b9b4473875272256e3c1#diff-a93db67ae2131954ac70d2f368ff0adf)

4. mastodonとはてブのアイコンの解像度を上げた

#### ほか

- 各記事画面のコメント機能を有効にした  
    (素のテーマに元々あった機能。GitHubのIssueに書き込まれる ※コメント付けたい人はGitHubアカウントが必要)  
    -> [utterances](https://utteranc.es/)

- トップページの草(一番下にある記事更新頻度を表示しているやつ)が幅狭画面でスクロールバーが表示されるとき、初期スクロール位置が再右端になるようにした  
    [/layouts/partials/calendar.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/calendar.html)

    ```js {linenos=table, linenostart=133}
    window.onload = function () {
        const elem = document.getElementById("graph-container")
        elem.scrollLeft = 880
    }
    ```

---

### 追記 (2020/02/22 15:15)

見出しの横にアンカーを表示するようにした。

![変更点SS8](/images/2019/12_26_00_08.png "変更点 - 見出しアンカー")

マークダウン→HTMLタグの変換時にフックする方法ないのかと思って調べてたら「生成後の文字列を正規表現で置き換える」やり方を見つけたので利用した。

[/layouts/partials/post.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/post.html)

```vue {linenos=true, linenostart=75}
<article class="markdown-body entry-content p-3 p-md-6" itemprop="text">
    {{- .Content | replaceRE "(<h[1-9] id=\"([^\"]+)\".+)(</h[1-9]+>)" "${1}<a class=\"anchor\" aria-hidden=\"true\" href=\"#${2}\"><svg class=\"octicon octicon-link\" viewBox=\"0 0 16 16\" version=\"1.1\" width=\"16\" height=\"16\" aria-hidden=\"true\"><path fill-rule=\"evenodd\" d=\"--省略--\"></path></svg></a> ${3}" | safeHTML -}}
</article>
```

参考  
[Hugo で markdown の hタグに自動で anchorタグをつける - n/a n/a nao](https://blog.nananao.com/entry/hugo-htag-add-atag)

---

### 追記 (2020/02/03 20:55)

#### 1. ヘッダ・フッタのocticonを変更した

<svg height="32" viewBox="0 0 16 16" version="1.1" width="32">
    <path style="fill-rule:evenodd" d="M 8,0 C 3.58,0 0,3.58 0,8 c 0,3.54 2.29,6.53 5.47,7.59 0.4,0.07 0.55,-0.17 0.55,-0.38 0,-0.19 -0.077797,-1.481017 -0.077797,-2.151017 C 5.9433346,12.597005 6.1160655,11.824118 6.56,11.53 4.78,11.33 3.8013559,10.978983 3.5810169,7.6647458 3.9355372,4.6446549 5.3653277,3.8240736 8.0538984,3.639661 10.967753,3.8102868 12.141357,4.4459308 12.425085,7.6477967 12.255593,10.378813 11.25,11.33 9.47,11.53 c 0.29,0.25 0.54,0.73 0.54,1.48 0,1.07 -0.01,1.93 -0.01,2.2 0,0.21 0.15,0.46 0.55,0.38 C 13.806446,14.490656 15.999121,11.437004 16,8 16,3.58 12.42,0 8,0 Z" />
    <rect style="fill-rule:evenodd" ry="0.23728813" y="6.3898306" x="6.0169492" height="1.0847455" width="3.9830508"/>
    <!--
    <path fill-rule="evenodd"
        d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z" />
    -->
</svg>

SVGの中身をhtmlファイルに直書きしないとスタイルが適用できないのがなんかなぁという感じはする。

#### 2. faviconを設定した

1. 16×16, 32×32 のpng画像を用意

2. GIMPで32×32の方を開いて16×16の方をレイヤーとして追加 → ico形式でエクスポート

3. `static/favicon.ico`として配置

4. `partial/head.html`を編集

    ```html {linenos=table, linenostart=30}
    <link rel="icon" type="image/x-icon" class="js-site-favicon" href='{{ "/favicon.ico" | absURL }}'>
    <link rel="shortcut icon" href='{{ "/favicon.ico" | absURL }}'/>
    ```

---

### 追記 (2020/02/01 02:40)

#### 1. CSSをキャッシュ避けするようにした

ブラウザにキャッシュされるとCSSの修正を伴う改修作業が著しく面倒なので、hugoコマンドでのサイト生成ごとにキャッシュ避けするクエリを追加するようにした。

[/layouts/partials/head.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/head.html)

```html {linenos=table, linenostart=8}
<link crossorigin="anonymous" media="all"
    rel="stylesheet" href='{{ printf "%s?%s" ("css/user.css" | absURL) (now.Format "20060102150405") }}'/>
```

#### 2. サイト内検索をヘッダ部分に追加

![変更点SS7](/images/2019/12_26_00_07.png "変更点 - 検索ボックス")

Googleカスタム検索を利用したサイト内検索を追加。

いまのところ一定以上の画面幅でのみヘッダ部分右側に検索ボックスが表示されるようになっている。

テンプレート該当箇所は以下。入力ボックスは「1文字以上を入力状態でEnter」でsubmitされるようにしてある。(`onkeydown="~~~"`部分)

こいつを使うには[Googleカスタム検索](https://cse.google.com/)にサイトを登録して、`cx`を取得する必要がある。cxは`config.toml`のパラメータ`googleCSE`に設定するようにした。この値を指定すると以下のテンプレート部分が有効になりサイトに表示される。

[/layouts/partials/header.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/header.html)

```html {linenos=table, linenostart=52}
<!-- Google Custom Search -->
{{- with $.Site.Params.googleCSE }}
<div class="Header-item position-relative mr-0 d-none d-lg-flex Details-content--hidden">
    <div class="header-search flex-self-stretch flex-lg-self-auto mr-0 mr-lg-3 mb-3 mb-lg-0 position-relative js-jump-to">
        <div class="position-relative">
            <form
                class="js-site-search-form"
                role="search"
                action="https://cse.google.com/cse"
                method="get">
                <input type="hidden" name="cx" value="{{ . }}" />
                <label class="form-control input-sm header-search-wrapper p-0 header-search-wrapper-jump-to position-relative d-flex flex-justify-between flex-items-center js-chromeless-input-container">
                    <input
                        type="text"
                        class="form-control input-sm header-search-input"
                        name="q"
                        value=""
                        placeholder="Search"
                        autocapitalize="off"
                        aria-label="Search"
                        spellcheck="false"
                        onkeydown="if(event.keyCode==13){if(this.value.length){document.getElementById('header-search-submit').click();return false}else{return false}};"
                        autocomplete="off">
                    <button id="header-search-submit" class="mr-1 ml-1 header-search-button" type="submit">
                        <svg class="header-search-button-icon" width="13" height="13" viewBox="0 0 13 13">
                            <title>サイト内検索</title>
                            <path d="~~~省略~~~"></path>
                        </svg>
                    </button>
                </label>
            </form>
        </div>
    </div>
</div>
{{- end }}
```

#### 3. 「Archives」をメニューに追加

日付ごとの記事リストを(草を使わないでメニューからも)表示できるようにした。

また、メニュー部分を別ファイルに分けた。

[/layouts/partials/menu.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/menu.html)

```html
<!-- ページ上部メニュー部分 -->
<div class="UnderlineNav width-full user-profile-nav js-sticky top-0">
    <nav class="UnderlineNav-body">
        <a class='UnderlineNav-item mr-0 mr-md-1 mr-lg-3 {{ if eq .Path "" }}selected{{ end }}' href="{{ .Site.BaseURL }}">
            Overview
        </a>
        <a class='UnderlineNav-item mr-0 mr-md-1 mr-lg-3 {{ if hasPrefix .Path "posts" }}selected{{ end }}' href='{{ absURL "posts/" }}'>
            Posts
            <span class="Counter hide-lg hide-md hide-sm">
                {{- $mainSections := .Site.Params.mainSections | default (slice "post") }}
                {{- $section := where .Site.RegularPages "Section" "in" $mainSections }}
                {{- len $section }}
            </span>
        </a>
        <a class='UnderlineNav-item mr-0 mr-md-1 mr-lg-3 {{ if hasPrefix .Path "tags" }}selected{{ end }}' href='{{ absURL "tags/" }}'>
            Tags
            <span class="Counter hide-lg hide-md hide-sm">
                {{ len .Site.Taxonomies.tags }}
            </span>
        </a>
        <a class='UnderlineNav-item mr-0 mr-md-1 mr-lg-3 {{ if hasPrefix .Path "archives" }}selected{{ end }}' href='{{ absURL "archives/" }}'>
            Archives
        </a>
        <a class='UnderlineNav-item mr-0 mr-md-1 mr-lg-3 {{ if eq .Path "about.md" }}selected{{ end }}' href='{{ absURL "about/" }}'>
            About
        </a>
    </nav>
</div>
```

---

### 追記 (2020/01/16 05:00)

#### トップページに草生やした

![変更点SS6](/images/2019/12_26_00_06.png "変更点SS6 - 草")

GitHubで何か活動した日には草が生えるやつ。  
ここではとりあえず「記事を新規投稿したらcount+=1」するようにした。

動的なことはやりたくないので、この草も頑張ってHugoで生成している。(ので、まぁちょっとアレな感じではある)

■をクリックするとその日に投稿した記事一覧画面に遷移するようにした。  
この画面は一年分の記事を表示する画面をHugoで生成しておいて、表示時に要らない部分をjavascriptで消している。paginatorとの兼ね合いとかに問題がありそうなのでなんとかしたい。  
余裕があれば日毎の記事一覧もなんかこういい感じにトップ画面に表示できるようにしたい。

---

### 追記 (2020/01/15 04:20)

#### 1. GitHubのファイル変更履歴へのリンクを追加

![変更点SS4](/images/2019/12_26_00_04.png "変更点SS4 - History")

[/layouts/partials/post.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/post.html)

```html {linenos=table, linenostart=64}
<div class="d-flex py-1 py-md-0 flex-auto flex-order-1 flex-md-order-2 flex-sm-grow-0 flex-justify-between">
    <div class="BtnGroup">
    {{ $historyUrl := add (substr (printf "https://github.com/%s/hugo_files/commits/master/content%s" .Site.Params.github .RelPermalink) 0 -1) ".md" }}
    <a rel="nofollow" class="btn btn-sm BtnGroup-item" href="{{ $historyUrl }}">History</a>
    </div>
</div>
```

ファイルパスの取得に`.File.Path`を使用すると、ディレクトリの区切り文字がエスケープされてしまってどうしようもなかった。  
`.Permalink`、`.RelPermalink`を使用する場合は何故かエスケープは回避されるようなのでこれで`/posts/2019/hoge/`みたいな文字列を取得し、最後の`/`を削って`".md"`をくっ付ける力技で(無理矢理)解決。

#### 2. `post.html`の「投稿日時」「更新日時」を絶対時間で表示するように変更

![変更点SS5](/images/2019/12_26_00_05.png "変更点SS5 - 絶対時間に変更")

ついでに「更新日時」は「投稿日時」と異なる場合のみ表示するようにした。

[/layouts/partials/post.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/post.html)

```html {linenos=table, linenostart=37}
<div class="d-block text-small text-gray">
    Created at <time datetime="{{ .PublishDate.Format "2006-01-02 15:04" }}" class="no-wrap">
        {{ .PublishDate.Format "2006/01/02 15:04" }}</time>
{{ if ne .PublishDate .Lastmod }}
    <span class="file-info-divider"></span>
    Updated at <time datetime="{{ .Lastmod.Format "2006-01-02 15:04" }}" class="no-wrap">
        {{ .Lastmod.Format "2006/01/02 15:04" }}</time>
{{ end }}
</div>
```

---

### 追記 (2019/12/26 04:50)

タグ一覧画面を追加。

![変更点SS3](/images/2019/12_26_00_03.png "変更点SS3 - タグ一覧画面")

- [/layouts/_default/terms.html](https://github.com/suihan74/github-style/blob/master/layouts/_default/terms.html) を追加。
- [/layouts/partials/tags.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/tags.html) を追加。terms.htmlのコンテンツ部分。posts.htmlをベースに作成。
- 他の全ての画面（overview, posts, about）のメニュー部分にタグ一覧画面へのリンクを作成。  

```html
<a class="UnderlineNav-item mr-0 mr-md-1 mr-lg-3" href="{{ absURL "tags/" }}">
    Tags
    <span class="Counter hide-lg hide-md hide-sm">
        {{ len .Site.Taxonomies.tags }}
    </span>
</a>
```

---

### SS

![変更点SS1](/images/2019/12_26_00_01.png "変更点SS1 - Overview画面(大)")

![変更点SS2](/images/2019/12_26_00_02.png "変更点SS2 - Post画面 + 画面(小)")

### 詳細

#### 1. サイトタイトルを追加

[layout/layouts/partials/header.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/header.html)

```html
<header class="Header js-details-container Details flex-wrap flex-lg-nowrap p-responsive" role="banner">
    <div class="Header-item d-none d-lg-flex">
        <a class="Header-link" href="{{ .Site.BaseURL }}" aria-label="Homepage" data-ga-click="Header, go to dashboard, icon:logo">
            <svg class="octicon octicon-mark-github v-align-middle" height="32" viewBox="0 0 16 16" version="1.1"
                width="32" aria-hidden="true">
                <path fill-rule="evenodd" d="略" />
            </svg>
            <!-- ここ -->
            <span style="margin-left: 8px;">
                {{ .Site.Title }}
            </span>
        </a>
    </div>
    <!-- 以下略 -->
</header>
```

`style` の部分はなんとかした方がいいのかもしれない。

---

#### 2. Authorアイコンをクリックしても何も起きないように変更

[layout/layouts/partials/user-profile.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/user-profile.html)

クリックで画像単体で表示できても特に面白いことはないので。

---

#### 3. 利用中アカウントへのショートカットを追加

MastodonとHatenaを（割と無意味に）追加。

---

#### 4. 小プロフィールアイコンを削除

押しても何も起きなかったので。

利用中アカウント表示をこっちにも追加してもいいかもしれない。

---

#### 5. 記事本文の冒頭ではなく概要を設定し表示するように変更

[layout/layouts/partials/overview.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/overview.html)

```html
<div class="text-gray text-small d-block mt-2 mb-3">
    {{ .Description | safeHTML }}
</div>
```

[layout/layouts/partials/posts.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/posts.html)

```html
<div name="posts-post">
    {{ .Description | safeHTML }}
</div>
```

記事markdownのメタデータに`description`を設定するとこれが表示されるように変更。

[hugo_files/archetypes/default.md](https://github.com/suihan74/hugo_files/blob/master/archetypes/default.md) を設定することで
`$ hugo new posts/hoge.md` した時にdefault.mdに設定した内容が新しい記事に挿入される。

ちなみにこの記事の場合はこれ。

```yaml
---
title: "Hugoテーマのカスタマイズ箇所メモ"
description: このサイトのテーマの改造に関するメモ
tags: ["Hugo", "html"]
date: 2019-12-26T01:09:42+09:00
lastmod: 2019-12-26T01:09:42+09:00
draft: false
---
```

---

#### 6. 記事に設定したタグを表示

[layout/layouts/partials/overview.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/overview.html)

```html
{{ with .Params.tags }}
    <span class="f6 text-gray mt-1">
        <svg class="octicon octicon-tag" viewBox="0 0 14 16" version="1.1" width="14" height="16" aria-hidden="true">
            <path fill-rule="evenodd" d="略" />
        </svg>
        {{ range . }} <a href="/tags/{{ lower . }}/">{{ . }}</a>{{ end }}
    </span>
{{ end }}
```

[layout/layouts/partials/posts.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/posts.html) もほぼ同様。

```hugo
{{ with HOGE }}
~~~
{{ end }}
```
でHOGEが存在する場合に生成されるhtmlファイルに挿入される。

`{{ . }}` は`with`なり`range`なりで現在参照されている項目を表す。

`{{ lower STR }}` でASCII文字列を小文字に変換する  
（タグページのURLは設定したタグの小文字になる（英数字の場合））

---

#### 7. 記事の更新時間の表示を変更・修正

[layout/layouts/partials/overview.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/overview.html)

```html
<div class="mt-1">
    Updated <relative-time datetime="{{ .Lastmod.Format "2006-01-02 15:04" }}" class="no-wrap">
        {{ .Lastmod.Format "2006-01-02 15:04" }}</relative-time>
</div>
```

日付だけでなく時分も表示するように変更。

---

#### 8. 改造後のテーマのリポジトリへのリンクを追加

はい。

---

#### 9. 小幅画面にもサイトタイトルを追加

はい。

---

#### 10. 小幅画面でも記事ヘッダ部分を省略しないように変更

[layout/layouts/partials/post.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/post.html)

```html
<div class="application-main " data-commit-hovercards-enabled="">
    <div itemscope="" itemtype="http://schema.org/SoftwareSourceCode" class="">
        <main id="js-repo-pjax-container" data-pjax-container="">
            <div class="pagehead repohead instapaper_ignore readability-menu experiment-repo-nav pt-lg-4 "> <!-- pt-0を除去; 小画面でもタイトル表示するため -->
                <div class="repohead-details-container clearfix container-lg p-responsive d-lg-block"> <!-- d-noneを除去 -->
                    <div class="mb-3 d-flex">
                        ~~~ 表示内容 ~~~
                    </div>
                </div>
            </div>
        </main>
    </div>
</div>
```

クラスpt-0を除去することで小幅画面で記事タイトル部分のtopマージンが無くなるのを回避。  
d-noneを除去することで小幅画面でも記事ヘッダスペースを表示し続けるようになった。

- 著者名をクリックしたときにトップページに飛ばされていたのを、aboutページに遷移するように変更
- 記事タイトルを改行許可して省略しないように変更 -> [e696b5a](https://github.com/suihan74/github-style/commit/e696b5a22a3a79d81f0ef0d1045aed6ae9ace20e#diff-0c10dbee20afbafb8202ba360b55546a)
- 他画面同様descriptionとタグを追加
- 誤字:ModifyedをModifiedに変更……した後にUpdatedに変更（それほど意味はない）
- 時刻のフォーマットを他画面同様に変更

---

#### 11. `<head>`部分

[layout/layouts/partials/head.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/head.html)

- OGPタグを追加 -> [f0be7ad](https://github.com/suihan74/github-style/commit/f0be7adcce5203d76a44409bf41f7ad45a3baa3e#diff-35755203408c34159ac6094e42351391)  
同様にしてTwitterカードなども追加可能だが、別にこれだけでよくね？みたいにはなっている。  
.IsHome == trueのときとは、要するに https://suihan74.github.io/ （トップページ）が表示されているとき。それ以外を記事扱いでいいのかみたいな感じはする。  
あとは記事ごとに画像やらを追加したい場合はelse側にも`og:image`を追加して、記事のメタデータに`image: "url"`とか設定するようにしておけばいい（何故しない）

```html
    <!-- OGP -->
    <meta property="og:url" content="{{ .URL | absURL }}"/>
    <meta property="og:site_name" content="{{ .Site.Title }}"/>
    {{ if .IsHome }}
    <meta property="og:type" content="website" />
    <meta property="og:title" content="{{ .Site.Title }}" />
    <meta property="og:description" content="{{ .Site.Params.Description }}"/>
    <meta property="og:image" content="{{ $.Site.Params.avatar }}"/>
    {{ else }}
    <meta property="og:type" content="article" />
    <meta property="og:title" content="{{ .Title }}" />
    <meta property="og:description" content="{{ .Params.Description }}"/>
    {{ end }}
```

- description, keywordsを追加 → [2fdf2bc](https://github.com/suihan74/github-style/commit/2fdf2bc1fb62f03f6b1f2ecad71b0901d51093a0#diff-35755203408c34159ac6094e42351391)  
記事markdownのメタデータに`keywords: "~~~"`を追加すると出力したhtmlにも追加される。  
なおkeywordsメタタグは現在ではSEO的には意味がない模様（じゃあ何故追加した）
