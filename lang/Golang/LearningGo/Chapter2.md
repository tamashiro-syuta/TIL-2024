# 基本型と宣言
## 基本型
- Goは **「明快さ」と「可読性」に重きをおく言語** なので暗黙的な型変換はしない
  - 型が異なる場合は、明示的に変換する必要がある
  - こうした方が型変換のルールを覚える必要がなく、わかりやすくなるという考え方
- Goでは基本的にキャメルケースが使われる

### 整数リテラル
- 通常は10進数で表記するが、先頭に以下の文字をそれぞれ加えることで、他の進数で表記することもできる
  - `0b` : 2進数 ( **b** inary )
  - `0o` : 8進数 ( **o** ctal )
  - `0x` : 16進数 ( he **x** adecimal )
- 例
  - `42` : 10進数
  - `0b101010` : 2進数
  - `0o52` : 8進数
  - `0x2a` : 16進数
- Goでは数字を読みやすくするために、_(アンスコ)を任意の箇所にかける
  - 例
    - `1_000_000` : 1,000,000
    - `0b1010_1010` : 170
    - `0o52_52` : 2706
    - `0x2a_2a` : 10794
  - ただし、先頭と最後に書くことと、アンスコを連続で書くことはNG

### 浮動小数リテラル
- 指数を使った表記もできる
  - ex: `6.03e23` = 6.03 * 10^23 (10の23乗)
- 整数リテラルと同じく、アンスコで区切れる

### runeリテラル
- runeリテラル = Unicodeコードポイントを表す整数リテラル(int32型のエイリアス) = 文字を表す
- ex:
  - 改行文字 = `\n`
  - タブ文字 = `\t`
  - シングルクォート = `\'`
  - バッククォート = `\"`
  - バックスラッシュ = `\`

### 整数型
- 整数型の値の範囲
  - `int8` : -128 ~ 127
  - `int16` : -32768 ~ 32767
  - `int32` : -2147483648 ~ 2147483647
  - `int64` : -9223372036854775808 ~ 9223372036854775807
  - `uint8` : 0 ~ 255
  - `uint16` : 0 ~ 65535
  - `uint32` : 0 ~ 4294967295
  - `uint64` : 0 ~ 18446744073709551615
- 特別な整数型
  - `byte` : `uint8`のエイリアス
    - `byte`の方が一般的
  - `int` : CPUによって64ビット(int64)または32ビット(int32)の符号付き整数
  - `unit` : intの符号なし版 = 0以上
  - `rune` : Unicodeコードポイントを表す整数リテラル(int32型のエイリアス) = 文字を表す
    - **int32型のエイリアスだが、文字を表す際はrune型と明示した方がコードの意図を明確にできるので良い**
  - `uintptr` : ポインタを格納するのに使う整数型
- どの整数型を選ぶかの基準
  - 特定の大きさと特定の符号の整数バイナリのファイルあるいはネットワークプロトコルを処理しているならば、対応する整数型を使う
  - 全ての整数型について機能するライブラリ関数を書く場合は2つの関数を書く
      1. 引数と戻り値(返り値)がint64のもの
      2. unit64のもの
  - それ以外は、`int`型を使う
  - **理由なくint型以外を使うのは「早すぎる最適化」**

### 浮動小数点数型
- 浮動小数点数型は、`float32`と`float64`の2つ
- それぞれの範囲は以下
  - `float32`
    - 最大値: 3.4028234663852885981170418348451692544e+38
    - 最小値: 1.401298464324817070923729583289916131280261941876515771757068283496126897789e-45
  - `float64`
    - 最大値: 1.797693134862315708145274237317043567981e+308
    - 最小値: 4.940656458412465441765687928682213723651e-324
- 通常は`float64`を使う
  - 他の言語とかも大体`float64`を使っている(IEEE754ってやつに準拠しているらしい)
- そもそも浮動小数点数を使うかも検討する必要あり
  - Goにおいて浮動小数点数が表す範囲はとても広く、精度も高いが、計算誤差が発生することがある
  - なので、使うなら多少の誤差は許容できる場合のみ使う方が良い(グラフィックや物理計算とかに限定されるかもね by著者)
- 浮動小数点数型はあくまでの近似値なので、等価性の比較は避けるべき
  - 等しいことを確かめる場合は、2つの値の差が一定の範囲内に収まるかを判定するのが良い
  - `一定の精度` をどのくらいにするのかは、どの型(どの精度)かによる


## 変数の宣言
- 宣言方法は複数あるが、大切なのは **「開発者の意図が伝わるような宣言方法を選ぶ」** こと
  - ex) ゼロ値を表現したい → `var x int`
  - ex) intではなく、byte型として扱いたい → `var x byte = 2`
- 複数の変数をワンライナーで宣言できるが、これは複数の値を返す関数からの戻り値を代入する時だけ使った方が良い
  - こうすることで、関数からの戻り値が複数であることを明示的に表現できる(体と思う)
  - `var result, err = someFunction()` ← `someFunction()`が複数の値を返す関数であると表現している
- `:=`を用いて宣言できるのは、関数の中だけでパッケージのトップレベルでは使えないので注意
  - トップレベルでは`var`や`const`を使う

## 定数
- **Goにおける定数 = リテラルに名前を付与するもの**
  - **変数がイミュータブルであることを宣言する方法はない**
  - Rubyでいう`freeze`みたいなものはない
- 定数には「型なし」と「型付き」の2種類がある
  - 型なし : `const x = 10`
    - この時、`x`は型なしなので、以下のように異なる型に代入できる
      ```go
      const x = 10
      var y int = x
      var z float64 = x
      ```
    - 型なしのメリットは「柔軟さ」
  - 型付き : `const x int = 10`
    - この時、`x`は`int`型なので、以下のように異なる型に代入できない
      ```go
      const x int = 10
      var y int = x
      var z float64 = x // コンパイルエラー
      ```
    - 型付きのメリットは「安全性」
- 定数は未使用でもコンパイルエラーにならない
  - コンパイル時に未使用の定数は、削除されバイナリファイルには含まれなくなる