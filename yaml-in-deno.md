# Deno で yaml を扱う

Deno (ディーノ) <img src="https://raw.githubusercontent.com/kt3k/drafts/master/assets/deno.png" width="20"> Advent Calendar 25日目の記事です.

今日は Deno で YAML を扱う話です.

# YAML モジュール

Deno は標準ライブラリで YAML モジュールを持っていて, parse, stringify などの API を使って YAML のパースやシリアライズを行うことが出来ます.

```ts
import { parse, stringify } from "https://deno.land/std/encoding/yaml.ts";

const data = parse(`
foo: bar
baz:
  - qux
  - quux
`);
console.log(data);
// => { foo: "bar", baz: [ "qux", "quux" ] }

const yaml = stringify({ foo: "bar", baz: ["qux", "quux"] });
console.log(yaml);
// =>
// foo: bar
// baz:
//   - qux
//   - quux
```

`parse` は引数に YAML 文字列を受け取って, パースしたオブジェクトを返します.

`stringify` は逆にオブジェクトを受け取って, YAML 文字列を返します.

YAML は JSON とは違って, 1ファイルに複数のドキュメントを持つことが出来ます. 1ファイルに複数の YAML ドキュメントを記述するには各ドキュメントを `---` という行で区切ります.

```yaml
id: 1
name: Alice
---
id: 2
name: Bob
---
id: 3
name: Eve
```

`parse` 関数はこのような複数ドキュメントを持った YAML 文字列を parse することはできません. 複数ドキュメントの YAML をパースしたい場合は parseAll というメソッドを使うことが出来ます.

```ts
import { parseAll } from "https://deno.land/std/encoding/yaml.ts";

const data = parseAll(`
---
id: 1
name: Alice
---
id: 2
name: Bob
---
id: 3
name: Eve
`);
console.log(data);
// => [ { id: 1, name: "Alice" }, { id: 2, name: "Bob" }, { id: 3, name: "Eve" } ]
```

# YAML モジュールの歴史

YAML モジュールは [js-yaml](https://github.com/nodeca/js-yaml) という npm で一番よく使われている YAML ライブラリがベースになっています. YAML モジュールの[最初の移植作業](https://github.com/denoland/deno/pull/3361)はフランスの Deno meetup である [Paris Deno](https://www.meetup.com/ja-JP/Paris-Deno/) のオーガナイザの Lilian Saget-Lethias ( [@bios21](https://github.com/bios21) ) さんによって行われました.

# まとめ

今日は Deno で yaml を扱う方法を紹介しました.
