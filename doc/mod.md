# Go Modules Reference
## はじめに
モジュールはGoがパッケージを管理する仕組みです。

このドキュメントはGoのモジュールに関する詳細なリファレンスマニュアルです。Goのプロジェクトを作り始める際は、[How to Write Go Code](https://golang.org/doc/code)をご覧ください。モジュールを使用したり、モジュールにプロジェクトをマイグレートしたり、その他トッピクスについては、[Using Go Modules](https://blog.golang.org/using-go-modules)から始まる一連のブログ記事をご覧ください。

## モジュール、パッケージ、バージョン

**モジュール**は一緒にリリース、バージョンコントロール、配布されるパッケージの集合です。モジュールはバージョンコントロールリポジトリから直接ダウンロードしたり、モジュールのプロキシサーバーからダウンロードできます。

モジュールは、[`go.mod`ファイル](https://golang.org/ref/mod#go-mod-file)で宣言された[モジュールパス](https://golang.org/ref/mod#glos-module-path)と、そのモジュールの依存関係に関する情報によって識別されます。**モジュールルートディレクトリ**は`go.mod`ファイルを含むディレクトリです。**メインモジュール**は`go`コマンドが呼ばれるディレクトリを含むディレクトリです。

モジュール内のそれぞれの**パッケージ**は、同じディレクトリ内にあり一緒にコンパイルされるソースファイルの集合です。**パッケージパス**は、モジュールパスとパッケージを含むサブディレクトリ（モジュールルートからの相対パスになります）です。例えば、モジュール`"golang.org/x/net"`はディレクトリ`"html"`中のパッケージを含みます。そのパッケージパスは`"golang.org/x/net/html"`になります。

### モジュールパス

**モジュールパス**はモジュールの正式名称で、モジュールの[go.modファイル](https://golang.org/ref/mod#glos-go-mod-file)内で[`module`ディレクティブ](https://golang.org/ref/mod#go-mod-file-module)を使って宣言されます。モジュールのパスは、そのモジュール内に含まれるパッケージのプレフィックスです。

モジュールパスは、そのモジュールが何をするのか、そしてどこにあるのかを説明しなければなりません。通常、モジュールパスは、リポジトリルートパス、リポジトリ内のディレクトリ（通常は空）、メジャーバージョンサフィックス（メジャーバージョン2以上の場合のみ）で構成されています。

- **リポジトリルートパス**は、モジュールパスのうち、そのモジュールが開発されているバージョンコントロールリポジトリのルートディレクトリに対応しています。ほとんどのモジュールは、そのリポジトリのルートディレクトリで定義されているため、通常そのパスは全体のパスとなります。例えば、`golang.org/x/net`は同名のモジュールに対するリポジトリルートパスです。`go`コマンドがモジュールパスを元にしてHTTPリクエストでリポジトリを検索する方法については、[Finding a repository for a module path](https://golang.org/ref/mod#vcs-find)をご覧ください。
- モジュールがリポジトリのルートディレクトリ定義されていない場合、**モジュールサブディレクトリ**はモジュールパスの中のディレクトリ名の部分にあたり、メジャーバージョンサフィックスは含みません。これはセマンティックバージョンタグのプレフィックスとしても機能します。例えば、モジュール`golang.org/x/tools/gopls`は`golang.org/x/tools`をルートとするリポジトリのサブディレクトリ`gopls`にあるので、モジュールサブディレクトリ`gopls`を持っています。[Mapping versions to commits](https://golang.org/ref/mod#vcs-version)や[Module directories within a repository](https://golang.org/ref/mod#vcs-dir)をご覧ください。
- モジュールがメジャーバージョン2以上でリリースされている場合、モジュールパスは`/v2`のようなメジャーバージョンサフィックスで終わらなければなりません。これはサブディレクトリ名の一部であってもなくてもどちらでも構いません。例えば、`golang.org/x/repo/sub/v2`というパスのモジュールは、リポジトリ`golang.org/x/repo`のサブディレクトリ`/sub`または`/sub/v2`にあります。

あるモジュールが他のモジュールに依存している可能性がある場合、`go`コマンドがそのモジュールを見つけてダウンロードできるように、以上のルールに従う必要があります。また、モジュールパスで使用できる文字には、いくつかの[制限](https://golang.org/ref/mod#go-mod-file-ident)があります。

### バージョン

**バージョン**は、モジュールの不変的なスナップショットを識別し、[release](https://golang.org/ref/mod#glos-release-version)または[pre-release](https://golang.org/ref/mod#glos-pre-release-version)のいずれかになります。各バージョンは`v`で始まり、その後にセマンティックバージョンが続きます。バージョンの形式や解釈、比較についての詳細は[Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)をご覧ください。

要約すると、セマンティックバージョンは、ドットで区切られた3つの非負整数（左からメジャーバージョン、マイナーバージョン、パッチバージョン）で構成されています。パッチバージョンの後には、オプションでハイフンで始まるプレリリース文字列をつけることができます。プレリリース文字列やパッチバージョンの後には、プラスで始まるビルドメタデータ文字列をつけることができます。例えば、`v0.0.0`、`v1.12.134`、`v8.0.5-pre`、`v2.0.9+meta`は全て有効なバージョンです。

バージョンの各パートは、そのバージョンが安定しているかどうかや、以前のバージョンと互換性があるかどうかを示しています。

- パッケージが削除されるなど、モジュールのパブリックインターフェースやドキュメント化された機能に後方互換性のない変更が加えられた後は、[major version](https://golang.org/ref/mod#glos-major-version)をインクリメントし、マイナーバージョンとパッチバージョンをゼロに設定しなければなりません。
- 新しい機能が追加されるなど、後方互換性のある変更を行った後は、[minor version](https://golang.org/ref/mod#glos-minor-version)をインクリメントし、パッチバージョンをゼロに設定しなければなりません。
- バグフィックスや最適化など、モジュールのパブリックインターフェースに影響を与えない変更を行った後は、[patch version](https://golang.org/ref/mod#glos-patch-version)をインクリメントしなければなりません。
- プレリリースのサフィックスは、そのバージョンが[pre-release](https://golang.org/ref/mod#glos-pre-release-version)であることを示します。プレリリースバージョンは対応するリリースバージョンの前にソートされます。例えば、`v1.2.3-pre`は`v1.2.3`より前になります。
- バージョンを比較する際は、ビルドメタデータのサフィックスは無視されます。ビルドメタデータつきのタグは、バージョンコントロールリポジトリでは無視されますが、`go.mod`ファイルで明示されたバージョンにはビルドメタデータが保存されます。サフィックス`+incompatible`は、モジュールのバージョンがメジャーバージョン2以上に移行するする前にリリースされたバージョンであることを示します（[Compatibility with non-module repositories](https://golang.org/ref/mod#non-module-compat)をご覧ください）。

メジャーバージョンが0であったり、プレリリースのサフィックスがついていたりすると、不安定なバージョンと思われてしまいます。不安定なバージョンは互換性を満たさないかもしれません。例えば、`v0.2.0`は`v0.1.0`と互換性がないかもしれませんし、さらに言うと`v1.5.0-beta`は`v1.5.0`と互換性がないかもしれません。

Goはバージョンコントロールシステムのモジュールにアクセスする際に、このルールに従わないタグ、ブランチ、リビジョンを使用することがあります。しかし、メインモジュール内では、`go`コマンドがこのルールに従わないリビジョン名を自動的に正規のバージョンに変換します。`go`コマンドは、このプロセス中にビルドメタデータのサフィックス（`+incompatible`を除く）も削除します。この結果として、バージョンコントロールシステムからリビジョン識別子（Gitコミットハッシュなど）とタイムスタンプをエンコードして、リリース前のバージョンである[pseudo-version](https://golang.org/ref/mod#glos-pseudo-version)を生成することがあります。例えば、コマンド`go get -d golang.org/x/net@daa7c041`はコミットハッシュ`daa7c041`を疑似バージョン`v0.0.0-20191109021931-daa7c04131f5`に変換します。正規のバージョンはメインモジュールの外側で必要とされ、`go.mod`ファイルに`master`のような非正規なバージョンがある場合、エラーを出力します。

### 疑似バージョン

**疑似バージョン**は、バージョンコントロールリポジトリの特定のリビジョンに関する情報をエンコードした、特別な形式の[pre-release version](https://golang.org/ref/mod#glos-pre-release-version)です。例えば、`v0.0.0-20191109021931-daa7c04131f5`は疑似バージョンです。

疑似バージョンは、[semantic version tags](https://golang.org/ref/mod#glos-semantic-version-tag)がないリビジョンを参照します。例えば、開発ブランチなどバージョンタグを作成する前にコミットをテストするために使用されます。

各疑似バージョンは3つのパートから成り立っています。

- ベースバージョンプレフィックス（`vX.0.0`または`vX.Y.Z-0`）。リビジョンの前にあるセマンティックバージョンタグから派生したものか、そのようなタグがない場合は`vX.0.0`です。
- タイムスタンプ（`yyyymmddhhmmss`）。リビジョンが作成されたUTC時間です。Gitの場合は、AuthorDateではなくCommitDateが使われます。
- リビジョン識別子（`abcdefabcdef`）。コミットハッシュのプレフィックス12文字で、Subversionではゼロパディングされたリビジョン番号です。

各疑似バージョンは、ベースバージョンに応じて3つの形式のうちの1つが採用されます。これらの形式は、疑似バージョンがそのベースバージョン以上で、かつ次のタグつきバージョン以下であることを保証します。

- `vX.0.0-yyyymmddhhmmss-abcdefabcdef`は、既知のベースバージョンがない場合に使用されます。ほかのバージョンと同様に、メジャーバージョン`X`はモジュールの[major version suffix](https://golang.org/ref/mod#glos-major-version-suffix)と一致しなければなりません。
- `vX.Y.Z-pre.0.yyyymmddhhmmss-abcdefabcdef`は、ベースバージョンが`vX.Y.Z-pre`のようにプレリリースバージョンの場合に使用されます。
- `vX.Y.(Z+1)-0.yyyymmddhhmmss-abcdefabcdef`は、ベースバージョンが`vX.Y.Z`のようにリリースバージョンの場合に使用されます。例えば、ベースバージョンが`v1.2.3`の場合、疑似バージョンは`v1.2.4-0.20191109021931-daa7c04131f5`のようになります。

複数の疑似バージョンが異なるベースバージョンを使用して同じコミットを参照することがあります。これは疑似バージョンが書かれた後にそれより前のバージョンがタグづけされた場合に自然に起こることです。

これらの形式は、疑似バージョンに2つのメリットをもたらします。

- ベースバージョンが分かっている疑似バージョンは、そのベースバージョン以上で、かつ後方のプレリリースバージョン以下になります。
- ベースバージョンのプレフィックスが同じである疑似バージョンは、時系列順にソートされます。

`go`コマンドは、モジュール製作者が疑似バージョンと他のバージョンとの比較を制御できることと、疑似バージョンがモジュールのコミット履歴に実際に含まれているリビジョンを参照していることを保証するために、いくつかのチェックを行います。

- ベースバージョンが明示されている場合、対応するセマンティックバージョンタグが存在しなければなりません。そのタグは疑似バージョンに記述されたリビジョンの祖先です。これにより、開発者が`v1.999.999-99999999999999-daa7c04131f5`のようにタグづけされたすべてのバージョン以上の疑似バージョンを使って[minimal version selection](https://golang.org/ref/mod#glos-minimal-version-selection)を回避することを防ぎます。
- タイムスタンプはリビジョンのタイムスタンプと一致しなければなりません。これにより、攻撃者が[module proxies](https://golang.org/ref/mod#glos-module-proxy)に無制限に同一の疑似バージョンを作ることを防ぎます。また、モジュールの利用者がバージョンの相対的な順序を変更することも防ぎます。
- リビジョンはモジュールリポジトリのブランチやタグのいづれかの祖先でなければなりません。これにより、攻撃者が未承認の変更やプルリクエストを参照することを防ぎます。

疑似バージョンは手動で入力する必要はありません。多くのコマンドは、コミットハッシュやブランチ名を受け取ると、それを自動的に疑似バージョン（可能であればタグつきバージョン）に変換します。例えば以下のようになります。

```sh
go get -d example.com/mod@master
go list -m -json example.com/mod@abcd1234
```

### メジャーバージョンサフィックス

メジャーバージョン2以降、モジュールパスは`/v2`のようなメジャーバージョンに対応したサフィックスが必要です。例えば、`v1.0.0`で`example.com/mod`というパスを持つモジュールは、`v2.0.0`では`example.com/mod/v2`というパスを持つ必要があります。

メジャーバージョンサフィックスは、[import compatibility rule](https://research.swtch.com/vgo-import)を満たしています。

> 古いパッケージと新しいパッケージのインポートパスが同じ場合、新しいパッケージは古いパッケージとの後方互換性がなければなりません。

定義上、新しいメジャーバージョンのモジュールが持つパッケージは、それより前のメジャーバージョンの対応するパッケージとの後方互換性はありません。そのため、`v2`以降のパッケージには新しいインポートパスが必要になります。これは、モジュールパスにメジャーバージョンサフィックスを追加することで実現しています。モジュールパスは、モジュールが持つ各パッケージのインポートパスのプレフィックスにあたるため、モジュールパスにメジャーバージョンサフィックスを追加することで、互換性のないバージョンごとに異なるインポートパスを提供できます。

メジャーバージョンサフィックスは、メジャーバージョン`v0`や`v1`では使用できません。`v0`は不安定で互換性が保証されてないため、`v0`と`v1`の間でモジュールパスを変更する必要はありません。さらに、ほとんどのモジュールにおいて、`v1`は`v0`の最後のバージョンと後方互換性があります。`v1`は`v0`に対して互換性のない変更があることを示すのではなく、互換性を約束するという意味を持ちます。

特別なケースとして、`gopkg.in/`で始まるモジュールパスは、`v0`や`v1`でも、常にメジャーバージョンサフィックスを持つ決まりがあります。ただし、サフィックスはスラッシュではなくドットで始まります（例えば、`gopkg.in/yaml.v2`）。

メジャーバージョンサフィックスを使うと、同じビルドに複数のメジャーバージョンのモジュールを共存させることができます。これは、[diamond dependency problem](https://research.swtch.com/vgo-import#dependency_story)を回避するために必要になります。通常、推移的依存関係によって2つの異なるバージョンでモジュールが必要とされる場合、より新しいバージョンが使用されます。しかし、2つのバージョンに互換性がない場合、どちらを選んでもすべての利用者を満足させることができません。互換性のないバージョンは異なるメジャーバージョンを持たなければならないので、メジャーバージョンサフィックスを使って異なるモジュールパスを持たなければなりません。これによりコンフリクトが解消できます。つまり、別々のサフィックスを持つモジュールは別々のモジュールとして扱われ、それらのパッケージはたとえモジュールルートから見て同じサブディレクトリにあるパッケージだとしても、別々のものとして扱われるのです。

多くのGoプロジェクトは、モジュールに移行する前（あるいはモジュールが導入される前）に、メジャーバージョンサフィックスを使わずに`v2`以上のバージョンをリリースしています。これらのバージョンには、ビルドタグ`+incompatible`がつけられています（例えば、`v2.0.0+incompatible`）。詳しくは[Compatibility with non-module repositories](https://golang.org/ref/mod#non-module-compat)をご覧ください。

### パッケージからモジュールに解決する

`go`コマンドが[package path](https://golang.org/ref/mod#glos-package-path)を使ってパッケージをダウンロードする際、どのモジュールがそのパッケージを提供しているか判断する必要があります。

`go`コマンドはパッケージパスのプレフィックスから順に[build list](https://golang.org/ref/mod#glos-build-list)を検索していきます。例えば、`example.com/a/b`というパッケージがインポートされていて、`example.com/a`というモジュールがビルドリストにある場合、`go`コマンドは`example.com/a`のディレクトリ`b`にパッケージがあるかどうか確認します。パッケージとみなされるためには、拡張子`.go`を持つファイルが1つ以上ディレクトリになければなりません。この目的のために、[Build constraints](https://golang.org/pkg/go/build/#hdr-Build_Constraints)は適用されません。ビルドリストの中にパッケージを提供するモジュールが1つだけであれば、そのモジュールが使用されます。2つ以上のモジュールがパッケージを提供している場合は、エラーが出力されます。`go`コマンドに`-mod=mod`フラグをつけることで、不足しているパッケージを提供する新しいモジュールを見つけ、`go.mod`と`go.sum`を更新するよう指示することができます。[`go get`](https://golang.org/ref/mod#go-get)[`go mod tidy`](https://golang.org/ref/mod#go-mod-tidy)コマンドは自動的にこれを行います。

`go`コマンドはパッケージパスに対応する新しいモジュールを検索するときに環境変数`GOPROXY`を確認します。この環境変数には、カンマで区切られたプロキシURLのリストか、キーワード`direct`または`off`が指定されます。プロキシURLは、`go`コマンドが[`GOPROXY`protocol](https://golang.org/ref/mod#goproxy-protocol)を使用して[module proxy](https://golang.org/ref/mod#glos-module-proxy)に問い合わせることを示しています。`direct`は`go`コマンドが[communicate with a version control system](https://golang.org/ref/mod#vcs)ことを示しています。`off`は通信を行わないことを示しています。[environment variables](https://golang.org/ref/mod#environment-variables)`GOPRIVATE`と`GONOPROXY`を使って、この動作を制御することもできます。

`go`コマンドは`GOPROXY`リストの各エントリに対して、指定したパッケージを提供する可能性のあるモジュールパス（つまり、パッケージパスの各プレフィックス）の最新バージョンをリクエストします。見つかった各モジュールパスに対して、`go`コマンドは最新バージョンのモジュールをダウンロードし、そのモジュールに指定したパッケージが含まれているかどうかを確認します。指定したパッケージが1つ以上のモジュールに含まれている場合は、最も長いパスを持つモジュールが使用されます。1つ以上のモジュールが見つかっても、指定したパッケージが含まれていない場合は、エラーが出力されます。モジュールが見つからない場合、`go`コマンドは`GOPROXY`リストの次のエントリを試します。エントリーがない場合は、エラーが出力されます。

例えば、`go`コマンドが`golang.org/x/net/html`というパッケージを提供するモジュールを探していて、`GOPROXY`に`https://corp.example.com,https://proxy.golang.org`が設定されているとします。`go`コマンドは以下のようにリクエストを行うでしょう。

1. `https://corp.example.com/`に（並列に）リクエストを行う
    1. `golang.org/x/net/html`の最新バージョンをリクエストする
    1. `golang.org/x/net`の最新バージョンをリクエストする
    1. `golang.org/x`の最新バージョンをリクエストする
    1. `golang.org`の最新バージョンをリクエストする
1. `https://corp.example.com/`へのリクエストが全て404または410エラーの場合、`https://proxy.golang.org/`にリクエストを行う
    1. `golang.org/x/net/html`の最新バージョンをリクエストする
    1. `golang.org/x/net`の最新バージョンをリクエストする
    1. `golang.org/x`の最新バージョンをリクエストする
    1. `golang.org`の最新バージョンをリクエストする

適切なモジュールが見つかると、`go`コマンドは新しいモジュールのパスとバージョンを含む新しい[requirement](https://golang.org/ref/mod#go-mod-file-require)をメインモジュール`go.mod`ファイルに追加します。これにより、将来同じパッケージがロードされたときに、同じモジュールが同じバージョンで使用されるようになります。解決されたパッケージが、メインモジュールのパッケージによってインポートされていない場合、新しい`require`のモジュールには`// indirect`コメントがつきます。

## `go.mod`ファイル

### 字句要素

### モジュールパスとバージョン

### 文法

### `module` ディレクティブ

#### 非推奨

### `go` ディレクティブ

`go`ディレクティブは、モジュールがGoの特定バージョンのセマンティクスを想定して書かれていることを示します。バージョンは、有効なリリースバージョンでなければなりません。すなわち、正整数の後に、ドット、非負整数と続けたものになります（例えば、`1.9`、`1.14`）。

`go`ディレクティブは、元々はGo言語への後方互換性のない変更をサポートするために導入されました（[Go 2 transition](https://go.googlesource.com/proposal/+/master/design/28221-go2-transitions.md)を参照）。モジュールが導入されてからこれまで互換性のない言語の変更はありませんでしたが、`go`ディレクティブは新しい言語機能の使い方に影響を与えます。

- モジュール内のパッケージについては、`go`ディレクティブで指定されたバージョン以降に導入された言語機能の使用をコンパイラが拒否します。例えば、モジュールに`go 1.12`というディレクティブがある場合、そのパッケージではGo 1.13で導入された`1_000_000`のような数値リテラルを使用できません。
- 古いGoバージョンでモジュールのパッケージの1つをビルドしてコンパイルエラーが発生した場合、そのモジュールがより新しいGoバージョン用に書かれていることをエラーが知らせてくれます。例えば、あるモジュールが`go 1.13`で、パッケージが数値リテラル`1_000_000`を使っているとします。そのパッケージがGo 1.12でビルドされる場合、コンパイラはそのコードがGo 1.13用に書かれていることを示します。

さらに、`go`コマンドは`go`ディレクティブで指定されたバージョンを元に動作を変えます。これには次のような影響があります。

- `go 1.14`以降では、自動[vendoring](https://golang.org/ref/mod#vendoring)機能を有効にすることができます。`vendor/modules.txt`というファイルが存在し、`go.mod`と一貫していれば、`-mod=vendor`フラグを明示的につける必要はありません。
- `go 1.16`以降では、`all`パッケージパターンは、[main module](https://golang.org/ref/mod#glos-main-module)内のパッケージやテストによって他動的にインポートされたパッケージにのみ一致します。これは、モジュールが導入されてからは、`go mod vendor`が保持するパッケージの一覧と同じです。`go 1.15`以前では、`all`はメインモジュール内のパッケージによってインポートされたパッケージのテストやそれらのパッケージのテストなども含みます。

`go.mod`ファイルには高々1つの`go`ディレクティブを含めることができます。`go`ディレクティブがない場合、ほとんどのコマンドは現在のGoのバージョンを追記します。

```
GoDirective = "go" GoVersion newline .
GoVersion = string | ident .  /* valid release version; see above */
```

例:

```
go 1.14
```

### `require` ディレクティブ

### `exclude` ディレクティブ

### `replace` ディレクティブ

### `retract` ディレクティブ

### 自動アップデート

## Minimal version selection (MVS)

## 用語集

### ビルド制約（build constraint）
パッケージのコンパイル時にGoのソースファイルを使用するかどうか決める条件です。ビルド制約は、ファイル名のサフィックス（例: `foo_linux_amd64.go`）や、ビルド制約のコメント（例: `// +build linux,amd64`）で表現されます。[Build Constraints](https://golang.org/pkg/go/build/#hdr-Build_Constraints)をご覧ください。

### ビルドリスト（build list）
`go build`、`go list`、`go test`などのビルドコマンドで使用されるモジュールのバージョンのリストです。ビルドリストは、[メインモジュール](https://golang.org/ref/mod#glos-main-module)の[`go.mod`ファイル](https://golang.org/ref/mod#glos-go-mod-file)と、[minimal version selection](https://golang.org/ref/mod#glos-minimal-version-selection)を使用して推移的な操作を必要とするモジュール内にある`go.mod`ファイルから決定されます。ビルドリストには、特定のコマンドに関連するモジュールだけでなく、[モジュールグラフ](https://golang.org/ref/mod#glos-module-graph)にある全てのモジュールのバージョンが含まれています。

### 正規のバージョン（canonical version）
ビルドメタデータのサフィックスが`+incompatible`以外の、正しくフォーマットされたバージョンです。例えば、`v1.2.3`は正規のバージョンですが、`v1.2.3+meta`はそうではありません。

### カレントモジュール（current module）
[メインモジュール](https://golang.org/ref/mod#glos-main-module)と同義です。

### 非推奨のモジュール（deprecated module）
製作者によってサポートされなくなったモジュールです（ただし、メジャーバージョンはこの目的のために別のモジュールとみなされます）。非推奨のモジュールには、最新版の[`go.mod`ファイル](https://golang.org/ref/mod#glos-go-mod-file)に[非推奨のコメント](https://golang.org/ref/mod#go-mod-file-module-deprecation)がつけられます。

### `go.mod`ファイル（`go.mod` file）
モジュールのパス、必須のモジュール、その他のメタデータを定義するファイルです。[モジュールルートディレクトリ](https://golang.org/ref/mod#glos-module-root-directory)に配置されます。[`go.mod`ファイル](https://golang.org/ref/mod#go-mod-file)の項をご覧ください。

### インポートパス（import path）
Goのソースファイルでパッケージをインポートするために使われる文字列です。[パッケージパス](https://golang.org/ref/mod#glos-package-path)と同義です。

### メインモジュール（main module）
`go`コマンドが実行されるモジュールです。メインモジュールは、カレントディレクトリまたは親ディレクトリにある[`go.mod`ファイル](https://golang.org/ref/mod#glos-go-mod-file)で定義されます。[モジュール、パッケージ、バージョン](https://golang.org/ref/mod#modules-overview)をご覧ください。

### メジャーバージョン（major version）
セマンティックバージョンの最初の数字です（`v1.2.3`の`1`）。互換性のない変更のあるリリースでは、メジャーバージョンをインクリメントし、マイナーバージョンとパッチバージョンを0にしなければなりません。メジャーバージョンが0のセマンティックバージョンは、不安定とみなされます。

### メジャーバージョンのサブディレクトリ（major version subdirectory）
モジュールの[major version suffix](https://golang.org/ref/mod#glos-major-version-suffix)と一致するバージョンコントロールリポジトリ内のサブディレクトリで、モジュールを定義することができます。例えば、[root path](https://golang.org/ref/mod#glos-repository-root-path)が`example.com/mod`であるリポジトリのモジュール`example.com/mod/v2`は、リポジトリルートディレクトリやメジャーバージョンのサブディレクトリ`v2`に定義することができます。[Module directories within a repository](https://golang.org/ref/mod#vcs-dir)をご覧ください。

### メジャーバージョンサフィックス（major version suffix）
メジャーバージョンの数字と一致するモジュールパスのサフィックスです。例えば、`example.com/mod/v2`の`/v2`です。メジャーバージョンサフィックスは、`v2.0.0`以降では必須であり、それ以前のバージョンでは使用できません。[Major version suffixes](https://golang.org/ref/mod#major-version-suffixes)をご覧ください。

### minimal version selection (MVS)
ビルドで使用されるすべてのモジュールのバージョンを決定するアルゴリズムです。詳しくは、[Minimal version selection](https://golang.org/ref/mod#minimal-version-selection)をご覧ください。

### マイナーバージョン（minor version）
セマンティックバージョンの2番目の数字です（`v1.2.3`の`2`）。後方互換性のある新機能をリリースする際は、マイナーバージョンをインクリメントし、パッチバージョンを0にする必要があります。

### モジュール（module）
リリースされ、バージョン管理され、ひとまとまりで配布されるパッケージの集合体のこと。

### モジュールキャッシュ（module cache）
ダウンロードしたモジュールを保存するローカルディレクトリで、`GOPATH/pkg/mod`にあります。[Module cache](https://golang.org/ref/mod#module-cache)をご覧ください。

### モジュールグラフ（module graph）
[main module](https://golang.org/ref/mod#glos-main-module)をルートとする必須モジュールの有向グラフ。グラフの各頂点はモジュールで、各辺は`go.mod`ファイル中の`require`文を元にしたバージョンです（メインモジュールの`go.mod`ファイル中の`replace`文と`exclude`文に従う）。

### モジュールパス（module path）
モジュールを識別したり、そのモジュール内のパッケージインポートパスのプレフィクスとして機能したりするパス。例えば、`"golang.org/x/net"`のようになります。

### モジュールプロキシ（module proxy）
[`GOPROXY` protocol](https://golang.org/ref/mod#goproxy-protocol)を実装したWebサーバです。`go`コマンドは、モジュールプロキシからバージョン情報、`go.mod`ファイル、zip化されたモジュールのファイルをダウンロードします。

### モジュールルートディレクトリ（module root directory）
モジュールを定義する`go.mod`ファイルを含むディレクトリです。

### モジュールサブディレクトリ（module subdirectory）
[module path](https://golang.org/ref/mod#glos-module-path)のうち、[repository root path](https://golang.org/ref/mod#glos-repository-root-path)の後の部分で、モジュールが定義されているサブディレクトリを示します。空でない場合、モジュールサブディレクトリは[semantic version tags](https://golang.org/ref/mod#glos-semantic-version-tag)のプレフィックスにもなります。モジュールが[major version subdirectory](https://golang.org/ref/mod#glos-major-version-subdirectory)にあっても、[major version suffix](https://golang.org/ref/mod#glos-major-version-suffix)があれば、モジュールサブディレクトリはメジャーバージョンサフィックスを含みません。[Module paths](https://golang.org/ref/mod#module-path)をご覧ください。

### パッケージ（package）
同じディレクトリにあるソースファイルの集まりで、一緒にコンパイルされます。Go Language Specificationの[Packagesのセクション](https://golang.org/ref/spec#Packages)をご覧ください。

### パッケージパス（package path）
パッケージを一意に識別するためのパスです。パッケージパスは、モジュールパスとそのモジュール内のサブディレクトリを結合したパスです。例えば、`"golang.org/x/net/html"`はモジュール`"golang.org/x/net"`のサブディレクトリ`"html"`にあるパッケージのパッケージパスです。[import path](https://golang.org/ref/mod#glos-import-path)と同義語です。

### パッチバージョン（patch version）
セマンティックバージョンの3番目の数字です（`v1.2.3`の`3`）。モジュールの公開インターフェースに変更がないリリースでは、パッチバージョンをインクリメントする必要がります。

### プレリリースバージョン（pre-release version）
パッチバージョンの直後にダッシュが続くバージョンです。プレリリースバージョンは不安定とみなされ、ほかのバージョンとの互換性は想定されていません。プレリリースバージョンは、対応するリリースバージョンより前のバージョンです。すなわち、`v1.2.3-pre`は`v1.2.3`よりも前のバージョンです。[release version](https://golang.org/ref/mod#glos-release-version)もあわせてご覧ください。

### 疑似バージョン（pseudo-version）
バージョンコントロールシステムから取得したリビジョン識別子（Gitのコミットハッシュなど）とエンコードされたタイムスタンプを用いるバージョンです。例えば、`v0.0.0-20191109021931-daa7c04131f5`のようになります。モジュールを持たないリポジトリとの互換性や、タグつきバージョンが利用できない場合などに使用します。

### リリースバージョン（release version）
プレリリースのプレフィックスがついていないバージョンです。例えば、`v1.2.3-pre`ではなく`v1.2.3`です。[pre-release version](https://golang.org/ref/mod#glos-pre-release-version)もあわせてご覧ください。

### リポジトリルートパス（repository root path）
モジュールパスのうち、バージョンコントロールリポジトリのルートディレクトリに対応する部分です。[Module paths](https://golang.org/ref/mod#module-path)をご覧ください。

### 廃止バージョン（retracted version）
公開前だったり、公開後に重大な問題が発見されたりされ、依存すべきではないバージョンのことです。[`retract` directive](https://golang.org/ref/mod#go-mod-file-retract)をご覧ください。

### セマンティックバージョンタグ（semantic version tag）
[version](https://golang.org/ref/mod#glos-version)を明示したリビジョンに関連付けるバージョンコントロールリポジトリのタグです。[Mapping versions to commits](https://golang.org/ref/mod#vcs-version)をご覧ください。

### ベンダーディレクトリ（vendor direstory）
メインモジュールのパッケージをビルドするために必要なほかのパッケージを格納する`vendor`という名前のディレクトリです。[`go mod vendor`](https://golang.org/ref/mod#go-mod-vendor)で管理されています。[Vendoring](https://golang.org/ref/mod#vendoring)をご覧ください。

### バージョン（version）
モジュールの不変的なスナップショットを示す識別子で、`v`の後にセマンティックバージョンを続けて記述します。[Versions](https://golang.org/ref/mod#versions)をご覧ください。
