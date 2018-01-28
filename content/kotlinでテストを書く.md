# kotlin でテストを書く

## Android の2つのテスト

- jvm で実行されるテストと Android のデバイスやエミュレーターで実行されるテストがある
- それぞれ、メリット・デメリットがあり場合に応じて使い分ける

## jvm で実行するテスト
  - ソースコードの置き場: `src/test`
  - 開発をしている端末（パソコン）の jvm で実行される
  - 起動や動作がAndroidの端末と比べて速い
  - AndroidのAPIを利用することはデフォルトではできない。外部のライブラリを利用すればできる
  - 端末依存、機種依存のバグの発見は難しい
  - 画面遷移のテストができない
## Androidのデバイスで実行するテスト
  - ソースコードの置き場: `src/androidTest`
  - 開発をしている端末に接続をしているAndroidのデバイスやエミュレーターで実行される
  - Android の API や、UI操作をコードで行うことができる
  - 起動や実行にかかる時間がjvmと比べて遅い

## 2つのテストの使い分け

- jvm のテスト: 単体機能のテストに利用する。外部のライブラリを使ってAndroidのAPIをある程度利用できるようにしておくことが前提。
- Android デバイスのテスト: UIの操作を含むシナリオテストに利用をする。

## プロジェクトの設定

![](./content/ss1.png)

`Configure Kotlin in Project` を選択してプロジェクトで kotlin が利用できるようにする。

## app/build.gradle

```groovy
android {
  defaultConfig {
    testInstrumentationRunner(
            "android.support.test.runner.AndroidJUnitRunner")
  }
  testOptions {
    unitTests.returnDefaultValues = true
  }
}
dependencies {
    // 省略
    testImplements 'junit:junit:4.12'
}
```

## app/build.gradle

- `testInstrumentationRunner`: Androidでテストを実行するときのテストランナー
- `unitTests.returnDefaultValues`: jvm でテストするときに Android のライブラリに default value を返すようにする
- jUnit: 単体テストを実行するためのフレームワーク、ライブラリ。kotlin でテストを書くときにもこれを使う

`returnDefaultValues` 以外はプロジェクトを作ったときに勝手に追加されているはず。

## テストの書き方

`app/src/test` に追加する

```kotlin
import org.hamcrest.CoreMatchers.`is`
import org.junit.Assert.assertThat
import org.junit.Test

class CalculatorTest {

    @Test // 1
    fun 足し算をするテスト() { // 2
        val a = 1
        val b = 2
        val sum = a + b
        assertThat(sum, `is`(3)) // 3
    }

}
```

## テストの書き方

1. テストで実行をするメソッドには `@Test` アノテーションを付ける
1. テストメソッドの名前は再利用性よりも分かりやすさを優先する。ここでは、日本語を使うことで何をテストするのかがひと目で分かるようにしている
1. 値のチェックは `assertThat` メソッドを利用。第1引数に実際の値、第2引数には「どういう状態であって欲しいのか」を意味する `Matcher` をパラメータに与える
  - Matcher には様々な種類がある。 `is` は第1引数の値と等しいことを確認する
  - ちなみに `is` は kotlin の予約語なので ``(バッククォート)` で囲む必要がある

## テストを実行する

![](content/ss2.png)

テストクラスやメソッドの横にある `▲` をクリックする

## テストを書けるようにする

```kotlin
class UntestableClass {

  fun findUserByID(id: Long): User? {
    val users = getUserFromInternet() // 1
    return users.find { it.id == id }
  }

}
```

## テストを書けるようにする

1. 実装が外部の環境（ここではインターネット）に依存しているため、テストを書くことができない
  - テストを実行するときに、ネットワークの状態や接続先の状態などにテスト結果が左右される
  - テストに失敗したときに純粋にコードのミスなのか、環境の問題なのかの切り分けができない

## テストを書けるようにする

```kotlin
interface UserAPI {

  fun getUserFromInternet(): List<User>

}
class TestableClass(val api: UserAPI) { // 1

  fun findUserByID(id: Long): User? {
    val users = api.getUserFromInternet()
    return users.find { it.id == id }
  }

}
```

## テストを書けるようにする

1. 外部の環境への接続方法をコンストラクタで指定できるようになったため、テストが書ける
  - テストのときはネットワークに接続詞ないスタブを指定することができる
  - テストに失敗してもコードにミスが有ることが特定できる
  - このあたりの事情が Dagger2 を利用した DI とかにつながっていく


## 宣伝

[Androidアプリ開発のためのKotlin実践プログラミング](https://www.amazon.co.jp/Android%E3%82%A2%E3%83%97%E3%83%AA%E9%96%8B%E7%99%BA%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AEKotlin%E5%AE%9F%E8%B7%B5%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0-%E8%88%B9%E6%9B%B3%E5%B4%87%E4%B9%9F/dp/479805366X/ref=sr_1_2?ie=UTF8&qid=1517109083&sr=8-2&keywords=kotlin)

![](content/book.jpg)

本を書きました。今回の資料に含まれるような内容のことも書いてあります。