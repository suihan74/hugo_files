---
title: "startActivityForResult()を新APIに置き換える"
description: "ActivityResultLauncherの基本的な使い方"
tags: ["Android","Kotlin","Activity","Fragment","ActivityResultLauncher"]
date: 2021-02-25T14:11:51+09:00
archives:
    - 2021
    - 2021-02
    - 2021-02-25
hide_overview: false
draft: false
---

## fragment-ktx

`androidx.fragment:fragment-ktx`を`1.2.5`から`1.3.0`に更新した結果(というより、その内部で依存している`androidx.activity:activity-ktx`が`1.2.0`に変更された結果)、`Activity`や`Fragment`の扱い方に関する幾つかの変更が発生した。

今回の内容もそのひとつであり、「`onActivityResult`が非推奨になったので`ActivityResultLauncher`使ってくれや」という問題に対応する。

### 他に対応したこと

[(いまさら)ViewPagerからViewPager2へ移行する](/posts/2021/02_23_00_migrate_to_view_pager_2)

---

## startActivityForResult + onActivityResult

「他の`Activity`に遷移して行った操作の結果を遷移前の`Activity`や`Fragment`で受け取る」ということがしたい場合、  
これまでは`startActivityForResult(intent)`で画面遷移し、`onActivityResult(requestCode, resultCode, intent)`をオーバーライドして結果を受け取って処理する、というようにしていた。

```kt:これまでの方法.kt
class HogeActivity : AppCompatActivity {
    // ...

    /** リクエストコード */
    enum class RequestCode {
        FOO,
        BAR,
        BAZ
    }

    // ------ //

    fun startPiyoActivity() {
        val intent = Intent(this, PiyoActivity::class.java)
        startActivityForResult(intent, RequestCode.FOO.ordinal)
    }

    // ------ //

    /** 結果を受け取る */
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        if (resultCode != RESULT_OK) {
            // failed
        }
        when (requestCode) {
            RequestCode.FOO.ordinal -> {
                val msg = data?.getStringExtra(PiyoActivity.ResultExtra.MESSAGE.name)!!
                Log.i("piyo result", msg)
            }

            else -> ...
        }
    }
}
```

```kt:結果を返す.kt
class PiyoActivity : AppCompatActivity() {
    // ...

    enum class ResultExtra {
        MESSAGE
    }

    fun finishActivity() {
        val intent = Intent().apply {
            putExtra(ResultExtra.MESSAGE.name, "piyo")
        }
        setResult(RESULT_OK, intent)
        finish()
    }
}
```

## ActivityResultLauncher

これからは`ActivityResultLauncher`を使用した方法が推奨となる。  
上記コードをひとまず機械的に置換した例が次になる。

```kt:新しい方法でアクティビティ遷移.kt
class HogeActivity : AppCompatActivity() {
    // ...

    /**
     * Intentを開始するためのランチャ
     * STARTEDライフサイクル前に作成する
     */
    private val launcher = registerForActivityResult(
        contract = ActivityResultContracts.StartActivityForResult()
    ) { result -> // 結果を受け取る関数
        if (result.resultCode != RESULT_OK) {
            // failed
        }
        val msg = result.data?.getStringExtra(PiyoActivity.ResultExtra.MESSAGE.name)!!
        Log.i("piyo result", msg)
    }

    // ------ //

    fun startPiyoActivity() {
        val intent = Intent(this, PiyoActivity::class.java)
        launcher.launch(intent)
    }
}
```

なお、公式のドキュメント[アクティビティからの結果の取得 | Android デベロッパー | Android Developers
](https://developer.android.com/training/basics/intents/result?hl=ja)では`prepareCall()`を使用しているが、どうも情報が古いまま掲載されているようで、現時点(2021-02-25)ではこれは上記のように`registerForActivityResult()`を使用するように変更されている。

### ActivityResultContract<I,O>

要するに「何(I)を渡して`Intent`を呼んで、何の結果(O)を得るか」を表している。

`ActivityResultContracts.StartActivityForResult`の場合は、`ActivityResultContract<Intent, ActivityResult>`を継承しており、「任意の`Intent`を渡して画面遷移をし、結果を`ActivityResult`で受け取る」ということになる。

他にも基本的なIO用のコントラクトは用意されているので、用途に応じて選択する。

[ActivityResultContracts | Android デベロッパー | Android Developers](https://developer.android.com/reference/androidx/activity/result/contract/ActivityResultContracts)

`ActivityResultContract<I,O>`を継承して独自のコントラクトを作成することもできる。

[カスタム コントラクトを作成する | アクティビティからの結果の取得 | Android デベロッパー | Android Developers](https://developer.android.com/training/basics/intents/result?hl=ja#custom)

当記事で使用した`ActivityResultContracts.StartActivityForResult`はあまり推奨されなくて、用途ごとに適切なデフォルトコントラクトやカスタムコントラクトを作成・選択していくべきなのかな、と感じた。

### 呼び出しについて

`registerForActivityResult`は`ComponentActivity`や`Fragment`に生えている。

そのライフサイクルが`STARTED`になる前であれば新しくランチャを登録することができ、`CREATED`以降であればランチャを実行することができる。

また、結果を別クラスで受け取ることも可能で、公式ドキュメントでは`ActivityResultRegistry`を渡した`LifecycleObserver`を作ってオブザーバ側で結果を受け取る例が紹介されている。

```kt:公式ドキュメントから引用.kt
class MyLifecycleObserver(private val registry : ActivityResultRegistry)
        : DefaultLifecycleObserver {
    lateinit var getContent : ActivityResultLauncher<String>

    fun onCreate(owner: LifecycleOwner) {
        getContent = registry.register("key", owner, GetContent()) { uri ->
            // Handle the returned Uri
        }
    }

    fun selectImage() {
        getContent("image/*")
    }
}

class MyFragment : Fragment() {
    lateinit var observer : MyLifecycleObserver

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...

        observer = MyLifecycleObserver(requireActivity().activityResultRegistry)
        lifecycle.addObserver(observer)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val selectButton = view.findViewById<Button>(R.id.select_button)

        selectButton.setOnClickListener {
            // Open the activity to select an image
            observer.selectImage()
        }
    }
}
```
