---
title: "Androidアプリ開発Tips – dbファイルをぶっこ抜く→読み込む"
description: "ぶっこ抜いたRoomのDBファイルを再配置して読み込む方法"
tags: ["Android", "kotlin", "Room", "migrated_from_previous_blog"]
date: 2019-12-16T20:08:00+09:00
lastmod: 2020-02-01T03:10:11+09:00
archives: 2019/12/16
draft: false
hide_overview: true
---

## おことわり

この記事は旧ブログから引っ張ってきたものの再掲です。

## 当時の内容

そういうことしていいのだろうか……というのはともかく、Roomを使って管理しているアプリ用データベースを外部ファイルにぶっこ抜いて、そのファイルを再びアプリに突っ込み直す際の注意点。

アプリで作成されたdbファイルの保存場所は次のコードで取得できる。
ここでは例のため `localFileName = “app.db”` ということにする

```kt
// localFileNameはRoom.databaseBuilderに渡したファイル名
val dbFile: File = context.getDatabasePath(localFileName)
```

このdbFileの中身を読んでどっかに保存して、戻すときは同じファイル名で`getDatabasePath()`して返ってきたファイルに書き込みをすれば良い……と思ったが、これだけではどうやら不十分。ジャーナルモードによって一時ファイルが作成されている場合があり、`appDatabase.close()`などしてもどうもその中身はapp.dbに必ずしも即座に反映されないっぽい。

同じディレクトリに作成されている `“app.db-shm”` と `“app.db-wal”` も一緒に保存→復元する必要がある。（walモードの場合）

DBのジャーナルモードを明示的に指定する場合はDBオブジェクト作成時に以下のようにする。

```kt
appDatabase = Room.databaseBuilder(this, AppDatabase::class.java, APP_DATABASE_FILE_NAME)
    .setJournalMode(RoomDatabase.JournalMode.WRITE_AHEAD_LOGGING)
    .build()
```

また、復元先のアプリがインストール直後などでdatabaseディレクトリが作成されていない場合があるので、その時は手動で作成する必要がある。

```kt
val dir = File(dbFile.parent)
if (!dir.exists()) {
    dir.mkdir()
}
```

ファイルの置き換えはDBを閉じた状態で行い、置き換えが終わったらDBオブジェクトを作成し直す。必要に応じてMigrationを記述する。それはまた別のお話。
