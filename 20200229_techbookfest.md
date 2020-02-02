# Webアプリケーション構築から学ぶVの言語使用

## Vとは

Vとは昨年2019年6月にソースと公開と共にリリースされた新たな言語である。Vは以下のような特徴を持った言語である。

- 静的型付け言語
- クロスコンパイル
- 標準での組み込みが豊富
  - Read-Eval-Print Loop(REPL)を搭載
  - パッケージマネージャを搭載
  - ORMを搭載
  - フォーマッターを搭載
- CからVへ、VからCもしくはJavaScriptへのトランスパイスが可能
- システムのプログラミングだけでなく、Web、Game、GUI、モバイル、サイエンス、組み込みなどの殆どの分野で使用できる

と最初の2つだけみればGoogleが開発したGoと似たような特徴であるが、後半になればなるほどVの特徴が出てくる。ちなみにこのVはGoやRustにも強く影響されており構文は非常に似たものになっている。以下にVでのHello worldのプログラムをGoとRustで記載する。

```vlang
fn main() {
    areas := ['game', 'web', 'tools', 'science', 'systems', 'GUI', 'mobile']
    for area in areas {
        println('Hello, $area developers!')
    }
}
```

```golang
package main

import (
	"fmt"
)

func main() {
    areas := []string{"game", "web", "tools", "science", "systems", "GUI", "mobile"}
    for _, s := range areas {
        fmt.Printf("Hello, %s developers!\n", s)
    }
}
```

```rust
fn main() {
    let areas = ["game", "web", "tools", "science", "systems", "GUI", "mobile"];
    for area in areas.iter() {
        println!("Hello, {} developers!", area);
    }
}
```

まだリリースされて半年ほどしか経っていないVについて書いていこうと思います。

## Webアプリケーション構築
V自体もalpha版（執筆時：2020年2月2日）であるが、vwebというalpha版のモジュールが存在する。名前から見てわかるとおりWebアプリケーションを構築する上で必要となるモジュールである。かなりざっくりとした説明になるが、このモジュールはhttpのリクエストやクライアントへのレスポンスボディはもちろんのことhttpステータスラインやレスポンスヘッダを返す処理を行う。

このvwebモジュールを使ったVでのWebアプケーションの構築を紹介する。

まずは以下の `app.v` を作成する。

```vlang
module main

import (
	vweb
)

pub struct App {
mut:
	vweb vweb.Context
}

fn main() {
	vweb.run<App>(8080)
}

fn (app mut App) index() {
	app.vweb.text('Hello, world')
}

pub fn (app &App) init() {}
pub fn (app &App) reset() {}
```

見るとなんとなくわかるとは思いますが、説明していきます。
まず `import vweb` でvwebモジュールを読み込みます。これがないと何も始まりません。次に `App` という名で構造体を定義しています。フィールドには `vweb` という名の `vweb.Context` というhttpリクエストやレスポンス処理を返す複数のメソッドが定義されています。ちなみにVはクラスの定義はありません。ではどのようにメソッド定義するのかというと以下のようになります。

```vlang
import time

struct User {
    birthday string
}

fn (u User) age() int {
    now := time.now().format()
    now_date := now.split(' ')

    return (now_date[0].replace('-', '').int() - u.birthday.int()) / 10000
}

user := User{birthday: '20010623'}
println(user.age())
```

定義する関数にレシーバー引数を持つことによりメソッドとなります。今回は生年月日のフィールドを持っている `User` の構造体をレシーバーに指定した `age` つまり年齢を出力するメソッドを定義しています。
話を戻して `vweb.Context` も同じように構造体をレシーバー引数を持った関数が定義されており、メソッドとして使用することができます。

次に `main` 関数を見ていきます。
Vでは `main` 関数は1ファイルプログラムで1つのみ宣言でき、C言語などと同様にエントリポイントとしてみなされます。ですのでこの `app.v` を実行するとこの `main` 関数が呼ばれることとなります。この `main` で実行されているが8080ポートでリッスンするWebアプリケーションの立ち上げを行っています。ここでジェネリックを使用したコードを記載しています。Vは静的型付け言語ですので、定義する場合は型が必要となります。しかし型を定義してコードを書く際に必ずと行って出てくる問題にコードの再利用性が出てきます。例えば「引数に指定された連想配列のキーをキャメルケースにして返す関数」を定義するとしましょう。この場合引数に指定するのはコードを書く人によって千差万別です。そうなるとこの関数が返す型も千差万別になってしまいます。そこでこのジェネリックを使用することで、返り値をコードを書く人が記載することでコードの再利用性をあげるというものです。以下に例を記載します。

※この例はJavaScriptによってるのでVぽいやつに変える

```
```
