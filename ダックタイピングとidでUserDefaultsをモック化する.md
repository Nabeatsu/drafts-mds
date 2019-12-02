# ダックタイピングと id で UserDefaults をモック化する

https://speakerdeck.com/417_72ki/datukutaipingutoiddeuserdefaultswomotukuhua-suru?slide=6

## テストと UserDefaults

- protocol を噛ませて mock 化する
- suiteName を使う(これ知らなかった。)

protocol
冗長
書き換えの労力
suiteName を使う方法
永続領域をテストでいじるのに抵抗がある

## ダックタイピング

動的型付け言語で使われる
型付けに関する考え方
オブジェクト自体がメソッドやプロパティを持つかで判定(not 型)

一般的な静的型付け言語
interface による抽象化

ダックタイピングによる動的型付け
走れと命令する。走った場合は犬
実行してみないとわからない

Objective-C の型システム
実行時に型のチェックをしないので方が一致しないオブジェクトを代入してもコンパイルエラーにならない(警告のみ)
メソッドを呼ぶ際に問題があるとクラッシュする

実行時に型のチェックをしない
方のチェックをせずにメソッドを実行する
格納されているオブジェクトがそのメソッドを実行できれば OK
この考え方で Delegate が設計されている

UITableviewDelegate
@protocol がついてるメソッドは実装を強制されない
respondsToSelector
メソッドが実装されているかチェックする
あったら呼ぶみたいな感じで実装しているんじゃないか

id 型

- https://gyazo.com/0cb28b0e0a916b03a66613dbb1011d33

ポインタを保持しているので\*をつけないのが特徴
型安全性を捨てて柔軟性にステータスを振っている

## id 型

## MockUserDefaults
