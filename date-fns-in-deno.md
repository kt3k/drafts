# date-fns を Deno から使う

Deno Advent Calendar 18日目の記事です.

今日は date-fns を Deno から使う話です.

# date-fns とは

https://date-fns.org/

date-fns は非常に充実した, 時刻関連の処理が得意なライブラリです. 一昔前は, 時刻といえば [moment](https://github.com/iamkun/dayjs) を使うのが常識でしたが, 最近では moment のビルドサイズが大きすぎることや, API のデザインの筋の悪さ(破壊的なメソッドが多い)ことから moment 代替のライブラリがいくつか活発に開発されています. その中でもかなり人気の高いライブラリが date-fns です.

date-fns は moment や dayjs などと違い, 時刻を表す独自の object 型を持たない点が, 他のライブラリとは一味違います.

...

# Deno から使うには

date-fns を Deno から使うには以下のようにして API を import して使います.

...

# date-fns が Deno から使える理由

date-fns が Deno で使えることは, Deno で本格的な時間の処理が必要なアプリが十分書けるということを意味するため大変重要なことです. 重要なライブラリなので, 率先して Deno へ移植されたのかと思ってしまうところですが, 実はそうではありません. date-fns は Deno に移植する作業は誰も行なっていないのです. date-fns の src 以下はブラウザの module import で直接参照して使えるようなスタイルでプログラミングされています. node 用にそれらをトランスパイルしたものが npm にアップロードされています. deno.land/x/date-fns はこのブラウザ用のエントリポイントを指しているに過ぎないのですが, Deno の module のローディングが基本的にブラウザと互換性があるため, ブラウザ向けに書かれた ES Module のソースコードを Deno が直接 import することができるのです. ここでも Deno の「ブラウザと互換なモジュールロード」というデザイン/方針の筋の良さがまさに現れた例と見ることが出来ます.

# Recap

今日は Deno で date-fns を使う方法を紹介しました.

明日は sasurau4 さんの記事です.
