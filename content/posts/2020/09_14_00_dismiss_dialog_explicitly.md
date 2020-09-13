---
title: "AlertDialogで項目選択後に勝手に閉じないようにする方法"
description: "AlertDialogでsetItems()した項目をタップしたときにdismiss()以外で勝手に閉じないようにする方法"
tags: ["Android", "kotlin", "AlertDialog", "DialogFragment"]
date: 2020-09-14T00:30:06+09:00
lastmod: 2020-09-14T00:30:06+09:00
archives:
    - 2020
    - 2020-09
    - 2020-09-14
hide_overview: false
draft: false
---

## 前提

次のようにして生成したダイアログでは、表示された項目をタップすると引数の関数を実行した後ダイアログは自動的に閉じる。  
(勿論```DialogFragment```を使用する場合も同様。)

```kt
AlertDialog.Builder(context, R.style.AlertDialogStyle)
    .setItems(labels) { dialog, which ->
        // 項目をタップしたときの処理
    }
    .create()
```

## 問題

通常これで困ることはあまりないが、非同期的な処理を行いたい場合には注意が必要になる。  
```DialogFragment```が持っている```lifecycleScope```や、```ViewModel```の```viewModelScope```を使用してコルーチンを非同期に実行すると、処理がすべて完了する前にダイアログが破棄されてコルーチンがキャンセルされる場合があるためだ。

## 解決策

このような場合、まぁ一番手っ取り早く問題を解決できそうなのは```GlobalScope```でコルーチンを実行することだが、  
次のようにして「コルーチンが完了するまでダイアログを閉じない」という方法をとることもできる。

```kt
AlertDialog.Builder(context, R.style.AlertDialogStyle)
    .setItems(labels, null)
    .show()
    .also { dialog ->
        dialog.listView.setOnItemClickListener { adapterView, view, which, l ->
            lifecycleScope.launch(Dispatchers.Main) {
                // 何かいろいろ処理

                // 完了後にダイアログを明示的に閉じる
                dismiss()
            }
        }
    }
```

## 余談

「OK」「キャンセル」などのネガティブ・ポジティブボタンの処理についても似たようにすることでボタン押下時にダイアログを閉じるかどうかを制御できる。

```kt
AlertDialog.Builder(context, R.style.AlertDialogStyle)
    .setNegativeButton(R.string.dialog_cancel, null)
    .setPositiveButton(R.string.dialog_ok, null)
    .show()
    .also { dialog ->
        dialog.getButton(DialogInterface.BUTTON_POSITIVE).setOnClickListener {
            // ポジティブ
            // dismiss()が呼ばれなければ閉じない
            dismiss()
        }

        dialog.getButton(DialogInterface.BUTTON_NEGATIVE).setOnClickListener {
            // ネガティブ
            // dismiss()が呼ばれなければ閉じない
            dismiss()
        }
    }
```
