String
Collection

arrray
0 に添え字
startIndex()の方が良い
arraySlice

Int で添字にアクセスして取得する配列の要素の型

let string = "hoshdoih"
最初の要素にアクセスしたい場合は必ず startIndex というプロパティの型
添字アクセス専用の型

let index -

Stirng は Collection なのに
添字で変更できない

Collection に count プロパティが生えている
https://gyazo.com/644a386153b83b82da4a23f163b12245

Collection であっても扱いが少し違う

String が扱いにくい
そのためには Unicode について理解する必要がある

let pa1 = "..."
let pa2 = "..."

https://gyazo.com/fe71cfaab3d5cea7fa1041c03a63bda4

内部表現的には異なるのに count は同じ(見た目通りの count)

拡張書紀素クラスタ
// extended Graphenme Cluster
Swift の Character 型に相当
ユーザーにとっての 1 文字
2 つの scala 値を削除していてっもユーザーにとっての一文字(拡張書紀素クラスタ)を削除するべき

let crlf = "\r\n"
crlf.count // 1

仕様通り。

https://gyazo.com/cbe9b02dd9a80edca04d25399e725fa7

U+0d
一つの改行記号であることは確定
スライドのリンク参照

JP くにこーど is 何
国コードを 2 つをが組み合わさると一つの国を表す文字になるという仕様がある

N 番目の文字を知るためには、その前にある文字を知る必要がある。N 番目の文字を取得するための計算量は O(N)

添字アクセスの計算量は O(1)でないといけない

数字を添え字にしてアクセスする方向で実装すると Collection が要請する性能を満たすことができない

String.Index
メモリ上の始点からオフセット分だけ取るだけで良いから

## String と要素の置き換えについて

添字アクセスで set できるかできないかの違いは
MutableCollection

Sequence -> Collection -> MutableCollection

添字アクセスによろ set が O(1)が実現できないから
入れ替えた文字の以降の文字のバイト列を移動する必要がある
添字アクセスによる set の計算量は O(N)なる
よって MutableCollection に conform させない

Twitter 内部表現のバイト幅をみて
Swift の String の高性能さがわかる

String.Index それ専用の型
O(1)になる理由
内部的には文字列を表す utf8 bit 列
bit 列の byte のオフセット
オフセット分だけ機械的に進めたところを取得するだけで良いので計算量は O(1)になる

UnicodeScholor を知る方法
scholor 値をググったら Unicode
UnicodeScholor 型のプロパティ

String.unicodeScholors
count で内部表現の数がわかる

String.UnicodeScalarView
