# Deno にコントリビュートする (2)

<!--
想定読者:
- Deno に興味がある一般の人
- Deno に興味を持っているけど, 本体を clone してビルドする気はない人
- OSS になんでもいいからコントリビュートしてみたい気持ちはあるけど, 何をしていいのかわからない人
-->

Deno Advent Calendar 9日目の記事です.

昨日の記事では Deno 本体にコントリビュートする方法を紹介しましたが, Deno 本体に PR を投げることだけがコントリビュートではありません. 本記事では Deno 本体以外で Deno にコントリビュートする方法を紹介します.

# Deno のホームページ

[Deno のホームページ][hp]のソースコードは以下のコマンドで clone 出来ます.

```sh
git clone https://github.com/denoland/deno_website2.git
```

Deno のホームページは一般的な React プロジェクトです. 通常の React プロジェクトと同様に実行することが出来ます.

```sh
cd deno_website2
yarn
yarn start
```

デザインで気になった点, 表現で気になった点など, [ホームページ][hp] で気になった点を修正して, PR を投げてみましょう. またホームページの [issue](https://github.com/denoland/deno_website2/issues) もあるため, そちらを解決するのもおもしろいかもしれません.

# アイコン

@hashrock さんが下の Deno のアイコンをコントリビュートしたことは Deno コミュニティの中では有名です. 公式ドキュメントの中でも使われています.

<img src="https://deno.land/images/deno-circle-thunder.gif" />

@tanakaworld さんの下の Deno アイコンも公式アイコンとして Deno コミュニティに受け入れらています. 公式ドキュメントや, カンファレンスでの発表資料などで使われています.

<img src="https://deno.land/images/deno_logo_4.gif" />

画力に自信のある人はこういうコントリビュートの仕方もあります.

# 3rd パーティライブラリ

https://deno.land/x/ という URL に行くと Deno の 3rd party モジュールが列挙されています. ここは Deno のための (とても小さな) npm のようなページで, Deno で書かれた任意のライブラリはここに載せることが出来ます.

このリストに自分のライブラリを追加したい場合は [こちら](https://github.com/denoland/deno_website2/blob/master/src/database.json) の json ファイルに対して PR を投げて, 自分のライブラリの情報を追加してください. Deno エコシステムに貢献したことになります.

# コードリーディング

コードリーディング記事をまとめることも立派なコントリビュートです.

日本語では過去に以下のようなコードリーディング記事が上がってきています.

- [Deno をよむ(1)](https://blog.bokuweb.me/archive/2019/01/11) by bokuweb
- [deno_code_reading.md](https://gist.github.com/mizchi/31e5628751330b624a0e8ada9e739b1e) by mizchi
- [Dive into Deno：プロセス起動からTypeScriptが実行されるまで](https://blog.leko.jp/post/code-reading-of-deno-boot-process/) by leko
  - 上の改訂版が Denobook 2 第5章でよめます.

Denobook 2 の Leko さんの記事が最も最新の内容に近いですが, それ以降も微妙に変わってきているため, 上記を踏まえた最新のコードリーディング記事が出てくると, Deno コミュニティに大きく貢献したことになります.

# チャットに参加

https://deno-ja-slackin.herokuapp.com

この URL から, deno の日本コミュニティの slack に参加できます. #random チャネルなどで Deno に対する質問/意見/感想を述べてコミュニティのメンバーと会話をしてみましょう. コミュニティの活動を盛り上げることも立派なコントリビューションと言えるでしょう.

# wiki を読む/更新する

Deno 日本コミュニティは [scapbox](https://scrapbox.io/deno-ja/) に各種 tips を蓄積しています. scrapbox の更新も大切なコントリビューションの1つです.

# まとめ

今日は本体にコントリビュートする以外での Deno にコントリビュートする方法を紹介しました.

明日は @keroxp さんの「Denoと過ごした１年、現状そして展望」です.

[hp]: https://deno.land/
