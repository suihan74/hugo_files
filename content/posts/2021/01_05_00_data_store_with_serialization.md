---
title: "設定データをKotlin Serialization + DataStoreで扱う"
description: ""
tags: ["Kotlin","Android","Jetpack","serialization","DataStore","Flow"]
date: 2021-01-05T00:45:00+09:00
lastmod: 2021-01-05T00:45:00+09:00
archives:
    - 2021
    - 2021-01
    - 2021-01-05
hide_overview: false
draft: false
---

この記事は`DataStore 1.0.0-alpha05`,`Kotlin Serialization 1.0.1`について書かれています。  
(とくに`DataStore`の方はアルファ版なので)アップデートにより最新バージョンでは記事内容が合わなくなる可能性があります。

## DataStore

`SharedPreferences`に代わり良い感じにデータ永続化するやつとして`DataStore`が開発されている。

[DataStore | Android デベロッパー | Android Developers](https://developer.android.com/topic/libraries/architecture/datastore)

`SharedPreferences`といえば、データの出し入れの際の型の扱いがふわっとしていたり、キーがただの文字列だったり、非同期のこととかはとくに考えられていなかったりで、がっちり使い込むには自分でラッパを書いたりする必要があった。

`DataStore`の仕組みで実装されている`Preferences DataStore`を使うと、`SharedPreferences`相当のことがとりあえず少しは良い感じになる。

```kt:PreferencesDataStoreを開く.kt
val dataStore: DataStore<Preferences> = context.createDataStore(
    name = "settings"
)
```

```kt:読み込み.kt
val fooKey = preferencesKey<Int>("foo")
val fooFlow: Flow<Int> = dataStore.data
    .map { prefs ->
        prefs[fooKey] ?: 0
    }
```

```kt:書き込み.kt
val fooKey = preferencesKey<Int>("foo")
coroutineScope.launch {
    dataStore.edit { prefs ->
        prefs[fooKey] = 334
    }
}
```

ただこれはこれでキーの扱いが面倒臭いとか、プリミティブ型じゃないオブジェクトを簡単に扱えないとか、色々問題がある。

リンク先の解説では`Protocol Buffers`を使って`Proto DataStore`もできるよとか書いてあるが、Kotlin書く上で楽するために`DataStore`使うのにprotobufでデータ定義を別に記述してどうのこうのとか正直やりたくない。

そこで、`Kotlin Serialization`でオブジェクトをJsonにシリアライズして保存して、入出力インタフェースとして`DataStore`を使用する方法を使う。

## Kotlin Serialization + DataStore

`Proto DataStore`はどうもバイト列を入出力できれば必ずしも`Protocol Buffers`を必要としていないようなので、シンプルにJsonシリアライズした文字列をバイト列化して保存するようにする`DataStore`シリアライザを用意する。

参考

- [Kotlin/kotlinx.serialization: Kotlin multiplatform / multi-format serialization](https://github.com/Kotlin/kotlinx.serialization)

- [Jetpack DataStoreをProtobufではなくKotlin Serializationで使用する](https://zenn.dev/kafumi/articles/377bca464613f3)

ここではざっくりと結果だけ書くので、詳しいことは参照先を読んだ方がいいです。

```kt:Hoge.kt
/**
 * 雑なサンプル用設定データクラス
 *
 * すべてのプロパティにデフォルト値を記述しておくと良さげ
 * `data class`で作っておくと更新処理で`copy`メソッドが使えて楽
 */
@Serializable
data class Hoge(
    val foo : Int = 0,
    val bar : String = "default value",

    /** ユーザークラスには別途シリアライザを用意する必要がある */
    @Serializable(with = LocalTimeSerializer::class)
    val baz : LocalTime = LocalTime.MIN
)
```

`Kotlin Serialization`用のシリアライザは、外部のライブラリなどから持ってきているデータなら上記のようにプロパティで指定すれば良いし、自分で用意したクラスであるならクラス宣言に書いてしまっても良い。  
他に`Json`インスタンスの`serializersModule`にまとめてシリアライザを渡してしまって、プロパティには`@Contextual`アノテーションをつける方法もあるが、あんま良い感じはしない。

次コードがユーザークラス用のJsonシリアライザ例。

```kt:LocalTimeSerializer.kt
/**
 * `LocalTime`を`Long`値に変換するシリアライザ
 */
class LocalTimeSerializer : KSerializer<LocalTime> {
    override val descriptor: SerialDescriptor
        get() = PrimitiveSerialDescriptor(
            LocalTimeSerializer::class.qualifiedName!!,
            PrimitiveKind.LONG
        )

    override fun serialize(encoder: Encoder, value: LocalTime) {
        encoder.encodeLong(value.toSecondOfDay().toLong())
    }

    override fun deserialize(decoder: Decoder): LocalTime {
        return LocalTime.ofSecondOfDay(decoder.decodeLong())
    }
}
```

`PrimitiveSerialDescriptor`に渡す名前はユニークであればなんでも良いっぽいので、適当にシリアライザ自身のクラス名を渡している。

次コードは`DataStore`がデータをファイルに入出力する際に使用するシリアライザ例。  
ここで`Kotlin Serialization`を使って文字列化、それをさらにバイト列化して読み書きする。

```kt:HogeSerializer.kt
/**
 * `Hoge`クラスのインスタンスを`DataStore`がファイルに入出力するためのシリアライザ
 */
@OptIn(ExperimentalSerializationApi::class)
class HogeSerializer(
    private val stringFormat: StringFormat = Json
) : Serializer<Hoge> {

    // すべてのプロパティがデフォルト値なインスタンスを生成する
    override val defaultValue : Hoge
        get() = Hoge()

    override fun writeTo(value: Hoge, output: OutputStream) {
        val string = stringFormat.encodeToString(value)
        val bytes = string.encodeToByteArray()
        output.write(bytes)
    }

    override fun readFrom(input: InputStream) : Hoge {
        try {
            val bytes = input.readBytes()
            val string = bytes.decodeToString()
            return stringFormat.decodeFromString(string)
        }
        catch (e: SesrializationException) {
            throw CorruptionException("failed to read stored data", e)
        }
    }
}
```

保存するファイル内容を暗号化などしたいなら、ここで色々すれば良いと思う。ちょうどバイト列にしてるし。

### プログラムでの使用例

```kt:ApplicationとかActivityとか.kt
val dataStore = context.createDataStore(
    fileName = "hoge.json",
    serializer = HogeSerializer()
)
```

ファイル名は`Preferences DataStore`のときと違い拡張子は勝手に追加されないので書く必要があるが、別に拡張子なくても良いといえば良い。

```kt:設定画面とかのViewModel.kt
class HogeViewModel(
    private val dataStore: DataStore<Hoge>
) : ViewModel() {
    val fooLiveData = MutableLiveData<Int>()
    val barLiveData = MutableLiveData<String>()
    val bazLiveData = MutableLiveData<LocalTime>()

    /** `DataStore`から編集用の`LiveData`にデータを読み込み */
    init {
        dataStore.data
            .onEach {
                fooLiveData.postValue(it.foo)
                barLiveData.postValue(it.bar)
                bazLiveData.postValue(it.baz)
            }
            .flowOn(Dispatchers.Default)
            .launchIn(viewModelScope)
    }

    /** 編集内容の保存 */
    fun saveHoge() = viewModelScope.launch {
        dataStore.updateData { hoge ->
            hoge.copy(
                foo = fooLiveData.value!!,
                bar = barLiveData.value!!,
                baz = bazLiveData.value!!
            )
        }
    }
}
```

コンストラクタ引数付きの`ViewModel`については[この記事](/posts/2020/09_14_01_provide_view_model/)とか。  
今回別に関係ないのでDIでもなんでもとにかく良い感じに渡してやればいいと思う。

`DataStore`は`Application`継承したクラスとかに持たせればいいと思う。

`MutableLiveData`を`observe`してアレすれば値編集のたびに結果を保存することもできなくはないが、頻繁に値が更新されるようになっている場合その都度シリアライズ→ディスク書き込み→`Flow`への反映が発生して非常にアレな感じなので、保存ボタン押したときとか`Activity`終了前とか一定時間無操作とかで適当に`saveHoge()`呼ぶような作りにしておけばいいと思う。

ここまでとくに触れてなかったが、`DataStore`のデータの読み取りには[Kotlin Flow](https://kotlinlang.org/docs/reference/coroutines/flow.html)が使用されている。  
これはコルーチンの仕組みを使ってRxやろうといったもので、正直なところアプリ設定の読み取りとか簡単な編集画面くらいの用途ではそれほど要り様でもないような気もする。

上の例で使用している`onEach {...}`では値が更新されるたびにその内容が流れてきて`LiveData`に反映するようになっている(この実装では基本的には初期化時と保存時くらいしか流れてこないと思われるが)。`flowOn(CoroutineContext)`は上流の処理を指定した`CoroutineContext`で行う指示、`launchIn(CoroutineScope)`は指定した`CoroutineScope`でそれらを実行せよという指示だ。

編集画面での設定値の読み書きの場合は`Flow`を使ってこんな感じでいいが、実際にその設定値を使って処理をアレコレしたい場合には「`Flow`でデータが流れてきたら～」とか非同期にではなく、同期的に値を取得したくなると思うので、そういう時はコルーチン内で次のようにすれば良い。

```kt
coroutineScope.launch {
    try {
        val hoge: Hoge = dataStore.data.first()
        ...
    }
    catch (e: IOException) {
        ...
    }
}
```

公式ドキュメントによると「`IOExceptions`が発生する恐れがあるのでハンドルせよ」とのことだ。`try~catch`なり`runCatching`なり。
