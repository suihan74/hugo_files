---
title: "端末にインストールされているアプリ情報を取得する"
description: "ApplicationInfoの取得の仕方とちょっとした使い方"
tags: ["Android", "Kotlin", "PackageManager"]
date: 2021-01-09T17:20:00+09:00
lastmod: 2021-01-09T17:20:00+09:00
archives:
    - 2021
    - 2021-01
    - 2021-01-09
hide_overview: false
draft: false
---

型とか分かりやすくするために大体は関数で書いているが、あくまで紹介のためなので使うときは適当に変えるようにする。

## インストール済みの全アプリ情報

```kt:インストール済みの全アプリ情報.kt
val apps = context.packageManager.getInstalledApplications(0)
```

## パッケージ名を指定してアプリ情報を取得

第二引数のフラグに`PackageManager.MATCH_UNINSTALLED_PACKAGES`が指定されている場合、指定されたパッケージがインストールされていなかったら「過去にインストールされていたが削除済みで、かつデータが残っている」アプリも探す。  
それでも見つからなかったら`PackageManager.NameNotFoundException`が送出される。

```kt:パッケージ名からアプリ情報を取得.kt
fun applicationInfo(packageName: String) : ApplicationInfo? =
    try {
        context.packageManager.getApplicationInfo(packageName, 0)
    }
    catch (e: PackageManager.NameNotFoundException) {
        null
    }
```

## アプリの表示名を取得

たとえば[Satena](https://play.google.com/store/apps/details?id=com.suihan74.satena)の場合、パッケージ名は`com.suihan74.satena`でランチャーなどに表示される表示名は`Satena`である。

`ApplicationInfo`自体には表示名は格納されていない。  
以下のようにして手動で取得する。

```kt:アプリの表示名.kt
fun displayName(appInfo: ApplicationInfo) : CharSequense =
    context.packageManager.getApplicationLabel(appInfo)
```

または

```kt:アプリの表示名(別の方法).kt
fun displayName(appInfo: ApplicationInfo) : CharSequense =
    appInfo.loadLabel(context.packageManager)
```

表示名の取得に失敗した場合は、`name`(多分AndroidManifest.xmlに記述された`<application>`タグの属性`android:name`の内容)が返される。

[PackageManager#getApplicationLabel | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/PackageManager#getApplicationLabel(android.content.pm.ApplicationInfo))

[PackageItemInfo#loadLabel | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/PackageItemInfo#loadLabel(android.content.pm.PackageManager))

[PackageItemInfo#name | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/PackageItemInfo#name)

## アプリアイコンをロードする

```kt:アプリアイコンをロード.kt
fun loadAppIcon(appInfo: ApplicationInfo) : Drawable =
    context.packageManager.getApplicationIcon(appInfo)
```

または

```kt:アプリアイコンをロード(別の方法).kt
fun loadAppIcon(appInfo: ApplicationInfo) : Drawable =
    appInfo.loadIcon(context.packageManager)
```

アプリアイコンのロードに失敗した場合はデフォルトアイコンが返される。

`PackageManager#getApplicationIcon`を使う場合、引数は`ApplicationInfo`だけではなくパッケージ名の文字列でも可能。ただし先述した表示名の取得には`ApplicationInfo`が必要なので、大抵の場合どちらにしろ`ApplicationInfo`は必要になると思われる。

[PackageManager#getApplicationIcon | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/PackageManager#getApplicationIcon(android.content.pm.ApplicationInfo))

[PackageItemInfo#loadIcon | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/PackageItemInfo#loadIcon(android.content.pm.PackageManager))

ロゴ、バナーなどもあるようだが同様なので割愛。

## アプリ情報の種類を調べる

「`ApplicationInfo`が指すアプリがシステムアプリかどうか」などのフラグを読んで、それがどんな種類のアプリの情報なのかを調べることができる。

- `FLAG_DEBUGGABLE` ... デバッグできる
- `FLAG_SYSTEM` ... システムにプリインストールされたアプリ
- `FLAG_UPDATED_SYSTEM_APP` ... アップデートされたプリインストールアプリ
- `FLAG_INSTALLED` ... 現在のユーザーが利用できる

など。他にも大きい(or小さい)画面で使用できるか、など色々フラグがあるっぽい。

参考

[ApplicationInfo | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/ApplicationInfo.html#FLAG_ALLOW_BACKUP)

## アプリを起動するためのインテントを取得する

アクティビティとして起動できないアプリの場合`null`が返る。

```kt
fun getLaunchIntent(appInfo: ApplicationInfo) : Intent? =
    context.packageManager.getLaunchIntentForPackage(appInfo.packageName)
```

## アプリ情報取得時の追加オプション

とくに指定しない`0`の他、`PackageManager.GET_~~`を指定すれば該当する追加情報を付加して取得でき、`PackageManager.MATCH_~~`を指定すれば特殊な条件に合致するものを取得できる。

### `PackageManager#getInstalledApplications`に渡せるフラグ

[PackageManager#getInstalledApplications | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/PackageManager#getInstalledApplications(int))

### `PackageManager#getApplicationInfo`に渡せるフラグ

[PackageManager#getApplicationInfo | Android デベロッパー | Android Developers](https://developer.android.com/reference/android/content/pm/PackageManager#getApplicationInfo(java.lang.String,%20int))
