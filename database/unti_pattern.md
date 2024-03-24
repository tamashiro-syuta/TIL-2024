# 目次
- [前提知識](#前提知識)
- [履歴にまつわるアンチパターン](#履歴にまつわるアンチパターン)


# 前提知識
| 制約の種類 | 説明 |
|:-----------|:------------|
| PRIMARY KEY制約 | 重複とNULLがなく、そのテーブルで一意な行であることが確定される |
| NOT NULL制約 | NULLがないことが確定 |
| UNIQUE制約 | その値がテーブルで一意であることを確定される(NULLは許容) |
| CHECK制約 | 指定した条件の値のみが保存されていることを確定される |
| DEFAULT制約 | 値が指定されない時に保存される値を決める |
| FOREIGN KEY制約 | 別テーブルの主キーと参照整合性が保たれていることを確定させる |

CHECK制約 → 「ageカラムは0以上」のような条件をつけて、それをバリデーションに値を弾くやつ

## アンチパターンを生まないためには？
**「動くものを作る時に適切に作る」**

**「わかりづらい設計やな名前はデータベースの破綻の始まり」**

具体的には、
- 命名ミスは初期段階で対処
- 今後を想定した命名

一般的に **「DBの寿命はアプリケーションの寿命より長い」** → DBの技術的負債はより早く返済する必要がある

## リファクタリングの例
| 順番 | 例 |
|:-----------| :- |
| ① 変更後の名前のカラムを新規カラムとして生成 |
| ② ①で作ったカラムは、トリガーを使って変更前のデータを同じにする | 古いカラムへのinsertやupdateをトリガーにする
| ③ サービス単位やモデル単位で、参照or更新した際に新しい方のカラムに設定し直す |
| ④ 切り替えが完了したタイミングで、古いカラムはdropする |

↑(感想) 知らなかった。長期で使っていく前提でのやり方っぽいからかな

***

<!-- TODO: ここから第2章 -->
# 履歴にまつわるアンチパターン

## どんなアンチパターン??
### ❌ 履歴データから、どの操作を行ったかがわからない
  - ex) 売上データと商品のマスタデータの単価が合わない...

### ❌ 過去の "事実" が失われる
  - ex) 商品の単価が変わった時に、過去のデータを上書きしてしまう...
  - → **商品名や価格が変わると売り上げの事実と不整合が生じる**

### ❌ 過去の "過程" が見つけにくい
  - ex) 注文のステータスを変更する...

## 脱アンチパターン
⭕️ **履歴データを残す**
  - ex) 消費税の変更を例にすると...
    - 消費税のレコードに、有効期限を設ける
    - 売り上げのテーブルに消費税の値を持たせる
  - ex) 注文のステータスでは...
    - ステータスが変更される際は、新しくレコードを追加し、最新のレコードを有効なレコードとして扱う
  - ex) 金融系のシステムでは、削除処理も「打ち消しのINSERT」として保存して、合計値を算出することがよくある

履歴データをDBにないに残さず、ログで管理する方法もある！！ (cloud watchに残して障害時はそこを見るなど)

### 😓 デメリット
- データ量が増えやすい
  - → 検索スピードが落ちる
  - → テーブルサイズが増える(金もかかる)

## 設計時の観点・ポイント
- 非常時に欲しいデータが残っているか？？
- データの変化を追えているか？？

## 感想
トレードオフになるので、(時にスピードは如実にUXに関わるので)、設計段階でドメインエキスパートと非常時に必要なデータはどれで、どこまで捨てるか、拾うかのすり合わせが大事そう。
ログ残すのは大体やってると思うから、アプリケーション側で履歴を追うような要件(ユーザー側での履歴確認など)がなかったら、残さなくても良さそう？？

***

# JOINにまつわるアンチパターン
🚨 **JOINはパフォーマンスに直結するので扱いに注意**

## JOINの種類とイメージ図
![INNER JOIN](/image/database/unti_pattern/left_outher_join.jpg)
![そのほかのJOIN](/image/database/unti_pattern/other_join.jpg)

よく使うのは、だいたい`LEFT OUTER JOIN`

<details>
<summary><h2>JOINの仕組み(難しかった😭)</h2></summary>

JOINを高速化するには、JOINする際に用いるアルゴリズムを知る必要がある
### `Nested Loop Join(NLJ)`
![NLJ](/image/database/unti_pattern/nested_loop_join.jpg)
### `Hash Join`
![hashJoin](/image/database/unti_pattern/hash_join.png)
### `Sort Merge Join`
![sortMergeJoin](/image/database/unti_pattern/sort_merge_join.png)

## 各種アルゴリズムの特徴
![algorithmFeature](/image/database/unti_pattern/algorithm_feature.jpg)
</details>

## RDBMSによるJOINの違い
- MySQL
  - `NLJ`のみサポート
- PostgreSQL
  - 上記3種類のすべてをサポートしているので、MySQLより不等号や大きな2つの表のJOINが得意(詳しくは `JOINの仕組み` の`各種アルゴリズムの特徴` を参照)

**MySQLはより、INDEXを活用した設計が求められる**

## どんなアンチパターン？？
### ❌ JOINしすぎ
- JOINするテーブルが多いと、パフォーマンスが悪くなる
- パフォーマンスが悪くなる理由は、JOIN先が増えるたび、ベン図の重なる部分が指数関数的に増えるから(計算量も指数関数的に増加)
  - ex) 3つのテーブルをJOIN → 3パターン(3C2 = 3)
  - ex) 4つのテーブルをJOIN → 6パターン(4C2 = 6)

## 脱アンチパターン
### ⭕️ 無駄にJOINしない
### ⭕️ JOINするテーブルは小さくしてからJOINする(JOIN前に絞り込みをしてデータ量を減らす)
### ⭕️ INDEXを活用する
- INDEXを活用することで、データが大きくなった際は、SortMergeJoinが効く

## 感想
JUJU、データ量多くて、一覧画面でスロークエリになってたから、JOINする箇所とかタイミング、見直した方が良いかも。

***

# INDEXにまつわるアンチパターン
## どんなアンチパターン？？
### ❌ INDEXを張っているが、INDEXが効いていない
- where句でindexが指定されていない
  - ex) age にindexを貼っているとする
    - INDEXが効く例 : `SELECT * FROM users WHERE age > 20`
    - INDEXが効かない例 : `SELECT * FROM users WHERE age * 2 > 40`
    - **※ 同じ条件でも、後者ではindexではなく、あくまでも `age * 2`という計算結果を条件にしているため、indexが効かない**
- 検索結果が多い
- 全体の件数が少ない
- カーディナリティが低いカラムに対する検索
- あいまいな検索(LIKE)
- 統計情報と実際にテーブルで乖離がある

## INDEXの役割
- INDEXは、その名の通り、索引の役割を果たす
- イメージとしては、図書館でジャンル別に並べられているから本が探しやすくなるイメージ(だいぶ効率的✨)

ex) 以下は`WHERE user_id = 4000`というクエリがあった場合、50倍も早くなるよの図
- indexを使うと1つのデータを取り出すために、4ブロックにアクセスしている = 100行取り出すには400 I/O

  <img src="../image/database/unti_pattern/btree_index.jpg" width="500">
  <img src="../image/database/unti_pattern/full_scan.jpg" width="500">
  <img src="../image/database/unti_pattern/use_index.jpg" width="500">

## INDEXが効く条件
1. **検索結果がテーブル全体の20%以下**
    - できるなら、10%未満が良い
2. **検索対象のテーブルが十分に大きい**
    - 数万 ~ 数十万レコードが目安
    - 1000行程度では、テーブルスキャンの方が効率が良いケースが多い

## INDEXを貼る際の考慮事項
- テーブルのサイズが数年後、どれくらいになるか？ を見積もる
- INDEXが複合INDEXでまとめられる or 単一のINDEXで十分ではないか？？
  - インデックスショットガンを避けるため
- 今、このINDEXを貼るべきか？？
  - 新規サービスなど、データ傾向が見えない場合は、様子見をしてから判断した方が良い

## 脱アンチパターン
### ⭕️ where句でindexが指定する
  - ex) age にindexを貼っているとする
    - INDEXが効く例 : `SELECT * FROM users WHERE age > 20`
    - INDEXが効かない例 : `SELECT * FROM users WHERE age * 2 > 40`
    - **※ 同じ条件でも、後者ではindexではなく、あくまでも `age * 2`という計算結果を条件にしているため、indexが効かない**
### ⭕️ INDEXを張るカラムは、カーディナリティが高いカラムを選ぶ
  - カーディナリティが高い = 重複が少ないデータ
  - ex) booleanだと、0 or 1しかないので、カーディナリティが高い(かぶっているデータが多い)
    - これだと、データ量が少なくならないので、indexの意味がなくなる
### ⭕️ なるべくLIKE検索を避ける
  - LIKE検索は、前方一致以外はindexが効かない
  - ex) `SELECT * FROM users WHERE name LIKE '%山田%'`
    - この場合、indexが効かないので、全件検索になる
### ⭕️ 統計情報と日データの乖離をなくす
  - 統計情報が古いと、最適なindexが選ばれない
    - indexを使うかはオプティマイザと呼ばれるものが統計情報を元に決めている
    - この統計情報は定期的にサンプリングしたデータを元に作成されている
    - オプティマイザは、あくまで「統計情報」をもとに判断するので、未来のデータは見れない。そこも含めてどうテーブルを設計するかが、腕の見せ所
  - STGではindexが使われていたが、本番では使われていないということがある場合はこのケースが多い
### ⭕️ INDEXを設定しすぎない
  - あまりにもindexを張りすぎると、データの更新が遅くなる
    - オプティマイザが適切なINDEXを判断できなくなるため
    - 「インデックスショットガン」と言われている
  - indexを張ると、データの更新時にindexも更新されるため、データの更新が遅くなる
  - なので、indexを張るかどうかは、データの更新頻度によって変わる

## Tips
### MENTORの法則
| MENTORの法則 | 説明 |
|:-----------|:------------|
| M: Measure(測定)    | スロークエリやDBのパフォーマンスなどをモニタリング |
| E: Explain(解析)    | 実行計画を見て、クエリが遅くなっている原因を追求 |
| N: Nominate(指名)   | ボトルネックの原因を特定(インデックスが未定義など) |
| T: Test(試験)       | ボトルネックの改善を実施し、処理時間を測定。改善後の全体的なパフォーマンスを確認 |
| O: Optimize(最適化) | DBパラメータの最適化を定期的に実施し、インデックスがキャッシュメモリに載るように最適化 |
| R: Rebuild(再構築)  | 統計情報やインデックスを定期的に再構築 |