# Deno のディレクトリ構造 2019 冬

<!--
想定読者:
- Deno のコードを1回はどこかしら読んだ人
- 1回は読んだけどその後離れてしまい, 最新は追っていない人
- 今はコントリビュートする気はないけど, とりあえず雰囲気は押さえておきたい人
-->

Deno (ディーノ) <img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno.png" width="20"> Advent Calendar 23日目の記事です.

今日は Deno のディレクトリ構造について紹介します.

Deno のディレクトリ構造は何度も劇的に変わっています. 今の構造が今後ずっと続くという保証は全くありません. むしろ Deno の進化のフェーズに合わせて今後もどんどん変化していくでしょう. この記事では, 今日の時点での Deno のディレクトリ構造について解説します.

ディレクトリ構造の概略は以下です.

```
deno/
├── core - - - - - - - - - - - Deno のコア機能の実装
│   └── libdeno  - - - - - - - Deno のコア機能のうち, V8 の native binding に関する実装
├── cli  - - - - - - - - - - - Deno の CLI としての各種機能
│   ├── compilers  - - - - - - ts コンパイル機能の実装
│   ├── js - - - - - - - - - - ts で書かれた Deno の API の実装
│   ├── ops  - - - - - - - - - Deno の Op の実装
│   └── tests  - - - - - - - - CLI としての結合テストの実装
├── deno_typescript  - - - - - ts のバンドルとV8 スナップショット生成
├── std  - - - - - - - - - - - Deno の標準ライブラリ実装
├── third_party  - - - - - - - Deno の 3rd party 依存 (submodule)
└── tools  - - - - - - - - - - Deno の開発用各種スクリプト類
```

以下では各ディレクトリの役割について見ていきます.

# core/

`core/` は Deno のコア機能だけを含んだ rust のライブラリです. コア機能には rust から V8 を起動する機能や, V8 から Op という単位のリクエスト/レスポンスをやり取りするための機能が含まれます. 逆に Deno.readFile などのような Deno 名前空間の API などはコア機能には含まれていません. また, コマンドラインツールとしての機能などもコア機能には含まれません. それらの機能は後述の `cli/` というディレクトリの中で実装されています.

`core/` は rust の crate として crates.io に[パブリッシュ](https://crates.io/crates/deno)されています. core を再利用して, V8 バインディング機能だけを再利用した, Deno とは違った API 体系を持つ JS 処理系を作ることが出来るようになっています.

# core/libdeno/

`core/libdeno/` には V8 バインディングが C++ で書かれています. Node.js はこの層で各種の Native API を実装していますが, Deno ではこの層では print, send, recv など必要最小限の API 実装にとどめています. この層で実装した最低限の API を使って V8 と Rust がブリッジされるようになっています.

なお, 現在 denoland org では, [rusty_v8](https://github.com/denoland/rusty_v8) という rust crate が活発に開発されています. この crate は V8 の API をより細やかに rust でラッピングするライブラリで, 将来的には libdeno の実装は, この rusty_v8 を使って rust での実装で置き換えられることになります. そのための [issue](https://github.com/denoland/deno/issues/3530) が, ちょうどこの記事の執筆中に作られました.

# cli/

`cli/` は Deno の各種 API やコマンドラインツールとしての機能が実装される場所です. このディレクトリの直下には Deno CLI の Rust 側の挙動が実装されています. パーミッション管理やリソース管理など, Deno の各種機能が実装されています.

# cli/js/

`cli/js/` 以下には Deno の API の TypeScript 側の実装が配置されています. 典型的には Deno の Op を dispatch して結果を適切に返すという形の API 実装が多くを占めますが, EventTarget の実装などのように完全に TypeScript で完結して実装されている API も多数あります.

# cli/ops/

Deno は Op という仕組みを使って TypeScript と Rust が通信し合うような形で実装されています. `cli/ops/` ディレクトリの中では, Rust 側の Op の登録処理や, Op のハンドリング, Op から V8 へのレスポンス処理などが実装されます.

# cli/compiler/

`cli/compiler/` 以下では TypeScript のコンパイル機能が実装されています. TypeScript コンパイラーは Deno が実行される Isolate (V8 のインスタンス) とは別な Isolate 内で実行されるため, メインの Deno API 群とは独立しており, そのためディレクトリ単位で分離されています.

# cli/tests/

`cli/tests/` では Deno の CLI としてのインテグレーションテストが記述されています. hello world 的なものから, エラー処理のチェック, モジュールロードのチェック, パーミッション関連のチェックなど, 様々なテストケースがチェックインされています.

このディレクトリ下のテストは基本的には Deno を CLI ツールとして起動し, その標準出力/標準エラー出力, exit code が適切かをアサーションするという形のテストになっているため, ケースごとの実行が遅く, 出来るだけこのディレクトリのテストケースは増やすべきではないという方針で開発が進められています. ただし, コマンドラインツールとしてのライフサイクルに関わるようなテストなど, どうしてもユニットテストは書きにくいというものについてはこのディレクトリにテストを追加することが必要になります.

# tools/

`tools/` には Deno をビルドするために必要な各種ツールが配置されています. ツールの大部分は現状では Python2 で書かれています. Python で書かれている理由はクロスプラットフォームで安定して使えるスクリプト言語ということで, Python が選ばれています. また, Python 3 ではなくて 2 である理由は, V8 のビルドが Python 2 に依存しているからという理由です (なお V8 は現在 Python 3 移行作業の最中のため, 近い将来 Deno の Python も 3 に書き換わる可能性があります).

現状ではほとんどの開発に必要なスクリプトが Python で書かれていますが, 一部のビルドスクリプトは現在は rust で書き直されています (たとえばメインのビルドは完全に rust 化されています). 方針としてはできるだけ多くの処理を rust でやりたいという方向になっていて, そのための [issue](https://github.com/denoland/deno/issues/2988) が立っています.

# third_party/

`third_party/` は Deno の 3rd party 依存を全て格納したディレクトリです. このディレクトリは deno_third_party という別のレポジトリを submodule としてリンクしています. 以前はここに多くの npm モジュールが配置されていましたが, いまでは npm モジュールはリント関連だけが残っています. また, 以前は rust crate の依存解決が独自ルールで行われていたため, rust crate のソースコード一式が全てここに含まれていましたが, 現在は cargo コマンドで解決されるようになったため, `third_party/` から rust crate は無くなりました.

今このディレクトリに残っているものは V8 のソースコードとビルド用スクリプト類, リントに使っている npm modules, tools で使っている python packages などですが, いずれも将来的には無くなる方針のため, 将来的には third_party という依存の管理方法は無くなることが予想されます.

# std

`std/` では Deno の標準モジュールが実装されています. 標準モジュールは基本的には TypeScript で書かれています. 一部, 外部からコピーで持ってきたスクリプト類 (prettier など) は .js になっているものもあります.

標準モジュールはまだまだ発展途上のため, これからいろいろなモジュールを提案する余地があります. また, 多くの標準モジュールがほとんど誰も実際に使っていない, というようなものが多いと思われるため, バグを踏む可能性はまだまだ多いです. 筆者 <img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/kt3k.jpg" width="20" /> が先日 YAML モジュールの parseAll という API を使おうとしたところ, [全く動いておらず](https://github.com/denoland/deno/issues/3534), テストも書かれておらず, この API はおそらく誰も叩いたことがなかったであろうことが露見するということがありました ([自分で直しました](https://github.com/denoland/deno/pull/3535)). 良く言えば, いくらでもコントリビュートできる, 悪く言えば, ほぼ使い物にならないのが今の標準モジュールの状態です.

# deno_typescript

このディレクトリには Deno の API の ts 実装のための専用のバンドラープログラムが含まれています. 以前は Deno の API のバンドルには rollup (その前は parcel) が使われていましたが, できるだけ node の依存を減らしたいという方針があるため, deno のコア機能と TypeScript コンパイラーと AMD のローダを組み合わせて (TS 以外の)外部ツールに依存せずに Deno API をバンドルしているのが deno_typescript です.

# まとめ

今日は Deno のディレクトリ構造について説明しました. 明日は @chibato さんの記事です.
