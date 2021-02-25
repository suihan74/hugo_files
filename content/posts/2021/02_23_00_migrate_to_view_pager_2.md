---
title: "(いまさら)ViewPagerからViewPager2へ移行する"
description: "既存プロジェクトで継続使用していたViewPagerが非推奨になったのでViewPager2にいまさら置き換えた話"
tags: ["Android","Kotlin","ViewPager","ViewPager2"]
date: 2021-02-23T12:15:14+09:00
lastmod: 2021-02-23T12:15:14+09:00
archives:
    - 2021
    - 2021-02
    - 2021-02-23
hide_overview: false
draft: false
---

## 依存パッケージのアップデート

`androidx.fragment:fragment-ktx`を`1.2.5`から`1.3.0`に更新した結果、`Activity`や`Fragment`の扱い方に関する幾つかの変更が発生した。

今回の内容もそのひとつであり、「`FragmentPagerAdapter`が非推奨になったので`FragmentStateAdapter`に移行してくれ(そして必然的に`ViewPager`やめて`ViewPager2`に移行してくれ)」という問題に対応する。

### 他に対応したこと

[startActivityForResult()を新APIに置き換える](/posts/2021/02_25_00_relace_start_activity_for_result/)

## ViewPager

`ViewPager`はスワイプでページ切替できる複数の`Fragment`を表示するための`ViewGroup`であり、`TabLayout`などと一緒に使うことが多いように思う。

新しいプロジェクトでわざわざ`ViewPager`を使用する利点はとくにないので、普通に`ViewPager2`を導入すればいいのだが、既存のプロジェクトで横並びのシンプルなタブ実装に`ViewPager`を使用している場合に大急ぎで移行するほどの理由もとくに無い、ということで長らく放置していた。

`ViewPager`では表示内容の管理に`FragmentPagerAdapter`を使用しており、たとえば次のように用意する。

```kt:HogeFragmentPagerAdapter.kt
class HogeFragmentPagerAdapter(
    private val context : Contect,
    private val items : List<Item>,
    fragmentManager : FragmentManager
) : FragmentPagerAdapter(fragmentManager, BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT) {
    override fun getItem(position: Int) : Fragment = items[position].createFragment()

    override fun getPageTitle(position: Int) : CharSequence? =
        context.getText(items[position].textId)

    override fun getCount() = items.size
}

// ------ //

// 表示内容例
enum class Item(
    /** タイトル文字列リソースID */
    @StringRes val textId: Int,

    /** 対応するフラグメントを生成 */
    val createFragment : ()->Fragment
) {
    FOO(R.string.foo, { FooFragment() }),

    BAR(R.string.bar, { BarFragment() }),

    BAZ(R.string.baz, { BazFragment() })
}
```

```kt:HogeActivity.kt
override fun onCreate(savedInstanceState: Bundle?) {
    // ...

    // `ViewPager`にアダプタを設定
    binding.viewPager.adapter = HogeFragmentPagerAdapter(
        this,
        Item.values(),
        supportFragmentManager
    )

    // `TabLayout`と同期
    binding.tabLayout.setupWithViewPager(binding.viewPager)
}
```

[ViewPager を使用してフラグメント間をスライドする | Android デベロッパー | Android Developers](https://developer.android.com/training/animation/screen-slide?hl=ja)

## ViewPager2

`ViewPager2`は`ViewPager`の改良版であり、サポートが今後も継続するという他に幾つかの追加機能もある(縦方向に並べる、RTLサポート、`DiffUtil`利用など)。

[ViewPager2 を使用してフラグメント間をスライドする | Android デベロッパー | Android Developers](https://developer.android.com/training/animation/screen-slide-2?hl=ja)

[ViewPager から ViewPager2 に移行する | Android デベロッパー | Android Developers](https://developer.android.com/training/animation/vp2-migration?hl=ja)

移行といっても大きく変わる部分は少なく、先の`HogeFragmentPagerAdapter`を差し替える場合は次のように書き換える。

```kt:HogeFragmentStateAdapter.kt
class HogeFragmentStateAdapter(
    activity : FragmentActivity,
    private val items : List<Item>
) : FragmentStateAdapter(acitivity) {

    override fun getItemCount() : Int = items.size

    override fun createFragment(position: Int) : Fragment = items[position].createFragment()
}
```

```kt:HogeActivity.kt
override fun onCreate(savedInstanceState: Bundle?) {
    // ...

    // `ViewPager2`にアダプタを設定
    binding.viewPager2.adapter = HogeFragmentStateAdapter(this, Item.values())

    // `TabLayout`と同期
    TabLayoutMediator(binding.tabLayout, binding.viewPager2) { tab, position ->
        tab.setText(Item.values()[position].textId)
    }.attach()
}
```

### コンストラクタ

`FragmentStateAdapter`のコンストラクタには`FragmentActivity`、`Fragment`、`FragmentManager, Lifecycle`のどれかを渡す。

アダプタがフラグメントを扱うのに使用する`FragmentManager`は、`FragmentActivity`を渡す場合`FragmentActivity#supportFragmentManager`が、`Fragment`を渡す場合は`Fragment#childFragmentManager`が使用される。

### TabLayoutと同期

`TabLayoutMediator`を使用して、ページとタブ表示の同期を行う。

最後に`attach()`を忘れると外観に反映されないので注意が必要。

### 生成済みのページのFragmentを取得する

「現在表示している`Fragment`」など必要になる場合、アダプタのコンストラクタに与えた`FragmentManager`から次のようにして対応インデックスの`Fragment`を取得できる。

```kt:Fragment取得.kt
class HogeFragmentStateAdapter(
    private val activity : FragmentActivity
) : FragmentStateAdapter(activity) {
    // ...

    fun findFragment(poisition: Int) : Fragment? =
        activity.supportFragmentManager.findFragmentByTag("f$position")
}
```

アダプタが管理する`Fragment`は`"f" + インデックス`のタグがついて`FragmentManager`に登録されている。

### itemCount

`FragmentPagerAdapter`では`getCount()`メソッドを使用していたため、プロパティアクセス時には`count`を使用したが、  
`FragmentStateAdapter`では`getItemCount()`に変更しているため、プロパティアクセスでアイテム数を取得している箇所がある場合は`adapter.count`の部分を`adapter.itemCount`に修正する必要がある。
