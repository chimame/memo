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

実行すると `called A` と出力される。これは `dynamic_call` メソッド内で変数 `s` に文字列 `a` を代入しており、その `a` というメソッドを動的に `$s()` で実行するというプログラム例となっている。もちろん変数 `s` に文字列 `b` を代入するように変更すれば `called B` と表示されます。
このプログラムを見て違和感を覚えた方はいると思います。筆者自身もそもそも `dynamic_call` に文字列の `a` もしくは `b` を引数にもらい、それを動的にメソッドを呼び出せばいいのではないかとサンプルを用意するつもりでした。しかしV自体が動的メソッド呼び出し自体がまだまだ未完成らしくコンパイルで `C error. This should never happen.` として落ちてしまいます。

*******ここにコンパイルエラーの画像を添付*******

続いて、今までは `app.vweb.text` としてただのテキストを返していましたが、Webアプリケーションを構築する上でHTMLを返すためのテンプレートエンジンは必要な場合があります。このvwebモジュールにもテンプレートエンジンが実装されており、使用することができます。使用する例を以下に記載していきたいと思います。
まずは、以下の `index.html` を作成します。

```html:index.html
<p><img src="https://vlang.io/img/v-logo.png">is @message</p>
```

続いて作成した `index.html` をレンダリングするためのindexメソッドを以下のように書き換えます。

```diff
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
-	app.vweb.text('Hello, world')
+       message := 'awesome!!'
+	$vweb.html()
}

fn (app mut App) first() {
	app.vweb.text('Call first routing')
}

pub fn (app &App) init() {}
pub fn (app &App) reset() {}
```

こうすることで、 `http://localhost:8080` に再度アクセスすると作成した `index.html` でレンダリングされているのがわかると思います。

*******ここにindex.html描画の画像を添付*******

先程書いた動的メソッドの呼び出しや今回の `$vweb` なのですが、これは動的の呼び出しを記載しているわけですが、Vの特徴としてこれをコンパイル時に確定します。なので今回でいうと `index.html` のファイルが必要になりますが、もしそのファイルが存在しない場合はコンパイルエラーとなり事前にエラーを検出されます。さらには `index.html` もバイナリに含まれるためファイル検索するようなロジックが必要なくテンプレートの展開が容易に行えるといった特徴をもっています。

*******ここにtemplate not foundの画像を添付*******

Vの特徴を最初にあげた中で「ORMを搭載」と書きました。Webアプリケーションを作成する上でこのORMは非常に強力なものとなります。実際にORMを使用したWebアプリケーション構築例を記載していきます。
まずは必要なデータベースを準備します。以下にPostgreSQLのデータベースおよびテーブル作成のSQLを記載します。

```sql
create database library;

\c library

drop table books;

create table books (
	id serial primary key,
	title text default '',
	author text default ''
);

insert into books (title, author) values (
	'Kimetsu no Yaiba',
	'koyoharu gotoge'
);

insert into books (title, author) values (
	'One piece',
	'eichiro oda'
);
```


次にデータベースコネクション用の変数を定義します。同時にアプリケーション起動時データベースへ接続する処理も追加します。

```diff
module main

import (
    vweb
+   pg
)

pub struct App {
mut:
    vweb vweb.Context
+   db   pg.DB
}

fn main() {
    vweb.run<App>(8080)
}

fn (app mut App) index() {
    message := 'awesome!!'
    $vweb.html()
}

fn (app mut App) first() {
    app.vweb.text('Call first routing')
}

-pub fn (app &App) init() {}
+pub fn (app mut App) init() {
+  db := pg.connect(pg.Config{
+       host:   'db'
+       dbname: 'library'
+       user:   'postgres'
+   }) or { panic(err) }
+   app.db = db
+}
pub fn (app &App) reset() {}
```

ここで初めて `init` のメソッドを使用しました。 `init` のメソッドはどのタイミングで呼ばれるかというと `main` メソッドで `vweb.run` を実行していると思いますが、そのrun内ではhttpのリッスン処理後にこの `init` 処理が呼ばれる定義となっています。ですのでここでWebアプリケーションの初期化処理を記載するとよいです。ここまで追加してWebアプリケーションが立ち上がるか一度確認してください。正常にデータベースに接続できると立ち上がるはずです。
続いてORMに必要なStructの定義を追加するために以下のような `book.v` を作成してください。

```vlang
module main

struct Book {
    id     int
    title  string
    author string
}

pub fn (app &App) all_books() []Book {
    db := app.db
    books := db.select from Book
    return books
}
```

次にBookの一覧を `/` つまりindexで表示するように変更します。まずは以下のように `app.v` を変更します。

```diff
module main

import (
    vweb
    pg
)

pub struct App {
mut:
    vweb vweb.Context
    db   pg.DB
}

fn main() {
    vweb.run<App>(8080)
}

fn (app mut App) index() {
-   message := 'awesome!!'
+   books := app.all_books()
    $vweb.html()
}

fn (app mut App) first() {
    app.vweb.text('Call first routing')
}

pub fn (app mut App) init() {
  db := pg.connect(pg.Config{
       host:   'db'
       dbname: 'library'
       user:   'postgres'
   }) or { panic(err) }
   app.db = db
}
pub fn (app &App) reset() {}
```

最後に取得したBookのデータを表示するように `index.html` を変更します。

```diff
-<p><img src="https://vlang.io/img/v-logo.png">is @message</p>
+<ol>
+    @for book in books
+        <li>title: @book.title author: @book.author</li>
+    @end
+</ol>
```

ファイルの変更は以上です。最後に大事なのが起動コマンドです。今までは `v run app.v` と `app.v` をビルドのエントリーポイントとして指定してきました。しかし今回は `book.v` が必要となり、マルチエントリーポイントが必要となっています。なので起動は少し変更し `v run .` として対象のディレクトリ全体を指定して起動するようにします。起動後に `/` のindexにアクセスすると登録しているBookの一覧が表示されるはずです。

*******ここにBook一覧表示の画像を添付*******

今回はすべてのBookデータ一覧を表示するようにしましたが、実際には条件によってデータを絞り込む必要があるはずです。仮に絞り込み条件としてtitleに”Kimetsu no Yaiba”というものだけを一覧に表示したい場合は以下のようにデータ取得条件をSQLのようにすれば可能です。

```diff
-books := db.select from Book
+books := db.select from Book where title = 'Kimetsu no Yaiba'
```

残念なことにまだLIKE句は実装されていないのか使用することはできなかったが、大なりもしくは小なりという条件は使用可能です。

今度は表示ではなく、登録ができるようにしたいと思います。
登録用フォームを表示するためのテンプレートおよびルーティングを追加します。まずは以下のような登録フォームを記載したテンプレート `new_book.html` を準備します。

```html
<form action='/create_book' method='post'>
    <input type='text' placeholder='Title' name='title'> <br>
    <input type='text' placeholder='Author' name='author'></textarea>
    <input type='submit'>
</form>
```

次に先程作成した登録フォームテンプレートを表示するだけのためのルーティングを `app.v` に追加します。

```diff
module main

import (
    vweb
    pg
)

pub struct App {
mut:
    vweb vweb.Context
    db   pg.DB
}

fn main() {
    vweb.run<App>(8080)
}

fn (app mut App) index() {
    books := app.all_books()
    $vweb.html()
}

+fn (app mut App) new_book() {
+        $vweb.html()
+}

fn (app mut App) first() {
    app.vweb.text('Call first routing')
}

pub fn (app mut App) init() {
  db := pg.connect(pg.Config{
       host:   'db'
       dbname: 'library'
       user:   'postgres'
   }) or { panic(err) }
   app.db = db
}
pub fn (app &App) reset() {}
```

これを先程と一緒に `v run .` で起動し `/new_book` にアクセスすると登録フォームが表示されるはずです。

*******ここに登録フォーム表示の画像を添付*******

登録フォームを表示する処理まではできましたが、登録処理がまだできていません。ですので今から登録処理を作っていきたいと思います。
まずは `book.v` に登録するためのメソッドを追加します。

```diff
module main

struct Book {
    id     int
    title  string
    author string
}

pub fn (app &App) all_books() []Book {
    db := app.db
    books := db.select from Book
    return books
}

+pub fn (app &App) insert_book(title string, author string) bool {
+    if title == '' || author == ''  {
+        return false
+    }
+    book := Book{
+        title: title
+        author: author
+    }
+    db := app.db
+    //db.insert(book) 本当はこれで動くらしいが、動かないので仕方なく↓で直接INSERT文を実行
+    db.exec_param2('insert into books (title, author) values ($1,$2)', book.title, book.author)
+    return true
+}
```

引数に `title` と `author` をもらい入力チェック後にINSERTを実行すると単純なメソッドです。しかし残念なことにプログラム内のコメントで記載している通り、本来であればORMの機能を用いてINSERTできる処理がVの不具合で動作しないらしく今回は直接SQLを発行する形となっています。
次に登録処理を受け付けるルーティング処理を `app.v` に追加します。

```diff
module main

import (
    vweb
    pg
)

pub struct App {
mut:
    vweb vweb.Context
    db   pg.DB
}

fn main() {
    vweb.run<App>(8080)
}

fn (app mut App) index() {
    books := app.all_books()
    $vweb.html()
}

fn (app mut App) new_book() {
        $vweb.html()
}

+fn (app mut App) create_book() {
+    title := app.vweb.form['title']
+    author := app.vweb.form['author']
+    if app.insert_book(title, author) {
+        app.vweb.redirect('/')
+    } else {
+    app.vweb.text('Empty title or author')
+    }
+}

fn (app mut App) first() {
    app.vweb.text('Call first routing')
}

pub fn (app mut App) init() {
  db := pg.connect(pg.Config{
       host:   'db'
       dbname: 'library'
       user:   'postgres'
   }) or { panic(err) }
   app.db = db
}
pub fn (app &App) reset() {}
```

`create_book` メソッド追加して完了です。ここの入力値を取得する処理は `app.vweb.form['title']` のように取得していますが、本来であれば `fn (app mut App) create_book(title string)` と受け取れるようになるようです。

これで登録画面からtitleおよびauthorを入力することでBook一覧に入力したBookのデータが追加で表示されるようになるはずです。

# VでのWebアプリケーションを構築してみて
いかがだったでしょうか。これを書いている私自身はVという言語を初めて触ってみましたが書き心地としては悪くないなという印象です。ですが、まだまだ未完成の言語であるがゆえに色々としっかりと動作しないのが残念で仕方ありません。ただこのV自体はOSSであるため誰でも開発に参加することが可能です。発展途上の言語であるがゆえに直しどころが多くあり、コントリビューションチャンスがまだまだあると捉えることができます。ぜひ興味ある方は開発に参加して頂ければと思います。
