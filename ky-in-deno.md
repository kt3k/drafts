# ky を Deno から使う

Deno (ディーノ) <img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno.png" width="20"> Advent Calendar 25日目の記事です.

今日は ky というライブラリを Deno から使う話です.

# [ky][] とは

<img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/ky-in-deno/ky.png" width="300" />

[ky][] は HTTP client ライブラリです. HTTP client というと [axios](https://github.com/axios/axios) が有名ですが, Deno は XHR (XMLHttpRequest) オブジェクトを持たないため axios を使うことは出来ません. XHR ベースではなく fetch ベースの手軽な HTTP Client が ky です.

Deno から ky を使うには以下のように使います.

```js
import ky from 'https://deno.land/x/ky/index.js';

const parsed = await ky.post('https://postman-echo.com/post', { json: { foo: true } }).json();

console.log(JSON.stringify(parsed, null, 2));
// => JSON output
```

(`deno eval "上のコード"` で実行できます.)

上の ky の例と同じ内容のリクエストを fetch で実行すると以下のようになります.

```js
class HTTPError extends Error {}

const response = await fetch('https://example.com', {
  method: 'POST',
  body: JSON.stringify({ foo: true }),
  headers: {
    'content-type': 'application/json'
  }
});

if (!response.ok) {
  throw new HTTPError('Fetch error:', response.statusText);
}

const parsed = await response.json();

console.log(JSON.stringify(parsed, null, 2));
// => JSON output
```

promise を2回 await している点 (HTTP ヘッダーが帰ってきた時点と, レスポンス全体が帰ってきた時点で分けて promise を待っています) や, body が JSON であることを明示的に設定している点などが煩雑ですね. このような煩雑さを回避して, JSON を送って JSON を受け取るリクエストを自然に実行できる所に ky の手軽さがあります.

## ts で使う場合の注意点

現在 ky はまだ ts に対応できていません. ts から使う場合は import した ky を一旦 any にキャストするか, 各自で適切に型付けする必要があります.

# ky の Deno サポート

ky の Deno サポートはオフィシャルなものです. [date-fns のケース](https://qiita.com/kt3k/private/4ed644e7a1359ab2552e)と同様に Deno サポートをすることにほとんど労力はかけられていません. ky は Web 標準の fetch に依存していますが, Deno の fetch も Web 標準に準拠しているため, ky がそのまま Deno で動いたのでした. そのことに気づいた ky のコラボレータの [@sholladay](https://github.com/sholladay) さんが, 公式に Deno をサポートしていることを README に[追記](https://github.com/sindresorhus/ky/pull/86)してくれたのでした.

# Recap

今日は ky を Deno から使う方法を紹介しました.

[ky]: https://github.com/sindresorhus/ky
