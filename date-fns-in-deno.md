# date-fns を Deno から使う

Deno Advent Calendar 18日目の記事です.

今日は date-fns を Deno から使う話です.

# [date-fns][] とは

[date-fns][] はとても機能が豊富な時刻を扱うためのライブラリです. 一昔前は, 時刻といえば [moment](https://momentjs.com/) を使うのが一般的でしたが, 最近では moment のバンドルサイズが大きすぎることや, API のデザインの筋の悪さ(破壊的なメソッドが多い)から moment 代替のライブラリがいくつか活発に開発されています. その中でも人気が高いライブラリの一つが date-fns です.

date-fns は moment や [dayjs](https://github.com/iamkun/dayjs) などと違い, 時刻を表すための独自クラスを持たない点が特徴です. 独自クラスを使わずに JavaScript の builtin 型である Date 型を, API の入出力として使います. つまり date-fns は, ライブラリ内で状態を保持することはなく, 全てが純粋な関数としての API として表現されています. この点から date-fns は自身を `日付のための lodash` と表現しています.

具体的な date-fns の使い方を見てみましょう.

```
import { compareAsc, format } from 'date-fns'

const label = format(new Date(2014, 1, 11), 'yyyy-MM-dd', {})
console.log(label)
//=> '2014-02-11'

const dates = [
  new Date(1995, 6, 2),
  new Date(1987, 1, 11),
  new Date(1989, 6, 10)
]
dates.sort(compareAsc)
console.log(dates)
// => [ 1987-02-10T15:00:00.000Z, 1989-07-09T15:00:00.000Z, 1995-07-01T15:00:00.000Z ]
```

format は Date とフォーマット指定子を受け取って, フォーマット結果を返す関数です. 1引数目に builtin の Date 型を受け取っています.

compareAsc は Date を2つ受け取って, 大小関係を比較します, 上の例では sort の比較関数として利用しています.

この例からも分かるように, date-fns の API は純粋な関数で, Date を入出力として使い, ライブライ自体が状態を持つ事がありません.

# Deno から使うには

date-fns を Deno から使うには以下のようにして API を import して使います.

```
import { compareAsc, format } from 'https://deno.land/x/date_fns/index.js'

const label = format(new Date(2014, 1, 11), 'yyyy-MM-dd')
console.log(label)

const dates = [
  new Date(1995, 6, 2),
  new Date(1987, 1, 11),
  new Date(1989, 6, 10)
]
dates.sort(compareAsc)
console.log(dates)
```

(なお, 上の例を `deno eval "コピペ"` のように実行する事も出来ます.)

import 文以外は上の例と同じです. from に指定するモジュールが Deno の場合は `https://deno.land/x/date_fns/index.js` になります. Deno は URL から直接モジュールを import 出来るため, このような指定の仕方になります.

上の例を実行すると分かりますが, この書き方の場合, date-fns 全体を一旦ローカルに全てダウンロードすることになります. これは `https://deno.land/x/date_fns/index.js` というスクリプトが date-fns の全ての API を内部で import しているからです. 次のように import 文を書き換えると, date-fns の必要部分だけを import する事が出来ます.

```
import compareAsc from 'https://deno.land/x/date_fns/compareAsc/index.js'
import format from 'https://deno.land/x/date_fns/format/index.js'

...
```

# date-fns が Deno から使える理由

date-fns が Deno で使えることは, Deno で本格的な時間の処理が必要なアプリが十分書けるということを意味するため大変重要なことです.

では, date-fns を Deno に移植するために Deno チームはどのくらいの時間をかけたのでしょうか? 実は全く何もしていないのです. data-fns は [src 以下のソースコード](https://github.com/date-fns/date-fns/tree/master/src)をブラウザで直接 import できるようなスタイルでプログラミングしています (npm にはそれを transpile したコードを publish しています). Deno の import は可能な限りブラウザの import と同じルールで path を解決するという方針で作られているため, ブラウザ用の date-fns が Deno でそのまま使えたのでした.

この事実に気づいた @justjavac さんが, Deno のレジストリに date-fns を追加する [PR](https://github.com/denoland/registry/pull/154) を投げた事で, https://deno.land/x/date_fns/ 以下の名前空間で date-fns を利用する事が出来るようになっています.

# まとめ

今日は Deno で date-fns を使う方法を紹介しました.

明日は @sasurau4 さんの記事です.

[date-fns]: https://date-fns.org/
