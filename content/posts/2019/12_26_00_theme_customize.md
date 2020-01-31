---
title: "Hugoテーマのカスタマイズ箇所メモ"
description: このサイトのテーマの改造に関するメモ
tags: ["Hugo", "html"]
date: 2019-12-26T01:09:42+09:00
lastmod: 2020-02-01T01:40:00+09:00
archives:
    - 2019
    - 2019/12
    - 2019/12/26
draft: false
---

どこをどう変えたか、どうやって変えたか……etcを忘れそうなので急ぎメモ。

## ベーステーマ

[github-style](https://github.com/MeiK2333/github-style)

[サンプルページ](https://themes.gohugo.io//theme/github-style/)

GitHub風……というかCSSとか一部GitHubからそのまま持ってきてる感じのあるテーマ。

## 改修点

### 追記 (2020/02/01 01:40)

#### 1. CSSをキャッシュ避けするようにした

ブラウザにキャッシュされるとCSSの修正を伴う改修作業が著しく面倒なので、hugoコマンドでのサイト生成ごとにキャッシュ避けするクエリを追加するようにした。

[/layouts/partials/head.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/head.html)

```html {linenos=table, linenostart=8}
<link crossorigin="anonymous" media="all"
    rel="stylesheet" href="{{ printf "%s?%s" ("css/user.css" | absURL) (now.Format "20060102150405") }}"/>
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
