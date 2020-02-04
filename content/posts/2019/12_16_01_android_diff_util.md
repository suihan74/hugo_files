---
title: "Androidアプリ開発Tips – DiffUtilでリスト差分更新"
description: "RecyclerViewの更新にDiffUtil使うと楽でもあり苦でもあるみたいな話"
tags: ["Android", "kotlin", "migrated_from_previous_blog"]
date: 2019-12-16T22:21:00+09:00
lastmod: 2020-02-01T03:10:11+09:00
archives:
    - 2019
    - 2019/12
    - 2019/12/16
draft: false
hide_overview: true
---

## おことわり

この記事は旧ブログから引っ張ってきたものの再掲です。

## 追記

[DiffUtil](https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil)をそのまま使うと、アイテム数が膨大になったときUIスレッドを止めてしまう可能性がある。なので[AsyncListDiffer](https://developer.android.com/reference/androidx/recyclerview/widget/AsyncListDiffer)を使ってバックグラウンドでよろしくやりましょうというようなことがリンク先に書いてある。

## 当時の内容

`RecyclerView.Adapter`に項目を追加削除していくときに、いちいち自分でリストへの挿入削除書いてnotifyナンタラ呼んで……っていうことは必要ないらしく、[DiffUtil](https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil)を使うと楽でしょうという話。

Androidアプリ作り始めた三ヶ月ほど前にも（それどころか数年前から）余裕であったけど知らなかったよね。知らないというのは罪だ。勉強しろ。と思った。思ってばかりいる。そして今日も罪を産み続けている。

```kt
open class UsersAdapter : RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    private var states = RecyclerState.makeStatesWithFooter(emptyList<User>())
    fun setItems(users: List<User>) {
        val newStates = RecyclerState.makeStatesWithFooter(users)
        val result = DiffUtil.calculateDiff(object : DiffUtil.Callback() {
            override fun getOldListSize() = states.size
            override fun getNewListSize() = newStates.size
            // 同じ項目を探すやつ（追加・削除・更新のどれかを判別）
            override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
                val old = states[oldItemPosition]
                val new = newStates[newItemPosition]
                return old.type == new.type && old.body?.id == new.body?.id
            }
            // 項目の中身が同じかを判別するやつ（同一項目の中身が更新されたかを確認）
            override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
                val old = states[oldItemPosition]
                val new = newStates[newItemPosition]
                return old.type == new.type &&
                        old.body?.id == new.body?.id &&
                        old.body?.name == new.body?.name
            }
        })
        states = newStates
        result.dispatchUpdatesTo(this)
    }
    // ほか省略...
}
```

リスト表示を更新したくなったら、最新状態のリストをポンと丸ごと渡すだけでいい。楽。

ちなみに、`RecyclerState`はデータ項目以外の項目（ヘッダやフッタ）をリストに挿入するためのものとして別に用意したもの。今回はあまり関係ない。
項目が何を表しているかを示す`type: RecyclerType`と、データ項目ならその中身である`body: T?`をもつ。
