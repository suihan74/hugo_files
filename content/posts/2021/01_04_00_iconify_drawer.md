---
title: "アイコン化可能なドロワーをでっち上げる"
description: "MotionLayoutを使ってアイコン化可能なDrawerLayout風のものを作る例"
tags: ["Android","Kotlin","MotionLayout"]
date: 2021-01-04T17:42:49+09:00
archives:
    - 2021
    - 2021-01
    - 2021-01-04
hide_overview: false
draft: false
---

## 追記 (2021-01-26)

メニュー開閉時に仮想キーボードを閉じるためのコードを追加  
(該当位置は[commit: 847804a](https://github.com/suihan74/hugo_files/commit/847804ab21e23ca38bf6f0c100f9a32688837a8f)を参照)

---

## なにこれ

{{<img src="sample.gif" zoom="0.8" title="アイコン化可能なドロワー">}}

こんなの。

画面横幅の狭いスマホ用レイアウトでこういうことをするのは恐らく非推奨で、おとなしくツールバーにハンバーガーボタンでも置いておけという話なのかもしれないが、やりたくなったのだから仕方がない。

最初のサンプルのアプリは最近作ってるこれ。

[suihan74/NotificationReporter: 消灯後の画面に通知LED代わりのものとかを表示するやつ](https://github.com/suihan74/NotificationReporter)

実際に今回の内容使ってる部分のコードはここ。

[NotificationReporter/app/src/main/java/com/suihan74/notificationreporter/scenes/preferences at main · suihan74/NotificationReporter](https://github.com/suihan74/NotificationReporter/tree/main/app/src/main/java/com/suihan74/notificationreporter/scenes/preferences)

## 実装例

### レイアウト

```xml:layout/activity_hoge.xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
    <data>
        <variable
            name="vm"
            type="com.suihan74.example.HogeViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/mainLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.constraintlayout.motion.widget.MotionLayout
            android:id="@+id/motionLayout"
            android:background="@android:color/transparent"
            app:layoutDescription="@xml/motion_menu_toggle"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_width="0dp"
            android:layout_height="0dp">

            <!-- 画面右側に表示するコンテンツ部分 -->
            <!-- 今回は選択中のメニュー項目にあわせてFragmentを切り替える -->
            <androidx.viewpager2.widget.ViewPager2
                android:id="@+id/contentPager"
                currentItem="@={vm.selectedMenuItem}"
                android:layout_width="0dp"
                android:layout_height="0dp"
                android:layout_marginStart="@dimen/menuWidthCompact"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintTop_toTopOf="parent">
            </androidx.viewpager2.widget.ViewPager2>

            <!-- メニュー表示中の背景 -->
            <View
                android:id="@+id/clickGuard"
                android:background="#99000000"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                android:layout_width="0dp"
                android:layout_height="0dp"/>

            <!-- メニュー部分 -->
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/menuRecyclerView"
                style="@style/RecyclerView.Linear"
                android:background="@color/menuBackground"
                android:elevation="16dp"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                android:layout_width="@dimen/menuWidthCompact"
                android:layout_height="0dp">
            </androidx.recyclerview.widget.RecyclerView>

        </androidx.constraintlayout.motion.widget.MotionLayout>
    </androidx.constraintlayout.widget.ConstraintLayout>

    <!-- より前面に表示するものがあればこの辺に -->
    ...
</layout>
```

メニュー部分と、メニュー最大化中に表示する背景の他、それらより下に表示する(タッチを処理する必要がある)ものはすべて`MotionLayout`の子にする。

### MotionScene

```xml:xml/motion_menu_toggle.xml
<?xml version="1.0" encoding="utf-8"?>
<!--
    メニュードロワ部分を常にアイコン表示、右にスワイプして詳細表示にする
-->
<MotionScene
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:motion="http://schemas.android.com/apk/res-auto">

    <!-- メニュー部分横スワイプでメニュー開閉 -->
    <Transition
        motion:constraintSetStart="@id/compact"
        motion:constraintSetEnd="@id/full">

        <OnSwipe
            motion:touchAnchorId="@+id/menuRecyclerView"
            motion:touchRegionId="@+id/menuRecyclerView"
            motion:onTouchUp="autoComplete"
            motion:maxAcceleration="80"
            motion:touchAnchorSide="end"
            motion:dragDirection="dragEnd" />
    </Transition>

    <!-- 背景クリックで閉じる -->
    <Transition
        motion:constraintSetEnd="@id/compact">
        <OnClick
            motion:targetId="@+id/clickGuard"
            motion:clickAction="transitionToEnd" />
    </Transition>

    <!-- 最小状態(アイコン化) -->
    <ConstraintSet android:id="@+id/compact">
        <Constraint
            android:id="@id/clickGuard"
            android:visibility="invisible"
            android:layout_width="0dp"
            android:layout_height="0dp"
            motion:layout_constraintTop_toTopOf="parent"
            motion:layout_constraintBottom_toBottomOf="parent"
            motion:layout_constraintStart_toStartOf="parent"
            motion:layout_constraintEnd_toEndOf="parent" />

        <Constraint
            android:id="@id/menuRecyclerView"
            android:layout_width="@dimen/menuWidthCompact"
            android:layout_height="0dp"
            motion:layout_constraintTop_toTopOf="parent"
            motion:layout_constraintBottom_toBottomOf="parent"
            motion:layout_constraintStart_toStartOf="parent" />
    </ConstraintSet>

    <!-- 最大状態(コンテンツとメニューの間に背景表示してメニュー横幅を最大化) -->
    <ConstraintSet android:id="@+id/full">
        <Constraint
            android:id="@id/clickGuard"
            android:visibility="visible"
            android:layout_width="0dp"
            android:layout_height="0dp"
            motion:layout_constraintTop_toTopOf="parent"
            motion:layout_constraintBottom_toBottomOf="parent"
            motion:layout_constraintStart_toStartOf="parent"
            motion:layout_constraintEnd_toEndOf="parent" />

        <Constraint
            android:id="@id/menuRecyclerView"
            android:layout_width="@dimen/menuWidthFull"
            android:layout_height="0dp"
            motion:layout_constraintTop_toTopOf="parent"
            motion:layout_constraintBottom_toBottomOf="parent"
            motion:layout_constraintStart_toStartOf="parent" />
    </ConstraintSet>

</MotionScene>
```

### メニュー項目のレイアウト

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
    <data>
        <import type="androidx.lifecycle.LiveData"/>
        <import type="com.suihan74.example.MenuItem"/>

        <variable
            name="item"
            type="MenuItem"/>

        <variable
            name="selectedItem"
            type="LiveData&lt;MenuItem&gt;"/>
    </data>

    <FrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:background="@{selectedItem == item ? @color/selectedMenuItemBackground : @android:color/transparent}"
        android:elevation="@{selectedItem == item ? 48f : 0f}"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:background="?selectableItemBackground"
            android:paddingHorizontal="@dimen/menuIconPadding"
            android:paddingVertical="@dimen/menuIconPadding"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <ImageView
                android:id="@+id/icon"
                src="@{item.iconId}"
                app:tint="?android:textColor"
                android:background="@android:color/transparent"
                android:contentDescription="@null"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                android:layout_width="@dimen/menuIconSize"
                android:layout_height="@dimen/menuIconSize"/>

            <TextView
                android:id="@+id/label"
                textId="@{item.labelId}"
                android:background="@android:color/transparent"
                android:textSize="18sp"
                android:singleLine="true"
                android:ellipsize="none"
                app:layout_constraintStart_toEndOf="@id/icon"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                android:layout_marginStart="12dp"
                android:layout_width="0dp"
                android:layout_height="wrap_content"/>

        </androidx.constraintlayout.widget.ConstraintLayout>
    </FrameLayout>
</layout>
```

### dimens

```xml:values/dimens.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- メニュー部分を閉じているときの横幅 -->
    <dimen name="menuWidthCompact">48dp</dimen>
    <!-- メニュー部分を開いているときの横幅 -->
    <dimen name="menuWidthFull">256dp</dimen>

    <!-- メニューアイコンサイズ -->
    <dimen name="menuIconSize">24dp</dimen>
    <!-- メニューアイコンパディング(2倍してmenuIconSizeと足したらmenuWidthCompactになるようにする) -->
    <dimen name="menuIconPadding">12dp</dimen>
</resources>
```

### コード

```kt:HogeActivity.kt
class HogeActivity : AppCompatActivity() {

    // `lazyProvideViewModel`は良い感じに`ViewModel`を生成するための拡張
    // -> https://suihan74.github.io/posts/2020/09_14_01_provide_view_model/
    val viewModel by lazyProvideViewModel {
        HogeViewModel()
    }

    // ------ //

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding = ActivityHogeBinding.inflate(layoutInflater).also {
            it.vm = viewModel
            it.lifecycleOwner = this
        }
        setContentView(binding.root)

        // ページ選択メニュー
        initializeMenu(binding)

        // ページビュー
        binding.contentPager.also { pager ->
            pager.adapter = PageStateAdapter(supportFragmentManager, lifecycle)
        }
    }

    // ------ //

    /** ページ選択メニューの準備 */
    private fun initializeMenu(binding: ActivityHogeBinding) {
        // メニュー項目を初期化
        val list = binding.menuRecyclerView.also { list ->
            // 項目のDataBinding用に`ListAdapter<T,VH>`を拡張したやつ
            // 関係ないので中身は割愛
            val adapter = BindingListAdapter<MenuItem, ListItemHogeMenuBinding>(
                R.layout.list_item_hoge_menu,
                this,
                MenuItem.DiffCallback(),
            ) { binding, item ->
                binding.item = item
                binding.selectedItem = viewModel.selectedMenuItem
            }

            list.adapter = adapter.apply {
                setOnClickItemListener { binding ->
                    viewModel.selectedMenuItem.value = binding.item
                }

                // リストにヘッダを追加するための`submitItems`の拡張
                // 中身は割愛
                submit(
                    items = MenuItem.values().toList(),
                    header = { parent ->
                        // 適当なheightを指定した空の`View`をヘッダとして生成
                        ListHeaderMenuBinding.inflate(layoutInflater, parent, false).root.also {
                            it.setOnClickListener {}
                            it.setOnLongClickListener { false }
                        }
                    }
                )
            }
        }

        // メニュー操作時に仮想キーボードを閉じる
        binding.motionLayout.setTransitionListener(object : MotionLayout.TransitionListener {
            override fun onTransitionStarted(p0: MotionLayout?, p1: Int, p2: Int) {
                // 仮想キーボードを閉じてフォーカスをクリアする`View`の拡張関数
                // 実装は割愛
                currentFocus?.hideSoftInputMethod(binding.mainLayout)
            }
            override fun onTransitionChange(p0: MotionLayout?, p1: Int, p2: Int, p3: Float) {}
            override fun onTransitionCompleted(p0: MotionLayout?, p1: Int) {}
            override fun onTransitionTrigger(p0: MotionLayout?, p1: Int, p2: Boolean, p3: Float) {}
        })

        // `MotionLayout`にタッチイベントを伝播させる
        // リスト、各項目のタッチイベント処理で伝播が止まってしまうので、
        // その前に明示的に`MotionLayout`にもイベントを送り付けるようにしている

        list.setOnTouchListener { _, motionEvent ->
            binding.motionLayout.onTouchEvent(motionEvent)
            return@setOnTouchListener false
        }

        list.addOnItemTouchListener(object : RecyclerView.SimpleOnItemTouchListener() {
            override fun onInterceptTouchEvent(rv: RecyclerView, e: MotionEvent): Boolean {
                binding.motionLayout.onTouchEvent(e)
                return false
            }
        })
    }
}
```

`RecyclerView`には空のクリックリスナを設定したヘッダを追加することで上部余白を調整するようにしているのだが、  
これをしないで`RecyclerView`に`android:paddingTop="~~"`とかして余白を作ると、端末によってはリストの最初の項目のクリックがうまく処理できない場合があった。(原因よくわからん)  
ヘッダ部分にクリックで何かが起きるようなものを配置するとそれはうまく処理できないと思われる。

```kt:ViewPager2BindingAdapters.kt
/**
 * `ViewPager2`に選択中のメニュー項目を反映させるやつ
 */
object ViewPager2BindingAdapters {
    /**
     * 選択中メニュー項目をセットする
     */
    @JvmStatic
    @BindingAdapter("currentItem")
    fun setCurrentItem(viewPager: ViewPager2, menuItem: MenuItem?) {
        if (menuItem == null) return

        val nextIdx = MenuItem.values().indexOf(menuItem)
        if (viewPager.currentItem != nextIdx) {
            viewPager.currentItem = nextIdx
        }
    }

    /**
     * 現在表示中のページを選択中メニュー項目に反映する
     */
    @JvmStatic
    @InverseBindingAdapter(attribute = "currentItem")
    fun getCurrentItem(viewPager: ViewPager2) : MenuItem =
        MenuItem.values().getOrElse(viewPager.currentItem) { MenuItem.GENERAL }

    /**
     * 双方向バインド用の設定
     */
    @JvmStatic
    @BindingAdapter("currentItemAttrChanged")
    fun bindListeners(viewPager: ViewPager2, listener: InverseBindingListener) {
        viewPager.setPageTransformer { _, _ ->
            listener.onChange()
        }
    }
}
```

```kt:MenuItem.kt
/**
 * ページ遷移用メニュー項目
 */
enum class MenuItem(
    @StringRes val labelId : Int,
    @DrawableRes val iconId : Int,
    val fragment : ()->Fragment
) {
    GENERAL(
        R.string.prefs_menu_label_generals,
        R.drawable.ic_settings,
        { GeneralPrefsFragment.createInstance() }
    ),

    APPLICATIONS(
        R.string.prefs_menu_label_applications,
        R.drawable.ic_apps,
        { InstalledApplicationsFragment.createInstance() }
    ),

    WHITE_LIST(
        R.string.prefs_menu_label_while_list,
        R.drawable.ic_notifications_active,
        { Fragment() } // TODO: まだ作ってる途中
    ),

    BLACK_LIST(
        R.string.prefs_menu_label_black_list,
        R.drawable.ic_notifications_off,
        { Fragment() } // TODO: まだ作ってる途中
    ),

    INFORMATION(
        R.string.prefs_menu_label_information,
        R.drawable.ic_info,
        { InformationFragment.createInstance() }
    ),

    ;

    // 動的にメニュー項目を変化させる予定がないなら`ListAdapter<T,VH>`使わないで
    // `RecyclerView`用のもっと簡単なアダプタを使えばいいのでその場合これも要らない
    class DiffCallback : DiffUtil.ItemCallback<MenuItem>() {
        override fun areItemsTheSame(oldItem: MenuItem, newItem: MenuItem) =
            oldItem.name == newItem.name

        override fun areContentsTheSame(oldItem: MenuItem, newItem: MenuItem) =
            oldItem.name == newItem.name
    }
}
```
