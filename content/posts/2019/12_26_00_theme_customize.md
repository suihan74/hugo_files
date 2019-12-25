---
title: "Hugoテーマのカスタマイズ箇所メモ"
description: このサイトのテーマの改造に関するメモ
tags: ["Hugo", "html"]
date: 2019-12-26T01:09:42+09:00
lastmod: 2019-12-26T01:09:42+09:00
draft: false
---

どこをどう変えたか、どうやって変えたか……etcを忘れそうなので急ぎメモ。

# ベーステーマ

[github-style](https://github.com/MeiK2333/github-style)

[サンプルページ](https://themes.gohugo.io//theme/github-style/)

GitHub風……というかCSSとか一部GitHubからそのまま持ってきてる感じのあるテーマ。

# 改修点
## SS

![変更点SS1](/images/12_26_00_01.png "変更点SS1 - Overview画面(大)")

![変更点SS2](/images/12_26_00_02.png "変更点SS2 - Post画面 + 画面(小)")

## 詳細

### 1. サイトタイトルを追加

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

### 2. Authorアイコンをクリックしても何も起きないように変更

[layout/layouts/partials/user-profile.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/user-profile.html)

クリックで画像単体で表示できても特に面白いことはないので。

---

### 3. 利用中アカウントへのショートカットを追加

MastodonとHatenaを（割と無意味に）追加。

---

### 4. 小プロフィールアイコンを削除

押しても何も起きなかったので。

利用中アカウント表示をこっちにも追加してもいいかもしれない。

---

### 5. 記事本文の冒頭ではなく概要を設定し表示するように変更

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

```md
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

### 6. 記事に設定したタグを表示

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

```
{{ with HOGE }}
~~~
{{ end }}
```
でHOGEが存在する場合に生成されるhtmlファイルに挿入される。

`{{ . }}` は`with`なり`range`なりで現在参照されている項目を表す。

`{{ lower STR }}` でASCII文字列を小文字に変換する  
（タグページのURLは設定したタグの小文字になる（英数字の場合））

---
### 7. 記事の更新時間の表示を変更・修正

[layout/layouts/partials/overview.html](https://github.com/suihan74/github-style/blob/master/layouts/partials/overview.html)

```html
<div class="mt-1">
    Updated <relative-time datetime="{{ .Lastmod.Format "2006-01-02 15:04" }}" class="no-wrap">
        {{ .Lastmod.Format "2006-01-02 15:04" }}</relative-time>
</div>
```

日付だけでなく時分も表示するように変更。

---

### 8. 改造後のテーマのリポジトリへのリンクを追加

はい。

---

### 9. 小幅画面にもサイトタイトルを追加

はい。

---

### 10. 小幅画面でも記事ヘッダ部分を省略しないように変更

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

[主な変更履歴](https://github.com/suihan74/github-style/commit/f0be7adcce5203d76a44409bf41f7ad45a3baa3e#diff-e720269f10ce3098515abef3dc8a581d)

クラスpt-0を除去することで小幅画面で記事タイトル部分のtopマージンが無くなるのを回避。  
d-noneを除去することで小幅画面でも記事ヘッダスペースを表示し続けるようになった。

- 著者名をクリックしたときにトップページに飛ばされていたのを、aboutページに遷移するように変更
- 記事タイトルを改行許可して省略しないように変更 -> [e696b5a](https://github.com/suihan74/github-style/commit/e696b5a22a3a79d81f0ef0d1045aed6ae9ace20e#diff-0c10dbee20afbafb8202ba360b55546a)
- 他画面同様descriptionとタグを追加
- 誤字:ModifyedをModifiedに変更……した後にUpdatedに変更（それほど意味はない）
- 時刻のフォーマットを他画面同様に変更

---

### 11. `<head>`部分

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
