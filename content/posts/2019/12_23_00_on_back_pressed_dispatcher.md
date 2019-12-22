---
title: "OnBackPressedDispatcherのコールバック追加タイミングに関するTips"
description: タイミングを誤ると画面の一番上にあるFragmentのコールバックが呼ばれないので注意が必要。
tags: ["android", "kotlin", "OnBackPressedDispatcher"]
date: 2019-12-23T01:18:10+09:00
lastmod: 2019-12-23T01:18:10+09:00
draft: false
---

# おさらい
- Fragmentの戻るボタン処理についてのお話。
- `AppCompatActivity.onBackPressed()`をオーバーライドしてごちゃごちゃやるよりは`AppCompatActivity.onBackPressedDispatcher.addCallback()`を使うのがナウい。
- `addCallback(owner: lifecycleOwner,...)`は最後に追加されたものでかつ有効なものだけが処理される。
- `AppCompatActivity.onBackPressed()`の中で`onBackPressedDisptacher.onBackPressed()`を呼ぶようになっているので、自前のActivityでオーバーライドした場合その中で`super.onBackPressed()`を呼ばないとコールバックは全て無視される。

# 問題
>`addCallback(owner: lifecycleOwner,...)`は最後に追加されたものでかつ有効なものだけが処理される。

1. Activity側の`onCreate()`内でActivityの戻るボタン処理を`addCallback()`する。
2. `onCreate()`内でFragmentを作成してsupportFragmentManagerにaddする。
3. Fragment側では`onCreateView()`内でFragmentの戻るボタン処理を`addCallback()`する。

という3ステップを踏んだ場合、どうも戻るボタンコールバックのスタックに積まれる順番は想定していた `1:Activity → 2:Fragment` ではなく、`1:Fragment → 2:Activity` になる。

# 原因

[公式リファレンス / OnBackPressedDispatcher](https://developer.android.com/reference/androidx/activity/OnBackPressedDispatcher) には次のように記述がある。

>Receive callbacks to a new OnBackPressedCallback when the given LifecycleOwner is at least started.

`onStart()`が呼ばれる順番は `1:Activity → 2:Fragment` じゃなかったかしらと思ったら、
LifecyclerStateがSTARTEDになるのは`onStart()`が呼ばれたときではなく**抜けたとき**なのであった。

>STARTED  
>Started state for a LifecycleOwner. For an Activity, this state is reached in two cases:  
>- after onStart call;  
>- right before onPause call.

[公式リファレンス / Lifecycle.State](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State.html#STARTED)

Activity.onStart()内でFragment.onStart()が呼ばれるので、つまりLifecycle.StateがSTARTEDになる順番は `1:Fragment → 2:Activity` となる。

戻るボタンコールバックはSTARTEDになった順に積まれるので、`Fragment.onCreateView()`とかでコールバックを追加するとActivityのコールバックの方が後に積まれることになり、結果的にFragmentのコールバックが無視されたような動作になる。

# 解決
というわけで、`Activity.onStart()`が完了する前にFragmentを作成してアタッチする場合、Fragment側のコールバックを登録するコードは`Fragment.onResume()`～のタイミングで行うようにしましょうということらしい。
