---
title: "SearchViewのカーソル色を指定する方法"
description: "SearchView内のEditText部分のカーソル色を指定する。"
tags: ["android", "kotlin", "SearchView", "style"]
date: 2020-03-31T00:05:00+09:00
lastmod: 2020-03-31T00:05:00+09:00
archives:
    - 2020
    - 2020-03
    - 2020-03-31
hide_overview: false
draft: false
---

## 前提

### res/menu/search.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
            android:id="@+id/search_view"
            android:title="search"
            app:showAsAction="always"
            app:actionViewClass="androidx.appcompat.widget.SearchView"/>

</menu>
```

たとえばこんな感じでmenuアイテムのリソースを用意しているとする。  
ここで使用している`SearchView`上の文字入力部分(`SearchView.SearchAutoComplete`)の現在入力位置を示すカーソルの色(と形状)を明示的に指定する方法について以下に記す。

## Drawableリソースの用意

### res/drawable/searchview_cursor.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle" >
    <solid android:color="@color/searchCursor" />
    <size android:width="2dp" />
</shape>
```

`@color/searchCursor`の部分を指定したい色に書き換える。

## styleの指定

### res/values/styles.xml

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <!-- アプリ用の基本テーマ。SearchView以外の色々はすべて省略している -->

    <!-- SearchView -->
    <item name="searchViewStyle">@style/SearchViewStyle</item>
    <item name="autoCompleteTextViewStyle">@style/AutoCompleteTextView</item>
</style>

<!-- SearchView -->
<style name="SearchViewStyle" parent="Widget.AppCompat.SearchView">
    <item name="queryBackground">@android:color/transparent</item>
    <item name="submitBackground">@android:color/transparent</item>
    <item name="closeIcon">@drawable/ic_baseline_close</item>
    <item name="searchIcon">@drawable/ic_baseline_search</item>
    <item name="commitIcon">@drawable/ic_baseline_search</item>
    <item name="goIcon">@drawable/ic_baseline_search</item>
    <item name="textColor">@color/colorPrimaryText</item>
</style>

<!-- SearchView上の文字入力部分 -->
<style name="AutoCompleteTextView" parent="Widget.AppCompat.AutoCompleteTextView">
    <!-- 入力位置のカーソル -->
    <item name="android:textCursorDrawable">@drawable/searchview_cursor</item>
</style>
```

SearchViewStyleの指定内容の詳細はこの辺参考に。

[android - How can I style the SearchView when using a Toolbar as an Action Bar? - Stack Overflow](https://stackoverflow.com/a/28018439)

>
>```xml
><style name="MySearchViewStyle" parent="Widget.AppCompat.SearchView">
>   <!-- Background for the search query section (e.g. EditText) -->
>   <item name="queryBackground">...</item>
>   <!-- Background for the actions section (e.g. voice, submit) -->
>   <item name="submitBackground">...</item>
>   <!-- Close button icon -->
>   <item name="closeIcon">...</item>
>   <!-- Search button icon -->
>   <item name="searchIcon">...</item>
>   <!-- Go/commit button icon -->
>   <item name="goIcon">...</item>
>   <!-- Voice search button icon -->
>   <item name="voiceIcon">...</item>
>   <!-- Commit icon shown in the query suggestion row -->
>   <item name="commitIcon">...</item>
>   <!-- Layout for query suggestion rows -->
>   <item name="suggestionRowLayout">...</item>
></style>
>```
