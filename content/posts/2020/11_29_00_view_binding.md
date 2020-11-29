---
title: "ViewBindingの使い方と旧手法からの移行"
description: "Viewの参照方法が(多分随分前に)新しくなっていた件"
tags: ["Android", "Kotlin", "ViewBinding", "Kotlin Android Extensions", "synthetics"]
date: 2020-11-29T15:40:00+09:00
lastmod: 2020-11-29T15:40:00+09:00
archives:
    - 2020
    - 2020-11
    - 2020-11-29
hide_overview: false
draft: false
---

Kotlinを`1.4.20`にアップデートしたら「`'kotlin-android-extensions'`は非推奨になったからよ、`ViewBinding`使ってくれや」というメッセージが出てきて初めて気づいたやつ。

[Migrate from Kotlin synthetics to Jetpack view binding](https://developer.android.com/topic/libraries/view-binding/migration)

[ビュー バインディング | Android デベロッパー | Android Developers](https://developer.android.com/topic/libraries/view-binding)

公式のリファレンスが普通に読みやすいのでとくに書くこと無いのだが、`Kotlin synthetics`からの移行作業が少々面倒だったのでその記録。

なお、`synthetics`は2021年9月かそのくらいには削除されるみたい。

[Google Developers Japan: Kotlin Android Extensions の未来](https://developers-jp.googleblog.com/2020/11/the-future-of-kotlin-android-extensions.html)

---

## findViewById

コード側から`View`を参照する原始的な方法は`findViewById(id)`メソッドを使用することである。

```xml
<Button
    android:id="@+id/hoge"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

```kt
val button = activity.findViewById<Button>(R.id.hoge)
```

### 問題点

- **レイアウトの変更時にコードを直し忘れると実行するまで気づかない**

  先例の場合、IDが変更されたり`Button`以外に変更したり、という変更を施した場合、コードの方を修正していなくてもビルドまで通ってしまう。

- **存在しないIDを参照できる**

  同根の問題だが、対象のレイアウトには存在しない`View`のIDを参照しようとすることができてしまう。(結果はnull)

- **型安全ではない、null安全ではない**

  要するに以上の問題はそういうこと。扱おうとしている型が正しいかどうか分からない、参照先が存在しないかもしれない。結果が`T!型`なのでKotlinでは`nullable`として扱うのが最も安全だが、その辺の判断が開発者に委ねられるふわっと感がある。

- **結果がキャッシュされない**

  呼び出すたびに子を順番に探索して指定した`View`を取得し直す。(自分で参照結果を保持しておく必要がある)

- **いちいち長い**

  型をこちらで指定して、IDを間違いなく指定して、findViewById

## kotlin synthetics

`Kotlin Android Extensions`に含まれる`View`参照を拡張プロパティ化する仕組み。  
レイアウトファイルごとに自動生成されるコードをインポートすることで、`Activity`や`View`の子`View`を拡張プロパティとして参照することができるようになる。

```kt
import kotlinx.android.synthetic.main.activity_main.*

val button = activity.hoge
```

### 問題点

- **どこからでも参照できる**

  たとえば`Activity`は基本的にはアタッチされた`Fragment`側から参照することができるが、`Fragment`側のコードで`Activity`のレイアウト用の拡張をインポートすれば、`Activity`のすべての`View`に直接アクセスすることができる。(開発者が参照を間違う可能性も当然ある)  
  場合によっては楽でいいが、外に見せる必要のないものは見えないようにしておくべきである。

- **関係ないレイアウト用の拡張をインポートできる**

  同じIDの`View`がある複数のレイアウトファイルが存在する場合、コード補完時にそのファイル分だけインポート候補が表示されるので、うっかりミスる可能性はある。(早々間違えないが可能性はゼロではない)

- **参照結果が一部キャッシュされない場合がある(らしい)**

- **拡張プロパティである**

  故にKotlinでしか使えない(Javaで使えない)。これは個人的にはどうでもいい。  
  拡張プロパティなのでインポートさえすれば`Activity`やら`View`やらにいくらでも生えて煩雑になる。

- **`View`のIDをスネークケースで書いている場合、プロパティ命名規則的にキモい**

  IDを`@+id/hoge_button`とかしているとそれがそのままプロパティ名になるので、基本キャメルケースで書いてあるはずのコードに`root.hoge_button`とかを書かなくてはならなくなる。(それかIDを全て書き直すことになる)  
  結果的に「これは`View`の参照である」という目印になってある意味分かりやすくもあるが。

## ViewBinding

(アプリ開発者から見える表面上の)コード側では`DataBinding`とほぼ同じように`View`参照が扱える仕組み。

### `Activity`の場合

```kt
override fun onCreate(savedInstanceState: Bundle) {
    super.onCreate(savedInstanceState)

    val binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    val button = binding.hoge
}
```

### `Fragment`の場合

```kt
override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
) : View {
    val binding = FragmentHogeBinding.inflate(
        inflater,
        container,
        false
    )

    val button = binding.hoge

    return binding.root
}
```

### `Fragment`で`binding`を保持し続ける場合

```kt
private var _binding : FragmentHogeBinding? = null
private val binding
    get() = _binding!!

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
) : View {
    _binding = FragmentHogeBinding.inflate(
        inflater,
        container,
        false
    )

    val button = binding.hoge

    return binding.root
}

/**
 * FragmentはViewが破棄されても生き続ける場合があるので、
 * Viewが破棄された時点で参照を解く
 */
override fun onDestroyView() {
    super.onDestroyView()
    _binding = null
}
```

### 利点

- **レイアウトとコードで齟齬が発生しない**

  各レイアウトファイルごとに`ViewBinding`を継承した専用のクラスが生成され、それを使って紐づいたレイアウトを直接生成するので、コード側ではレイアウトリソースIDを扱う必要が一切なくなった。そのうえ`View`の参照は`binding`のプロパティを通して行うので「存在しない`View`を参照しようとする」ことは原理的に不可能になった。

- **型安全、null安全**

  先の点に関連して、参照時に「型が合っているか」「nullではないか(`T!型`)」を気にする必要がなくなった。

  画面の向き専用のレイアウトファイルを用意していて、向きによっては特定の`View`が存在しない場合には、対応するプロパティは自動的に`nullable型`になる。`T!型`ではないので`null`かどうかを気にする必要がそもそもあるのか無いのかに悩む必要がない。

- **キャッシュがはたらく**

  どのような場合でも`ViewBinding`内部でしっかり`View`参照をキャッシュしてくれる。

  自動生成されたコードを見てみると、`bind()`が呼ばれたとき(`inflate()`したとき)に内包するすべての`View`に対して`findViewById()`を行うようだ。  
  なので、コード側で参照しない`View`が沢山あると初期化時に無駄なキャッシュ処理が大量に挟まることにはなるが。  
  ちなみに、この処理が行われたとき(`inflate`or`bind`)に「あるはずの`View`が見つからない」不具合が起きていたらヌルポを飛ばしてくれる。

- **`binding`の扱い方が`DataBinding`とほぼ可換**

  「途中まで`ViewBinding`で書いていたが、動的な値更新がある箇所が増えたから`DataBinding`に変えたい」とかその逆とかで書き換えたくなった時に、コード表面上での取り扱い方はほぼ同じなので移行が楽。

  ちなみに「`DataBinding`使えるならそっち使えばいいじゃんMVVMでいいじゃん」については、値更新がない(少ない)場合などについては`ViewBinding`の方が(MVC的にやった方が)軽量でシンプルであるという利点はある。

### 自動生成されたコードがある場所

`(ProjectRoot)/app/build/generated/data_binding_base_class_source_out/debug/out/(com)/(hoge)/(appName)/databinding/~Binding.java`

AndroidStudioで(`ViewBinding` `DataBinding`問わず)`HogeBinding`クラスの宣言を開こうとするとレイアウトファイルが開かれるため、これらのファイルは普通直接開くことはない。こういう記事を書くとかしなければ普通にそれがありがたい。

---

## syntheticsからViewBindingへの移行

これから新規にプロジェクトを始める場合には`ViewBinding`+`DataBinding`で始めればいいのだが、既に`synthetics`を使用していた場合の移行作業は少々手間だった。

行った作業を以下に書いておく。

### アプリレベルの`build.gradle`を修正

[Migrate from Kotlin synthetics to Jetpack view binding](https://developer.android.com/topic/libraries/view-binding/migration)

1. 以下を削除

    ```gradle
    apply plugin: `kotlin-android-extensions`

    androidExtensions {
        experimental = true
    }
    ```

2. 以下を追加

    ```gradle
    android {
        ...
        buildFeatures {
            viewBinding true
            ...
        }
    }
    ```

### `synthetics`のインポートをすべて削除する

```kt
import kotlinx.android.synthetic.
```

を検索してすべて削除。ヒットしたファイルで`synthetic`が使用されているので、次項からの修正を行う。

### `Activity#setContentView()`を置き換える

`Activity#onCreate()`内でのレイアウト生成部分を書き換えた。

- 移行前

    ```kt
    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate()
        setContentView(R.id.activity_hoge)
    }
    ```

- 移行後

    ```kt
    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate()
        val binding = ActivityHoge.inflate(layoutInflater)
        setContentView(binding.root)
    }
    ```

### `LayoutInflater#inflate()`を置き換える

`Fragment`や`ViewHolder`、カスタムビューなどで`LayoutInflater#inflate()`していた部分をすべて`ViewBinding#inflate()`に書き換えた。

- 移行前

    ```kt
    val inflater = LayoutInflater.from(activity)
    val root = inflater.inflate(R.layout.fragment_hoge, container, false)
    ```

- 移行後

    ```kt
    val inflater = LayoutInflater.from(activity)
    val binding = FragmentHogeBinding.inflate(inflater, container, false)
    val root = binding.root
    ```

### `View`参照を置き換える

IDをスネークケースでつけている場合、それらをすべてローワーキャメルケースで書き直す必要がある。

- 変更前

    `Activity`の場合

    ```kt
    val hoge = hoge_button
    ```

    `Fragment`などで`View`の子を参照している場合

    ```kt
    val hoge = root.hoge_button
    // or
    val hoge2 = view?.hoge_button!!
    ```

- 変更後

    ```kt
    val hoge = binding.hogeButton
    ```

    すべての箇所を`binding`を経由して参照するように書き直す必要があるので、場合によっては`Activity`や`Fragment`に`binding`をプロパティとして保持するように書き足すとか、`Activity#onCreate`やら`Fragment#onCreateView`やらから他のメソッドを呼ぶ際の引数にする必要がある。

### 外部からの直接的な`View`参照を修正する

(そもそもそんなことするなよというのは置いておく)

- 変更前

    ```kt
    // たとえば子のフラグメント側で
    val hogeActivity = requireActivity() as HogeActivity
    val activityToolbar = hogeActivity.toolbar
    ```

- 変更後

    `Activity`

    ```kt
    class HogeActivity : AppCompatActivity() {
        private lateinit var binding : ActivityHogeBinding
        ...
        val toolbar : Toolbar
            get() = binding.toolbar
        ...
    }
    ```

    子`Fragment`

    ```kt
    val hogeActivity = requireActivity() as HogeActivity
    val activityToolbar = hogeActivity.toolbar
    ```
