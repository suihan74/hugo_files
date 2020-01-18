---
title: "外部からprivateなsuspendメソッドを実行する方法"
description: private suspendなメソッドをリフレクションで取得して非同期実行する方法。
tags: ["kotlin", "coroutine"]
date: 2020-01-09T17:08:19+09:00
lastmod: 2020-01-09T17:08:19+09:00
archives:
    - 2020
    - 2020/01
    - 2020/01/09
draft: false
---

プライベートメソッドのテストを書きたいときなどの方法。  
`Class.getDeclaredMethod(...)`を使ってプライベートメソッドを取得して実行する際に気を付けなければいけないことが通常のメソッドに比して少し増える。

# 実行したいメソッド

たとえば以下のようなメソッドをリフレクションでぶっこ抜いてきて実行したいとする。

```kt
class Hoge {
    private suspend fun hoge(foo: Int, bar: String) = withContext(Dispachers.IO) {
        ...
    }
}
```

# メソッドの取得方法

```kt
val hogeMethod = Hoge::class.java.getDeclaredMethod(
        "hoge",
        Int::class.java,
        String::class.java,
        Continuation::class.java
    ).apply {
        isAccessible = true
    }
```

引数型の最後に`Continuation::class.java`を記述する必要がある。

# 取得したメソッドの実行方法

次のような拡張メソッドを用意しておく（とまぁ便利）。

```kt
suspend fun Method.invokeSuspend(obj: Any, vararg args: Any?) : Any? =
    suspendCoroutineUninterceptedOrReturn { cont ->
        invoke(obj, *args, cont)
    }
```

インスタンスと引数を渡して実行する。

```kt
hogeMethod.invokeSuspend(instance, 0, "hoge")
```

テストでこの処理結果を受けて`assertEquals(...)`とかしたい場合は、  
runBlockingするなりして終了を待機しておけばいいんだろうか。  
いい感じの方法が他にあれば知りたい。

---

参考  
[java - How to run suspend method via reflection? - Stack Overflow](https://stackoverflow.com/questions/47654537/how-to-run-suspend-method-via-reflection)

---

## 余談

`Continuation<ResultT>`を作って`method.invoke(...)`の引数に渡すことも可能と言えば可能。

```kt
val continuation = Continuation<HogeResult>(coroutineContext) { result ->
    // hogeMethodの完了後呼ばれる
    ...
}
hogeMethod.invoke(instance, 0, "hoge", continuation)
```

この場合hogeMethodは待機されないのでそのままではうまくテストできなくて困るのだが。
