# GitHub Actions で Deno を使う

Deno Advent Calendar 14日目の記事です.

今日は GitHub Actions で Deno を使う方法を紹介します. GitHub Actions で Deno を使うには [denolib/setup-deno](https://github.com/denolib/setup-deno) という Action を使います.

# 使い方

使い方はとても簡単で, action の step で `denolib/setup-deno@v1.1.0` を指定すれば, ホスト環境に `deno` コマンドがインストールされます.

```yml
jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - uses: denolib/setup-deno@v1.1.0
        with:
          deno-version: 0.x
      - run: deno run https://deno.land/std/examples/welcome.ts
```

(実際にはさらに `on` プロパティでトリガーをセットする必要があります.)

複数の Deno バージョンでマトリックステストをしたい場合は以下のようにして実現できます.

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        deno: [0.24.0, 0.23.0]
    name: Deno ${{ matrix.deno }} sample
    steps:
      - uses: actions/checkout@master
      - name: Setup Deno
        uses: denolib/setup-deno@v1.1.0
        with:
          deno-version: ${{ matrix.deno }}
      - run: deno run https://deno.land/std/examples/welcome.ts
```

以下のレポジトリにデモを用意しています. 参考にしてみてください.

https://github.com/kt3k/deno-github-action-demo

# Recap

今日は setup-deno を使って GitHub Actions 環境に Deno をインストールして使う方法を紹介しました.
