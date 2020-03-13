---
title: "Androidアプリ開発Tips – アプリバーをコード側で明示的に表示/非表示"
description: "AppBarLayoutをコード側から開閉する方法"
tags: ["android", "kotlin", "migrated_from_previous_blog"]
date: 2019-11-20T02:41:00+09:00
lastmod: 2020-03-14T01:10:00+09:00
archives:
    - 2019
    - 2019-11
    - 2019-11-20
hide_overview: true
draft: false
---

## おことわり

この記事は旧ブログから引っ張ってきたものの再掲です。

---

## コンテンツスクロールにあわせてAppBarLayoutの表示状態を変える

`AppBarLayout`の`app:layout_scrollFlags`属性を適切に設定しておくと、画面内のViewのスクロールにあわせてアプリバー内のView（ここではToolbar）が出たり隠れたりする。

{{< figure src="/images/2019/11_20_01_01.gif" width="25%" attr="ツールバー（「総合」と書かれた部分）がリストのスクロールに合わせて隠れる" >}}

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbarLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
        <Toolbar
                android:id="@+id/toolbar"
                app:layout_scrollFlags="enterAlways|scroll"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"/>
    </com.google.android.material.appbar.AppBarLayout>
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

ちなみにコード側で設定するなら概ねこんな感じ。

```kt
toolbar.updateLayoutParams<AppBarLayout.LayoutParams> {
    scrollFlags = AppBarLayout.LayoutParams.SCROLL_FLAG_SCROLL or AppBarLayout.LayoutParams.SCROLL_FLAG_ENTER_ALWAYS
}
```

スクロールを監視するViewには`app:layout_behavior`属性をこんな感じで記述しておく。

```xml
<!-- ViewPagerなのは適当 -->
<androidx.viewpager.widget.ViewPager
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"/>
```

この`@string/appbar_scrolling_view_behavior`とかいう文字列リソースなんやねんというと中身はこんなやつだった。

```xml
<string name="appbar_scrolling_view_behavior" translatable="false">com.google.android.material.appbar.AppBarLayout$ScrollingViewBehavior</string>
```

## コード側から強制的にAppBarLayoutの表示状態を変更する

さて本題。

ツールバーが出たり消えたりの状態をコンテンツのスクロールに依らずにコード側から変更するには次のように`AppBarLayout.setExpanded(Boolean, Boolean)`メソッドを使用する。

なんでこんなことをしたいかというと、ひとつのActivityにおいてコンテンツのスクロールによってツールバーが隠された状態でコンテンツ部分を別の物に変更したときにツールバーが隠れたままだと都合が悪いことがあったからだ。

次の「不正な挙動」のアニメーションでは、画面遷移後にIMEが表示されるがツールバーが隠れているため何を入力するためにIMEが表示されたのか全く分からない。

```kt
// 第一引数: 表示する場合true、隠す場合false
// 第二引数: アニメーションするか否か
appbarLayout.setExpanded(true, true)
```

{{< figure src="/images/2019/11_20_01_02.gif" width="25%" attr="不正な挙動: 画面遷移後、アプリバー上のSearchViewが見えない" >}}

{{< figure src="/images/2019/11_20_01_03.gif" width="25%" attr="意図した挙動: 画面遷移後、アプリバーを強制的に再表示できた" >}}
