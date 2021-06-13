---
title: "CookieManagerでSecure属性のついたCookieを取得する方法"
description: "CookieManagerを使ったcookieの参照のしかた"
tags: ["Android","Kotlin","Cookie","WebView"]
date: 2021-06-13T13:17:37+09:00
lastmod: 2021-06-13T13:17:37+09:00
archives:
    - 2021
    - 2021-06
    - 2021-06-13
hide_overview: false
draft: false
---

## 要点

`CookieManager`を使用して`Secure`属性の付いたCookieを取得するには、`CookieManager#getCookie`に渡す文字列には`https://`から始まるURLを渡さなければならない。
(※HttpOnly属性も同様)

## WebViewとCookieManager

Androidで`WebView`を使用して閲覧しているページのCookieをプログラム側で取得するには次のように`WebView`と`CookieManager`を準備する。

```kt:CookieManagerの使い方.kt
class HogeActivity : AppCompatActivity() {
    private val cookieManager = CookieManager.getInstance()
    private lateinit var binding : ActivityHogeBinding

    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        binding = ActivityHogeBinding.inflate(layoutInflater)
        setContentView(binding.root)
        initializeWebView(binding.webView)
    }

    override fun finish() {
        // WebView次回利用時に現在のページから始まらないようにするために必要
        binding.webView.loadUrl("about:blank")
        super.finish()
    }

    // ------ //

    private fun initializeWebView(webView: WebView) {
        cookieManager.acceptCookie()
        cookieManager.setAcceptThirdPartyCookies(webView, true)
        // ほか色々設定

        webView.loadUrl("https://~~~")
    }
}
```

`CookieManager#getCookie(URL or Domain)`を使用して`key0=value0; key1=value1; ...`の書式でCookie文字列が取得できるので、適当に連想配列にしたり必要なものだけ取り出したり。

```kt:Cookie参照.kt
@OptIn(ExperimentalStdlibApi::class)
val cookies = buildMap<String, String> {
    val cookiesStr = cookieManager.getCookie(".hoge.com")
    cookiesStr.split(";").forEach {
        val separatorIndex = it.indexOf("=")
        if (separatorIndex == -1) return@forEach
        val key = it.substring(0, separatorIndex)
        val value = it.substring(separatorIndex + 1)
        put(key, value)
    }
}
```

上記のように`CookieManager#getCookie`に".hoge.com"のようなドメイン名を渡す場合、得られる文字列に`Secure`属性や`HttpOnly`属性がついたCookieは含まれないので注意が必要。

`Secure`属性がついたCookieを取得する場合は、次のように明示的に`https://`から始まるURLを渡す必要がある。

```kt:Cookie参照.kt
@OptIn(ExperimentalStdlibApi::class)
val cookies = buildMap<String, String> {
    val cookiesStr = cookieManager.getCookie("https://www.hoge.com")
    cookiesStr.split(";").forEach {
        val separatorIndex = it.indexOf("=")
        if (separatorIndex == -1) return@forEach
        val key = it.substring(0, separatorIndex)
        val value = it.substring(separatorIndex + 1)
        put(key, value)
    }
}
```

せやろなという感じだった。

[HTTP Cookie の使用 - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Cookies#:~:text=Cookie%20%E3%81%B8%E3%81%AE%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E5%88%B6%E9%99%90&text=Secure%20%E5%B1%9E%E6%80%A7%E3%81%8C%E3%81%A4%E3%81%84%E3%81%9F,%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%9B%E3%82%93%E3%80%82&text=%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%81%AB%E9%80%81%E4%BF%A1%E3%81%95%E3%82%8C%E3%82%8B%E3%81%A0%E3%81%91%E3%81%A7%E3%81%99%E3%80%82)
