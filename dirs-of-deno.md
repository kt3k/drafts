# Deno のディレクトリ構造 2019 冬

<!--
想定読者:
- Deno のコードを1回はどこかしら読んだ人
- 1回は読んだけどその後離れてしまい, 最新は追っていない人
- 今はコントリビュートする気はないけど, とりあえず雰囲気は押さえておきたい人
-->

Deno (ディーノ) <img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno.png" width="20"> Advent Calendar 23日目の記事です.

今日は Deno のディレクトリ構造について紹介します.

Deno のディレクトリ構造は何度も劇的に変わっていますので, 今の構造が今後ずっと続くという保証は全くありません. むしろ Deno の進化のフェーズに合わせて今後もどんどん変化していくでしょう. この記事では, 今日の時点での Deno のディレクトリ構造について解説します.

ディレクトリ構造の概略は以下です.

```
deno/
├── core - - - - - - - - - - - Deno のコア部分の実装
│   └── libdeno  - - - - - - - Deno のコア部分のうち, V8 の native binding に関する実装
├── cli  - - - - - - - - - - - Deno の CLI としての各種機能
│   ├── compilers  - - - - - - TypeScript コンパイル機能の実装
│   ├── js - - - - - - - - - - TypeScript で書かれた Deno の API の実装
│   ├── ops  - - - - - - - - - Deno の Ops の実装
│   └── tests  - - - - - - - - CLI としての結合テストの実装
├── deno_typescript  - - - - - V8 スナップショット生成
├── std  - - - - - - - - - - - Deno の標準ライブラリ実装
├── third_party  - - - - - - - Deno の 3rd party 依存
└── tools  - - - - - - - - - - Deno の開発用各種スクリプト類
```

以下では各ディレクトリの役割について見ていきます.

# core/

core という単位は意外と理解が難しいです. `core/` には Deno のコマンドラインとしての機能は含まれていません. また Deno の基本的な API も含まれていません (`Deno` namespace 以下にある機能など). core は大雑把にいうと rust から V8 を使うための機能と rust で V8 からのコマンド (Op) を受け取るための最低限の仕組みだけが入っています. コマンド (Op) を受け取る仕組みはありますが具体的なコマンドは何も入っていないため, これだけで出来ることは殆どなく, rust に V8 を組み込んだ何かを作るための基礎的な仕組みのみが抽出されたものが core です.

core を分離する理由は, V8 を rust から使うための最低限の機能を crate として publish したいという意図があるようです. この crate を使って具体的にどういうプログラムが書かれることが意図されているのかはわかりませんが (作者も特にイメージはないのかもしれません), とにかくベースの仕組みだけを再利用可能な形で crates.io に publish しています.

<!--
なぜ core という単位を作るのかはあまり具体的な説明がされたことは無いように思いますが, core はそれ単独で rust の crate として publish されています. つまり core の rust crate を利用して, Deno とは全く違った API 体系をもった rust で書かれた V8 ベースの全く違う処理系を作ることが出来るような形になっています. もしかしたら mongodb のような shell として JS エンジンを持った DB など, Deno とは違った Deno core crate の利用方法が意図されているのかもしれません.
-->

# core/libdeno/

`core/libdeno/` には V8 バインディングが書かれています. 主に C++ です. Node.js はこの層で各種の Native API を実装していますが, Deno ではこの層では print, send, recv など必要最小限の API 実装にとどめています. この層で実装した最低限の API を使って V8 と Rust がブリッジされるようになっています.

# cli/

`cli/` は Deno の各種 API やコマンドとしての機能が実装される場所です. この cli ディレクトリの直下には Deno の Rust 側の挙動が実装されています. パーミッション管理やリソース管理など, Deno のコアになる機能がここで実装されています.

# cli/js/

`cli/js/` 以下には Deno の API の TypeScript 側の実装が配置されています. 典型的には Deno の Op を dispatch して結果を適切に返すという形の API 実装が多くを占めますが, EventTarget の実装など完全に TypeScript で完結して実装されている API も多数あります. (TypeScript の知識だけでも Deno 本体にコントリビュートすることが十分可能です.)

# cli/ops/

Deno は Ops という仕組みを使って TypeScript と Rust が通信し合うような形で実装されています. `cli/ops/` ディレクトリの中では, Rust 側の Ops の登録処理や, Ops のハンドリング, Ops から V8 へのレスポンス処理などが実装されます.

# cli/compiler/

`cli/compiler/` 以下では TypeScript のコンパイル機能が実装されています. TypeScript コンパイラーは Deno が実行される Isolate (V8 のインスタンス) とは別な Isolate 内で実行されるため, メインの Deno API 群とは独立しており, そのためディレクトリ単位で分離されています.

# cli/tests/

`cli/tests/` では Deno の CLI としてのインテグレーションテストが記述されています.

# tools/

`tools/` には Deno をビルドするために必要な各種ツールが配置されています. ツールの大部分は現状では Python2 で書かれています. Python で書かれている理由はクロスプラットフォームで安定して使えるスクリプト言語なので, Python が選ばれました. また, Python 3 ではなくて 2 である理由は, V8 のビルドが Python 2 に依存しているからという理由です (Deno のビルドは V8 のビルドに依存しています). Chrome チームは現在急ピッチで Python 2 の EOL 対応 (Python 3 移行) を進めている気配があるため, 来年には Deno の Python スクリプトも Python 3 に移行している可能性が高いです.

現状ではほとんどの開発に必要なスクリプトが Python で書かれているのですが, 一部のビルドスクリプトは現在は rust で書き直されています (たとえばメインのビルドは完全に rust 化されています). 方向性的には全てのビルドを rust に書き直す方針になっていて, そのための [issue](https://github.com/denoland/deno/issues/2988) が立っています.

# third_party/

`third_party/` は Deno の 3rd party 依存を全て格納したディレクトリです. このディレクトリは deno_third_party という別のレポジトリを submodule としてリンクしています. 以前はここに多くの npm モジュールが配置されていましたが, いまでは npm モジュールはリント関連だけが残っています. また, 以前は rust crate の依存解決が独自ルールで行われていたため, rust crate のソースコード一式が全てここに含まれていましたが, 現在は cargo コマンドで解決されるようになったため, `third_party/` から rust crate は無くなりました. 今ではこのディレクトリには V8 関連の諸々と少しの npm modules と python packages だけが含まれています.

# std

`std/` は Deno の標準モジュールが置かれている場所です. Deno の標準モジュールは全て TypeScript で書かれています. 従って, TypeScript が分かれば Deno の標準モジュールのコードは基本的に全て読んで理解することが出来ます. 標準モジュールはまだまだ発展途上のため, これからいろいろなモジュールを提案する余地があります.

# deno_typescript

このディレクトリにあるのはかなり不思議なプログラムです. このディレクトリにあるプログラムは Deno 本体の TypeScript のバンドラーの役割を担っています. バンドラーというと browserify / webpack / rollup のような複数の JavaScript ファイルをつなげて大きい JavaScript ファイルを出力するプログラムが思い浮かびますが, deno_typescript はバンドルした結果を V8 スナップショットというバイナリ形式で返します. 具体的に何をしているかというと, deno core に TypeScript 本体を読み込ませた V8 isolate を作りこれに TypeScript で書かれた `cli/js/` 以下のファイルをコンパイルさせます. そして snapshot を作るための別の V8 isolate にコンパイルしたコードを次々と ES モジュールとして load していきます. 最後にスクリプトをロードした V8 isolate のスナップショットを取ります. このスナップショットを Deno 本体にバイナリとしてバンドルすることで `cli/js/` 以下の API 実装(JS 側)をまとめて1ファイルとしてバンドルしています. つまり Deno は V8 自体をスクリプトのバンドラーとして使っていると見ることが出来ます.

# まとめ

今日は Deno のディレクトリ構造について説明しました.
