---
title: "Android - ListAdapter<T, VH>をつかう"
description: "リストの更新処理をユーティリティ頼りにする。そして非同期的に行う。"
tags: ["Android", "kotlin", "recyclerview"]
date: 2020-02-17T18:52:19+09:00
lastmod: 2020-02-17T18:52:19+09:00
archives: 2020/02/17
draft: false
---

[以前の記事](/posts/2019/12_16_01_android_diff_util/)で[DiffUtil](https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil.html)の使い方をメモして、非同期にやるやつ([AsyncListDiffer](https://developer.android.com/reference/androidx/recyclerview/widget/AsyncListDiffer))についてはあまり触れていなかったのでそいつの使い方を書く。

## [ListAdapter<T, VH>](https://developer.android.com/reference/androidx/recyclerview/widget/ListAdapter.html)

### なにそれ

内部で[AsyncListDiffer](https://developer.android.com/reference/androidx/recyclerview/widget/AsyncListDiffer)を使ってくれるRecyclerView.Adapter。もうこれでいいじゃんみたいな気持ちになる。

`submitList()`メソッドが`public`な点にモニョっときたら[AsyncListDiffer](https://developer.android.com/reference/androidx/recyclerview/widget/AsyncListDiffer)を自前で用意すればいいのかもしれない。

どちらを使用するにしても手間はそれほど変わらないように感じた。

### 使い方

```kt
// T: リスト項目の型, VH: ViewHolder
class HogeAdapter : ListAdapter<Item, RecyclerView.ViewHolder>(DiffCallback()) {
    private class DiffCallback : DiffUtil.ItemCallback<Item>() {
        // 同じ項目か判別する
        override fun areItemsTheSame(
            oldItem: Item,
            newItem: Item
        ): Boolean {
            return oldItem.id == newItem.id
        }

        // 項目の内容がアップデートされているかを判別する
        override fun areContentsTheSame(
            oldItem: Item
            newItem: Item
        ): Boolean {
            return oldItem == newItem
        }
    }

    // 現在のリストはcurrentListを参照する
    override fun getItemCount() = currentList.size

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int) =
        LayoutInflater.from(parent.context).let { inflater ->
// ...省略
        }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
// ...省略
    }

}
```

リストを更新したりアイテムを追加・削除したりするときは、**更新後のリスト**をsubmitList()に渡す。

```kt
adapter.submitList(newList)
```

渡した後で加工して表示したいなどの需要がある場合、なんかメソッド生やしたりする。`submitList`を外に見せたくない場合は上に書いたように`ListAdapter`ではなく[AsyncListDiffer](https://developer.android.com/reference/androidx/recyclerview/widget/AsyncListDiffer)使う。
