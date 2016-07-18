# パッケージ: Word Count

めっちゃシンプルなパッケージ書いて、開発をスムーズにするためのいくつかのツールを見ることから始めよか！
現在のバッファーにどれだけ文字が含まれているか数えて、小さいモーダルに表示するパッケージを書き始めるよ。


## Package Generator

パッケージを書き始める一番簡単な方法はAtomに最初から入っているPackage Generatorを使うことやで。
As you might expect by now, this generator is itself a separate package implemented in package-generator.

あなたはコマンドパレットを呼び出して、"Generate Package"を検索することでPackage Generatorを実行出来る。あなたの新しいプロジェクトの名前を尋ねるダイアログが出ます。「your-name-word-count」と命名してください。そうすればAtomはディレクトリーを生成し、スケルトンプロジェクトで埋め、「~/.atom/packages」ディレクトリーにリンクさせ、次回Atomを起動した際に読み込まれます。

Note: パッケージが読み込まれないといった状況に遭遇するかもしれません。atom.ioにホスティングされているパッケージと同じ名前(例えば、"wordcount"と"word-count")を新しいパッケージが使用していると、期待しているように読み込まれません。前述した「your-name-word-count」という名前を使うようにという指示に従ったのであれば問題ありません。

Atomがパッケージを構成する多くのファイルを生成したと思います。
パッケージがどのような構造になっているのか知るためにそれらを見てください。
, then we can modify them to get our word count functionality.


基本的なパッケージの構造は以下のとおりです。

```
my-package/
├─ grammars/
├─ keymaps/
├─ lib/
├─ menus/
├─ spec/
├─ snippets/
├─ styles/
├─ index.coffee
└─ package.json
```


すべてのパッケージが上記のディレクトリーを必要とするわけではありませんし、Package Generatorは「snippets」と「grammars」を生成しません。Let's see what some of these are so we can start messing with them.


### package.json

Node modulesと似て、Atomのパッケージはトップレベルディレクトリに「package.json」ファイルが存在しています。このファイルは、mainモジュールのパスやライブラリの依存関係、リソースの読み込み順のようなパッケージのメタデータを含んでいます。（and manifests specifying the order in which its resources should be loaded.の訳が怪しい）


Nodeの「package.json」のキーの一部が利用できるのに加えて、Atomの「package.json」は独自のキーを持っています。


+ main: パッケージのエントリポイントとなるCoffeeScriptファイルのパス。これが無い場合、Atomは「index.coffee」もしくは「index.js」を見に行きます。
+ styles: パッケージが読み込む必要があるスタイルシートの順番を定義する文字列配列です。これが定義されていない場合、「styles」ディレクトリーのスタイルシートがアルファベット順に読み込まれます。
+ keymap: パッケージが読み込む必要があるキーマッピングの順番を定義する文字列配列です。これが定義されていない場合、「keymaps」ディレクトリのマッピングがアルファベット順に読み込まれます。
+ menus: パッケージが読み込む必要があるメニューマッピングの順番を定義する文字列配列です。これが定義されていない場合、「menus」ディレクトリのマッピングがアルファベット順に読み込まれます。
+ snippets: パッケージが読み込む必要があるスニペットの順番を定義する文字列配列です。これが定義されていない場合、「snippets」ディレクトリ内のスニペットがアルファベット順に読み込まれます。
+ activationCommands: パッケージのactivationのトリガーとなるコマンドを定義するオブジェクトです。キーはCSSセレクター、valueはコマンドを定義する文字列配列です。パッケージはCSSセレクターで定義された範囲（associated scope?）で何かイベントがトリガーされるまで読み込まれません。
+ activationHooks: パッケージのactivationのトリガーとなるフックを定義する文字列配列です。このフックがトリガーされるまでパッケージは読み込まれません。現在、唯一のactivation hookは「language-package-name:grammar-used」（例えば「language-javascript:grammar-used」）のみです。



生成された「package.json」以下のようになっているはずです。

```
{
  "name": "wordcount",
  "main": "./lib/wordcount",
  "version": "0.0.0",
  "description": "A short description of your package",
  "activationCommands": {
    "atom-workspace": "wordcount:toggle"
  },
  "repository": "https://github.com/atom/your-name-word-count",
  "license": "MIT",
  "engines": {
    "atom": ">=1.0.0 <2.0.0"
  },
  "dependencies": {
  }
}
```

activationHooksを使いたいのであれば以下のようにする必要があります。

```
{
  "name": "wordcount",
  "main": "./lib/wordcount",
  "version": "0.0.0",
  "description": "A short description of your package",
  "activationHooks": ["language-javascript:grammar-used", "language-coffee-script:grammar-used"],
  "repository": "https://github.com/atom/your-name-word-count",
  "license": "MIT",
  "engines": {
    "atom": ">=1.0.0 <2.0.0"
  },
  "dependencies": {
  }
}
```

最初にすべきことのひとつとして、この情報が埋められていることを確認することです。name, description, プロジェクトのリポジトリのURL、そしてライセンスは出来るだけ早く埋めておきましょう。他の情報に関しては後々詳しく説明します。


### 警告

リポジトリのURLを更新することを忘れないようにしてください。The one generated for you is invalid by design and will prevent you from publishing your package until updated.
