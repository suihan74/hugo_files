---
title: "RecyclerViewロード時にスクロール位置を調整する"
description: "ListAdapterを使ってアイテムをロードしたときに初期スクロール位置が変な位置になることがあったので修正した。"
tags: ["Android", "kotlin", "RecyclerView"]
date: 2020-02-22T14:10:58+09:00
lastmod: 2020-02-24T14:45:00+09:00
archives:
    - 2020
    - 2020-02
    - 2020-02-22
draft: false
---

## 追記 (2020/02/24 14:45)

真ん中にスクロールしてしまう原因がフラグメントの初期化周りがぐちゃぐちゃでアイテムの渡し方がおかしなことになっていることだったので、  
「普通に使っていればこの記事の問題は発生しないのでは？？？ということがわかった。

具体的には初期化の直後に追加ロードが発生してしまっていたようだった。

なので、以下の内容は今回の問題の解決方法というよりも「ListAdapter使っているときにアイテム挿入を監視して何かしたいときのやり方」くらいに読んでもらいたい。（雑）

そしてはやいところ問題の画面を作り直すのです……という圧が強まった。

---

## 前の記事

[Android - ListAdapter<T, VH>をつかう](/posts/2020/02_17_00_list_adapter/)

## 発生した問題

最初のリスト登録時にスクロール位置がリストの最初ではなく真ん中あたりになってしまうことがあった。

## 解決した方法

adapterもRecyclerViewも見える場所(ActivityとかFragmentとか)で次のように`AdapterDataObserver`を登録する。

```kt
val adapter = HogeAdapter().apply {
    registerAdapterDataObserver(object : RecyclerView.AdapterDataObserver() {
        override fun onItemRangeInserted(positionStart: Int, itemCount: Int) {
            recyclerView.scrollToPosition(positionStart)
        }
    })
}
```

これで項目が挿入された直後にこの`onItemRangeInserted()`が呼ばれる。`positionStart`は挿入開始位置なので、明示的にこの位置にリストをスクロールするように指定すればよい。

ひとつ問題があるとすれば、追加ロードなどで挿入が発生した場合に毎回強制的にスクロールしてしまうので場合によっては少しうざったいかもしれない。  
ユーザーの操作と無関係にリスト内容が変更される可能性がある場合には他の方法を探した方がいいかもしれない。
