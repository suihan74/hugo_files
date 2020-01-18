---
title: "結果を返すActivityを呼ぶ"
description: startActivityForResult()で呼んでonActivityResult()で結果を受け取る。
tags: ["android", "kotlin", "activity"]
date: 2020-01-07T03:02:05+09:00
lastmod: 2020-01-07T03:02:05+09:00
archives:
    - 2020
    - 2020/01
    - 2020/01/07
draft: false
---

`Activity`から戻ってくる際にその結果を受け取る必要がある場合のやりかた。  
こういうネタ何書いても今さら感しかないわけだけど、しかし作っているもの的にこれまであまり必要なシーンも無かったので、備忘。

# 1. 結果を返すActivityの呼び方

`startActivity(intent)`の代わりに`startActivityForResult(intent: Intent, requestCode: Int)`を使用する。

```kt
startActivityForResult(intent, HogeActivity.REQUEST_CODE)
```

`requestCode`はInt型だが**16bit範囲内の値**である必要がある[^1]。  
これを守らないと`IllegalArgumentException`投げられる。  
定数値以外を使う場合0xffffでマスクするなりしてナントカする。

## Fragmentから呼んで、返ってきた結果をActivityで処理する場合

`Fragment.startActivityForResult(...)`を使用すると`Activity.onActivityResult(...)`に渡される`requestCode`が意図しないものになる。

以下のどちらかで対応。

- `Activity`で結果を受け取りたいなら`activity?.startActivityForResult(...)`とかに書き換える。
- `Fragment.onActivityResult(...)`で受け取る。


# 2. Activityから結果を返して終了する

`Activity.finish()`を呼ぶ前に`resultCode`と結果を返すための`Intent`をセットする。

```kt
val intent = Intent().apply {
    putExtra(RESULT_DATA, result)
}
setResult(Activity.RESULT_OK, intent)
finish()
```

成功なら`Activity.RESULT_OK`、失敗なら`Activity.RESULT_CANCELED`があるのでそれを使えばそれで良い気がする。  
何も指定しないと`Activity.RESULT_CANCELED`が返ってくるっぽい。

# 3. 返ってきた結果を受け取る

```kt
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)

    when (requestCode) {
        HogeActivity.REQUEST_CODE -> {
            when (resultCode) {
                Activity.RESULT_OK -> {
                    val result = data?.getSerializableExtra(HogeActivity.RESULT_DATA) as? HogeResult
                    ...
                }

                Activity.RESULT_CANCELED -> {
                    ...
                }
            }
        }
    }
}
```

この辺もう少し簡潔に書くための試みが色々あるようだけど今回はこれで十分な規模だったので割愛。  
必要になったら調べる。

[^1]: [[備忘録]AndroidでFragmentActivityのrequestCodeの範囲　(In-app Billingなどで使う、start～ForResult()などに影響。) - Qiita](https://qiita.com/toris-birds/items/7deae66318e093b42ee6)
