Webアプリケーション構築から学ぶVの言語仕様

# Vとは

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

# Webアプリケーション構築
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
Vでは `main` 関数は1ファイルプログラムで1つのみ宣言でき、C言語などと同様にエントリポイントとしてみなされます。ですのでこの `app.v` を実行するとこの `main` 関数が呼ばれることとなります。この `main` で実行されているが8080ポートでリッスンするWebアプリケーションの立ち上げを行っています。この `vweb.run` の処理でジェネリックを使用したコードが記載されています。
Vは静的型付け言語ですので、定義する場合は型が必要となります。しかし型を定義してコードを書く際に必ずと行って出てくる問題にコードの再利用性が出てきます。例えば「引数に指定された連想配列のキーをキャメルケースにして返す関数」を定義するとしましょう。この場合引数に指定するのはコードを書く人によって千差万別です。そうなるとこの関数が返す型も千差万別になってしまいます。そこでこのジェネリックを使用することで、返り値をコードを書く人が記載することでコードの再利用性をあげるというものです。以下に例を記載します。

※この例はJavaScriptによってるのでVぽいやつに変える

```
```

では `init` と `reset` の説明はまだですが、一度ここでこのコードを実行したいと思います。実行は `v run app.v` とするとWebアプリケーションが立ち上がります。

*******ここにターミナルの実行結果の画像を添付*******

正常に起動できた場合は

*******ここにHello Worldの画像を添付*******

こう表示されることでなんとなく理解できた方もいると思いますが、 `/` でアクセスした場合は `index` の関数が実行されております。 `index` の関数では `Hello, world` というテキストを返すという処理が記載されています。
ではそれ以外のURLにアクセスした場合はどうなるか試してみたいと思います。例えば `/test` というパスにアクセスしてみます。

*******ここに404の画像を添付*******

 上記のように `404 Not Found` のメッセージと共に404のhttpステータスが返ったきたと思います。vwebモジュールはどのようにルーティングを解釈しているのかというとさすがalpha版というべきかURLの第1ディレクトリ名のメソッドが呼ばれるようです。なので具体例を上げる
 
 ```
 http://localhost:8080/first/second
 ```
 
 とアクセスすると `first` と定義されたメソッドが呼ばれるような仕組みになっております。このルーティング処理に関する部分もこれから作られていくと思われます。なので作成した `app.v` のままでは上記のURLを受けることができないので `first` メソッドを作成します。
 
```vlang:app.v
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

fn (app mut App) first() {
	app.vweb.text('Call first routing')
}

pub fn (app &App) init() {}
pub fn (app &App) reset() {}
```
 
`first` メソッドを新たに追加することで上記URLのアクセスを受けることができます。
 
*******ここにCall first routingの画像を添付*******

ここで少し面白いのがVの動的なメソッド呼び出し方法です。上記のようにWebアプリケーションではアクセスされるURLに対して適切なメソッドを呼び出す必要があります。その場合に必要になるのがこの動的なメソッドの呼び出しなのですが、Vでは以下のように動的に呼び出すことができます。

```vlang
pub struct App {
}

fn (app mut App) a() {
    println('called A')
}

fn (app mut App) b() {
    println('called B')
}

fn (app mut App) dynamic_call() {
    s := 'a'
    app.$s() or { println('no method') }
}

mut app1 := App{}
app1.dynamic_call()
```

実行すると `called A` と出力される。これは `dynamic_call` メソッド内で変数 `s` に文字列 `a` を代入しており、その `a` というメソッドを動的に `$s()` で実行するというプログラム例となっている。もちろん変数 `s` に文字列 `b` を代入するように変更すれば `called B` と表示される。
このプログラムを見て違和感を覚えた方はいると思う。筆者自身もそもそも `dynamic_call` に文字列の `a` もしくは `b` を引数にもらい、それを動的にメソッドを呼び出せばいいのではないかとサンプルを用意するつもりだった。しかしV自体が動的メソッド呼び出し自体がまだまだ未完成らしくコンパイルで `C error. This should never happen.` として落ちてしまう。

*******ここにコンパイルエラーの画像を添付*******
