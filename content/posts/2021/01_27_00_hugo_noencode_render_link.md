---
title: "Hugoのrender-linkでURLエンコードしないリンクを挿入する"
description: "render-linkフックを使用している場合に選択的にパーセントエンコードを施さないリンクも作成する方法"
tags: ["Hugo","html"]
date: 2021-01-27T17:12:53+09:00
lastmod: 2021-01-27T17:12:53+09:00
archives:
    - 2021
    - 2021-01
    - 2021-01-27
hide_overview: false
draft: false
---

## render-link

`Hugo v0.62.0`以降では、マークダウンからHTMLを生成する際に`a`タグや`img`タグのフォーマットをユーザーがカスタマイズすることができる。  
[公式リファレンス](https://gohugo.io/getting-started/configuration-markup#link-with-title-markdown-example)の通りに用意している場合次のようなrender-linkフックになる。

```html:layouts/_default/_markup/render-link.html
<a
  href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}
  {{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>
    {{ .Text | safeHTML }}
</a>
```

`.Destination`はリンク先のURLなのだが、その内容はパス部分にパーセントエンコーディングが施されたものになる。

```md:マークダウン.md
[link](https://hoge.com/hoge#foo(bar))
```

▼

```html:生成されたHTML.html
<a href="https://hoge.com/hoge#foo%28bar%29" target="_blank" rel="noopener">link</a>
```

通常はこれで問題ないのだが、困ったことにリンク先によっては正しく遷移できない場合がなくもない。

- 遷移したいリンク  
    [https://developer.android.com/reference/android/content/pm/PackageManager#getApplicationInfo(java.lang.String,%20int)](https://developer.android.com/reference/android/content/pm/PackageManager#getApplicationInfo(java.lang.String,%20int) "_noencode")

- 実際のリンク  
    [https://developer.android.com/reference/android/content/pm/PackageManager#getApplicationInfo%28java.lang.String,%20int%29](https://developer.android.com/reference/android/content/pm/PackageManager#getApplicationInfo(java.lang.String,%20int))

## パーセントエンコーディングを回避する

パーセントエンコードされたURLをデコードした文字列を生成するには次のようにする。

```html
{{ printf "%q" .Destination | safeHTMLAttr }}
```

`printf "%q" xx`は文字列としてエスケープした状態で出力される(「"xx"」)。  
`.Destination`をただそのまま渡すとHugoはセキュリティ的に問題のあるタグ属性であると判断して強制的に代替文字列`zgotmplz`を挿入してしまう。`safeHTMLAttr`関数を使用してこれを回避する。

全てのリンクをエンコードしないようにするならrender-linkの`href`属性部分を単純に置き換えればいいが、「基本的にはリンクはURLエンコードして、問題がある場合だけデコード状態で挿入する」という風にするにはたとえば次のようにする。

```html:layouts/_default/_markup/render-link.html
<!-- タイトルを"_noencode"にするとURLエンコードを行わないようにする -->
<!-- 例: [text](url "_noencode") -->
<a
  {{ if eq .Title "_noencode" }}
    {{ printf "href=%q" .Destination | safeHTMLAttr }}
  {{ else }}
    href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}
  {{ end }}
  {{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>
    {{ .Text | safeHTML }}
</a>
```

```md:マークダウン.md
[link](https://hoge.com/hoge#foo(bar) "_noencode")
```

▼

```html:生成されたHTML.html
<a href="https://hoge.com/hoge#foo(bar)" target="_blank" rel="noopener">link</a>
```

「タイトルに`"_noencode"`を指定した場合はエンコードしない」という風にしている。この場合タイトルと併用できなくなってしまうわけだが、問題があれば`replace`とか`replaceRE`とか使って`_noencode`だけ消して残りの部分をタイトルとして挿入するようにすればいいと思う。
