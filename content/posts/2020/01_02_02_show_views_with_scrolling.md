---
title: "スクロールで画面上下端のツールバーやボタンを表示/非表示"
description: コード側からLayoutParamを設定する方法
tags: ["android", "kotlin"]
date: 2020-01-03T00:12:34+09:00
lastmod: 2020-01-03T00:12:34+09:00
archives:
    - 2020
    - 2020/01
    - 2020/01/02
draft: false
---

# やること

こういうやつ。

![やりたいやつ](/images/2020/01_02_02_01.gif "様子アニメーション")

画面中央の`RecyclerView`のスクロールに合わせて、画面上端の`Toolbar`が上に向かって隠れ、画面下端の`FloatingActionButton`(を複数持った`Fragment`)が下に向かって隠れている。

# レイアウトで指定する場合

レイアウト側で設定する場合は以下のようにする。  
Satenaで実際に使っているレイアウトファイルだが、必要な部分だけ適当に抜粋した。

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:background="?attr/panelBackground"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbar_layout"
            app:elevation="0dp"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

        <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                app:layout_scrollFlags="enterAlways|scroll"
                android:layout_width="match_parent"
                android:layout_height="match_parent"/>

    </com.google.android.material.appbar.AppBarLayout>

    <!-- ブクマリストを表示するための領域 -->
    <FrameLayout
            android:id="@+id/content_layout"
            app:layout_behavior="com.google.android.material.appbar.AppBarLayout$ScrollingViewBehavior"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

    <!-- ブクマリスト用下部ボタンを表示するための領域 -->
    <FrameLayout
            android:id="@+id/buttons_layout"
            app:layout_behavior="com.google.android.material.behavior.HideBottomViewOnScrollBehavior"
            android:layout_gravity="bottom"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

`FrameLayout`部分にそれぞれコンテンツ内容にあたる`Fragment`が`replace`される。

`@id/content_layout`には主に`RecyclerView`が配置されたフラグメントを複数タブとして持つフラグメントが置かれ、  
`@id/buttons_layout`には`FloatingActionButton`が配置されている。

ちなみに、`AppBarLayout`に`app:elevation="0dp"`が設定されているのは、こうすることで`@id/content_layout`のフラグメントがもつ`TabLayout`との間に不要な影が描画されなくなるため。


## トリガー

スクロールを監視して他のViewを隠すトリガーにしたい対象(`@id/content_layout`がそれにあたる)には、

```xml
app:layout_behavior="com.google.android.material.appbar.AppBarLayout$ScrollingViewBehavior"
```

を設定する。


## ツールバー

AppBarLayout内のViewをスクロールに合わせて画面上部に隠すには、

```xml
app:layout_scrollFlags="enterAlways|scroll"
```

とか設定する。`app:layout_scrollFlags`の値による差異については以下などを見ればいいように思う。

[Android Design — Collapsing Toolbar: ScrollFlags Illustrated](https://medium.com/martinomburajr/android-design-collapsing-toolbar-scrollflags-e1d8a05dcb02)


## ボトムナビゲーション

`HideBottomViewOnScrollBehavior`というまんまな名前のBehaviorがあるのでこれを設定すると、スクロールに合わせて画面下部に向かってViewが隠れる。

```xml
app:layout_behavior="com.google.android.material.behavior.HideBottomViewOnScrollBehavior"
```


# コードで指定する場合

設定値にあわせて表示切替をしたりしなかったりを動的に設定したい場合など、DataBindingとかしようと思わないならActivityなり何なりにkotlin側で記述することもできる。

次の例では、(少し手を加えた)`SharedPreferences`の設定値によって画面上下端のViewの表示切替をするかしないかをコード側で設定している。

```kt
// スクロールでツールバーを隠す
toolbar.updateLayoutParams<AppBarLayout.LayoutParams> {
    scrollFlags =
        if (prefs.getBoolean(PreferenceKey.HIDE_TOOLBAR_BY_SCROLLING))
            AppBarLayout.LayoutParams.SCROLL_FLAG_SCROLL or AppBarLayout.LayoutParams.SCROLL_FLAG_ENTER_ALWAYS
        else
            0
}

// スクロールでボタンを隠す
buttons_layout.updateLayoutParams<CoordinatorLayout.LayoutParams> {
    behavior =
        if (prefs.getBoolean(PreferenceKey.HIDE_BUTTONS_BY_SCROLLING))
            HideBottomViewOnScrollBehavior<View>(this@HogeActivity, null)
        else
            null
}
```
