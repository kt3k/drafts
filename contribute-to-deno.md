# Deno にコントリビュートする

<!--
想定読者:
- TypeScript にある程度自信ありな人
- Rust は得意では無いけど, やれば出来ると思っている人
- プログラムの腕に自信ありだけど, まだ大きい OSS にはそこまで手を出せていない人
- OSS にコントリビュートするのはまだちょっと怖いけど, 背伸びをしてみたい気持ちもある人
-->

Deno Advent Calendar 8日目の記事です.

今日は Deno にコントリビュートするためのヒントを紹介します.

# Deno 本体をビルドする

Deno にコントリビュートするためには, まずは Deno をビルドしてみましょう.

Deno 本体をビルドするためには以下のツールが必要になります.

- Rust 1.39
- Python 2.7
  - Python 3 はダメなので必ず2系をインストールしてください (ただし来年以降は3になる可能性があります)

ビルドには必要ありませんが lint やフォーマッティングのために更に以下が必要になります.

- Node.js 12

## Deno をビルドする

Deno のリポジトリは以下のコマンドで clone できます.

```sh
git clone https://github.com/denoland/deno.git
```

Deno レポジトリはいくつかの git サブモジュールを使っています. 以下のコマンドでサブモジュールの初期化と更新をしましょう.

```sh
cd deno
git submodule update --init
```

Deno は主に Rust 言語と TypeScript 言語で書かれていますが, そのコア部分は Rust です. したがって Deno は通常の Rust のプロジェクトと同様に cargo コマンドを使ってビルドします.

```
cargo build
```

ビルドが成功すると `target/debug/deno` に deno コマンドが生成されます.

```console
./target/debug/deno -v
deno: 0.24.0
v8: 8.0.192
typescript: 3.7.2
```

ビルドが無事に成功したら次にテストを実行してみましょう. テストの実行方法は通常の rust プロジェクトと同様に cargo コマンドから実行します.

```sh
cargo test
```

なお特定のパターンを含むテストだけを実行したい場合は以下のように実行できます.

```sh
cargo test foo_pattern
```

全てのテストがパスするには数分程度の時間がかかるはずです. 全てのテストが通ったら, コントリビュートする準備が整ったと言えます. うまくいかなかった場合は, 必要ツールのインストールを忘れていないかなどを確認してみましょう.

## 課題の探し方

次にコントリビュートする課題の探し方のヒントを紹介します.

### TODO コメント

おそらく一番取り組みやすい課題は TODO コメントを解決することです. 中にはどうやって解決するのか分からないような TODO コメントもまれにありますが, 多くの場合 TODO コメントにはかなり具体的に直すべき課題が書かれています. この記事を書いている現時点で master ブランチには 323個の TODO コメントが登録されています. いくつかを一部抜粋すると以下のようなものがあります.

```
cli/js/read_link_test.ts:  // TODO Add test for Windows once symlink is implemented for Windows.
cli/js/remove_test.ts:  // TODO(ry) Is Other really the error we should get here? What would Go do?
cli/js/repl.ts:// TODO(kevinkassimo): this list might not be comprehensive
cli/js/repl.ts:  // TODO(kevinkassimo): need a parser to handling errors such as:
cli/js/stat_test.ts:// TODO Add tests for modified, accessed, and created fields once there is a way
cli/js/streams/mod.ts:/* TODO The following are currently unused so not exported for clarity.
cli/js/streams/pipe-to.ts:// TODO reenable this code when we enable writableStreams and transport types
cli/js/streams/pipe-to.ts:// // TODO reenable this lint here
cli/js/streams/pipe-to.ts://       // TODO this should be a DOMException,
cli/js/streams/queue-mixin.ts:// TODO reenable this lint here
cli/js/streams/readable-byte-stream-controller.ts:// TODO reenable this lint here
cli/js/streams/readable-internals.ts:// TODO reenable this lint here
cli/js/streams/readable-internals.ts:/* TODO reenable this when we add WritableStreams and Transforms
cli/js/streams/readable-internals.ts:  /* TODO reenable these methods when we bring in writableStreams and transport types
cli/js/streams/readable-internals.ts:  // TODO remove the "as unknown" cast
cli/js/streams/readable-internals.ts:    // TODO remove the "as unknown" cast
cli/js/streams/readable-internals.ts:    // TODO remove the "as unknown" cast
cli/js/streams/readable-internals.ts:  // TODO remove the "as unknown" cast
cli/js/streams/readable-stream-default-controller.ts:// TODO reenable this lint here
cli/js/streams/readable-stream.ts:// TODO remove this, surpressed because of
cli/js/streams/readable-stream.ts:  /* TODO reenable these methods when we bring in writableSt
```

型に関すること, lint に関すること, テストに関すること, CI に関すること, etc など様々なことが, TODO コメントとして残っています.

なお, 以下のような形式の TODO コメントがあります.

```
TODO(kevinkassimo): this list might not be comprehensive
```

この場合 () の中に書かれている名前は, この TODO コメントを残した人の名前です.

### Issue あさり

次に取り組みやすい課題は GitHub の issue に取り組むことです. Issue は TODO コメントよりも粒度が大きいものが多いため, かなり取捨選択して見ていかないとなかなか取り組める課題を見つけることは難しいかもしれません. 中には議論をするための issue や, ただ夢が語られているだけの issue などもあります. うまくふるい分けながら見ていきましょう.

Issue の中には[1.0 マイルストーン](https://github.com/denoland/deno/issues/2473)に組み込まれているものがあります. これは現在 Deno チームの中で最も優先度が高いものです. 難しい機能もありますが, 単純なバグフィクスの issue もあるので, 必ずしもコアメンバーしか手が出せない issue という訳ではありません. 今後 Deno に大きく貢献したい人, コアメンバーを目指したい人はこれらの issue を眺めてみるのも良いかもしれません.

### API の提案

Deno はまだ API が整っているとは言えない状態です. 特に Node のコアに含まれている機能で, まだ Deno には搭載されていない機能がいくつもあります. そのようなものを Deno に対して提案すれば採用される可能性が非常に高いです. Node の API ドキュメントと Deno の API ドキュメントを見比べならがら足りないものを見つけて提案しましょう.

### 標準ライブラリの提案

Deno は Node とは違って標準ライブラリを持ちます. 標準ライブラリは Node にはなかった概念で, Python や Go 言語の標準ライブラリの考え方に近いものです. Deno の標準ライブラリは, まだ充実しているとは全く言えない状態です. Deno の標準ライブラリは Deno レポジトリの `std` ディレクトリの中で実装されています. 自分が知っている npm モジュールや, よく使う npm モジュール, 基本的だと思う npm モジュールの中で std の中に存在していないものがないか考えてみましょう. もしそういうものを思い付いたら, 標準ライブラリとして提案してみましょう. 採用される可能性は十分にあります.

## Pull Request の投げ方

うまく取り組む課題を見つけられたら Pull Request を作成しましょう. Pull Request の投げ方は一般的な OSS と同様です. Description で何をしたか説明しましょう. 解決出来る issue がある場合は issue へのリンクを貼りましょう. TODO コメントを解決している場合も, その旨を description で明記するのが無難です. OSS で一番困るのは目的の分からない Pull Request です. Deno ではよく分からない PR が飛んできた場合でも, いきなり閉じられることはありませんが, レビューが滞留し, レビュワーの負担になります. なぜその修正を入れるべきなのかが分かる説明を頑張って書きましょう.

Pull Request を出す時にもう一つやらなければならないことは CI を通すことです. Deno の CI は一般的な OSS と比べて比較的通りにくい部類に入るのではないかと思います. lint エラーやコード整形のし忘れがある場合は CI で検出されるようになっています. Lint チェック, コード整形はそれぞれ以下のコマンドで実行できます.

```sh
./tools/lint.py
```

```sh
./tools/format.py
```

CI が落ちてしまった場合はローカルでこれらのコマンドを叩いてみて, 出てきた指示に従ってコードを修正していきましょう.

また, Deno のテストは一部確率的に落ちてしまうものが含まれてることがあります (このようなテストは flaky なテストと呼ばれます). 自分が作業ブランチを切ったタイミングで運悪くそのようなテストを抱えてしまっている場合は空のコミットを push して CI を再起動してみましょう. 以下のコマンドで git の空のコミットを作ることが出来ます.

```sh
git commit --allow-empty
```

CI が無事に通ると(あるいは通らなくても) Deno チームメンバーから何かしらのフィードバックがあるはずです. 修正依頼がある場合は指示に従って修正するなり, 修正したくないなりの理由を述べましょう. この辺りは一般的な OSS と同様です. 議論が終わると多くの場合はマージされます. (OSS によっては, 散々議論してレビュー対応した結果マージせずリジェクトというようなケースもたまに見かけますが, Deno の場合はそういう場面はあまり見たことがありません. 修正依頼が出ている時点で方針としてはマージする方向に向かっていると考えて良い場合が多いでしょう.)

# まとめ

今日は Deno にコントリビュートするためのヒントを紹介しました.

今日は主に Deno 本体へのコントリビュートの仕方を中心に紹介しましたが, それ以外にも Deno にコントリビュートする様々な形があります. 明日は Deno 本体の修正を投げる以外の Deno へのコントリビュートの仕方を紹介します.
