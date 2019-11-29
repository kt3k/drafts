# Deno のディレクトリ構造 2019 冬

<!--
想定読者:
- Deno のコードを1回はどこかしら読んだ人
- 1回は読んだけどその後離れてしまい, 最新は追っていない人
- 今はコントリビュートする気はないけど, とりあえず雰囲気は押さえておきたい人
-->

Deno Advent Calendar 24日目の記事です.

今日は Deno のディレクトリ構造について紹介します.

Deno のディレクトリ構造は何度も劇的に変わっていますので, 今の構造が今後ずっと続くという保証は全くありません. むしろ Deno の進化のフェーズに合わせて今後もどんどん変化していくでしょう. この記事では, 今日の時点での Deno のディレクトリ構造について解説します.

ディレクトリ構造の概略は以下です.

```
deno/

```

以下では各ディレクトリの役割について見ていきます.

# core

core という単位は意外と理解が難しいです. core には Deno のコマンドラインとしての機能は含まれていません. また Deno の基本的な API も含まれていません (Deno namespace の機能など). core は大雑把にいうと rust から V8 を使うための機能と rust から V8 にコマンドを dispatch するための最低限の仕組みだけが入っています. コマンドを dispatch する仕組みはありますが具体的なコマンドは何も入っていないため, これだけで出来ることは殆どなく, rust に V8 を組み込んだ何かを作るための基礎的な仕組みのみが抽出されたものが core です. なぜ core という単位を作るのかはあまり具体的な説明がされたことは無いように思いますが, core はそれ単独で rust の crate として publish されています. つまり core の rust crate を利用して, Deno とは全く違った API 体系をもった rust で書かれた V8 ベースの全く違う処理系を作ることが出来るような形になっています. もしかしたら mongodb のような shell として JS エンジンを持った DB など, Deno とは違った Deno core crate の利用方法が意図されているのかもしれません.

# core/libdeno

V8 バインディングが書かれています. 主に C++ です. Node.js はこの層で各種の Native API を実装していますが, Deno ではこの層では print / send / recv など必要最小限の API 実装にとどめています. この層で実装した最低限の API を Rust からラップして主要部の実装を Rust に寄せるというのが Deno の方針です.

# cli

Deno の API が実装される場所です.

# cli/js

Deno の API の TypeScript 側の実装が配置されています. 典型的には Deno の Op を dispatch して結果を適切に返すという形の API 実装が多くを占めますが, EventTarget の実装など完全に TypeScript で完結して実装されている Deno の API なども多数あります.

# tools

Deno をビルドするために必要な各種ツールが配置されています. ツールの大部分は現状では Python2 で書かれています. Python で書かれている理由はクロスプラットフォームで安定して使えるスクリプト言語として Python が選ばれたという経緯があります. また, Python 3 ではなくて 2 である理由は, V8 のビルドが Python 2 に依存しているからという理由があります. Chrome チームは現在急ピッチで Python 2 の EOL 対応 (Python 3 移行) を進めている気配があるため, 来年には Deno の Python スクリプトも Python 3 に移行している可能性が高いと思われます.

# third_party

3rd party 依存を列挙したディレクトリです. 以前はここに多くの npm モジュールが配置されていましたが, いまでは npm モジュールはリント関連だけが残っている状態です. また, 以前は rust crate の依存解決が独自ルールで行われていたため, rust crate のソースコード一式が全てここに含まれていましたが, 現在は cargo コマンドで解決されるようになったため, third_party から rust crate は無くなりました.

# std

Deno の標準モジュールを実装する場所です. Deno の標準モジュールは全て TypeScript で書かれています. 従って, TypeScript が分かれば Deno の標準モジュールのコードは基本的に全て読んで理解することが出来ます. 標準モジュールはまだまだ発展途上のため, これからいろいろなモジュールを提案する余地があります.

# deno_typescript

このディレクトリにあるのはかなり不思議なプログラムです. このディレクトリにあるプログラムは基本的には Deno の TypeScript で実装されたスクリプト群のバンドラーのようなものですが, とても面白い挙動をします. バンドラーというと browserify / webpack / rollup のような JavaScript のモジュール関係を解決した大きな JavaScript ファイルを返すプログラムを指しますが, deno_typescript はバンドルした結果を V8 スナップショットというバイナリ形式で返します. 具体的に何をしているかというと, deno core に typescript を読み込ませた V8 isolate を作りこれに typescript で書かれた cli/js 以下のファイルをコンパイルさせます. そして snapshot を作るための別の V8 isolate にコンパイルしたコードを次々と load して行きます. これには V8 の loadModule という API が利用できます. 最後に必要なスクリプトが全てロードされた V8 isolate でスナップショットを取ります. このスナップショットを Deno 本体にバイナリとしてバンドルすることで cli/js 以下の Deno 実装をまとめて1ファイルとしてバンドルしたかのようになるのです. つまり Deno は V8 自体をスクリプトのバンドラーとして使っていると見ることが出来ます.