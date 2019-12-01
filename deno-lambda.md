# deno-lambda を使ってみる

Deno Advent Calendar 4日目の記事です.

今日は Deno で AWS Lambda 関数を実装するためのツール [deno-lambda](https://github.com/hayd/deno-lambda) の使い方を紹介します.

# 手順

[deno-lambda のリリースページ](https://github.com/hayd/deno-lambda/releases) に行き, 最新バージョンの deno-lambda-layer.zip をダウンロードしてください.

AWS Console の Lambda のページに行き Layers タブを選択し, Layer の作成選んでください.

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step0.png" />

任意の名前を入力し(例. deno-lambda-layer), ダウンロードした deno-lambda-layer.zip をアップロードしてください. アップロード完了したら作成ボタンを押して Lambda layer を作成します.

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step1.png" />

作成が完了したら画面右上のレイヤーの ARN をコピーして控えます.

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step2.png" />

次に Lambda 関数を作ります. 1から作成を選んで, ランタイムは「ユーザー独自のブートストラップ」を選択してください.

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step3.png" />

Lambda 関数が出来たら, Layers パネルから Layer の追加を選びます.

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step4.png" />

Layer の追加画面で, 「レイヤーバージョン ARN を提供」を選択し, レイヤーバージョン ARN に先ほど控えたレイヤーの ARN を入力し, レイヤーを追加します.

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step5.png" />

次に Lambda 関数を書いていきます. 関数の入力欄を選択し, ハンドラーを `handler.main` と入力し, handler.ts というファイルを作成します. handler.ts の内容は以下の内容を入力してください.

```ts
import { Context, Event } from "https://deno.land/x/lambda/mod.ts";

export function main(event: Event, context: Context) {
  return {
    statusCode: 200,
    body: `Welcome to deno ${Deno.version.deno} 🦕`
  };
}
```

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step6.png" />

以上で Lambda 関数の作成は完了です. 上のソースコードの main 関数部分が Lambda のハンドラーになっています. ここでは, event (リクエスト) の内容に依らず, 成功レスポンスを返す実装になっています.

この Lambda 関数をテストするために画面右上の「テスト」 から, テスト用のイベントを作りつつテスト実行してみましょう. 以下のような実行結果が出てくれれば成功です.

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno-lambda/step7.png" />

うまくいかない場合は, エラーメッセージを見ながら, ここまでの設定で何か忘れていないかを確認してみましょう. それでもわからない場合は [deno-ja slack](https://scrapbox.io/deno-ja/Slack%E3%81%AE%E5%8F%82%E5%8A%A0%E6%96%B9%E6%B3%95) で聞くか, deno-lambda に issue を立ててみましょう.

# Recap

今日は @hayd Andy Hayden の deno-lambda の使い方を紹介しました.
