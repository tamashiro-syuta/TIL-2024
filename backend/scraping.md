# スクレイピングに関するTips

## アンチパターン、個人的な失敗談
### コンテキスト
- スクレイピングでページング機能のあるサイトからデータを取得することが目的
- 激おもなサイトだった(ページ遷移に約30秒かかる)
- importしたデータはcsvファイルに出力

### 単一のCSVファイルに出力する
- 失敗した時に、切り戻しがしにくい
  - 失敗した箇所のid調べて、続きからできるようにコードなり何なりを修正して、、、、になる

## ベストプラクティス、やって良かったこと
### (時短目的だけど)、あらかじめタブをたくさん開いておいた
- これだけで、タブ数倍のスピードでデータを取得できる
- ただし、ブラウザのメモリなどには注意が必要
- Dos攻撃にならないように、適切な間隔を空けることも大事
  - 社内システムだったのである程度合意は取れていたが、外部サイトの場合は注意が必要

### (当たり前だけど)処理をメソッドごとにカプセル化
- スクレイピングはちょっとずつ実行して値とれているか確認のが、通常の実装より多いイメージ。
- そのため、カプセル化して、隠蔽しているとコードを編集していく時に編集しやすくて楽だった