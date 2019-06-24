# K7-Architecture-4th-Assignment

番号：sse02-19 日付：19/06/05

## 第四回のアーキテクチャ評価演習

以下、アーキテクチャ評価の内容

- 評価対象となるアーキテクチャモデル
    - 評価前
    - 評価後

* 品質特性ユーティリティーツリー
* 評価対象となる品質特性シナリオごとの評価結果 (センシティビティポイント，トレードオフ，リスク)

## Architecture Model

#### 評価前

[![Image from Gyazo](https://i.gyazo.com/db77ccb35c2f79c6fb0a065206a6403c.png)](https://gyazo.com/db77ccb35c2f79c6fb0a065206a6403c)

## Utility Tree

[![Image from Gyazo](https://i.gyazo.com/e2282ee0a06fedfb031193c1a9e5942c.png)](https://gyazo.com/e2282ee0a06fedfb031193c1a9e5942c)

## 品質属性シナリオ評価

### S1. 同時に10万ユーザーが接続されても、1分以内に結果を返すこと (性能)

[![Image from Gyazo](https://i.gyazo.com/8dddf8c82f362c7b331334ecc355f8c4.png)](https://gyazo.com/8dddf8c82f362c7b331334ecc355f8c4)

### S2. 新規ユーザーのMEMEの追加が必要となったときに少ない修正コストでシステムに追加可能 (変更容易性)

[![Image from Gyazo](https://i.gyazo.com/ac69beae555dfad97f74b4f874161bb7.png)](https://gyazo.com/ac69beae555dfad97f74b4f874161bb7)

### S3. ほかのデータ処理方式が必要となったときに少ない修正コストでシステムに追加可能 (変更容易性)

[![Image from Gyazo](https://i.gyazo.com/d342610c1bde17c4b7a982eab5aa73fb.png)](https://gyazo.com/d342610c1bde17c4b7a982eab5aa73fb)

## Architecture Model

#### 評価後

[![Image from Gyazo](https://i.gyazo.com/0bb44ed3897ee181ea8a177306960763.png)](https://gyazo.com/0bb44ed3897ee181ea8a177306960763)

#### 変更点①

- T5, R4を受けて、以下修正。
- デバイス側をpub/subで処理すると、MEME Processorが、新たなデータ処理方式となった場合（つまり、新MEME Processorに入れ替えた場合）に、topicを変更する等の修正が想定される。デバイス側のMEME Collectorの設定に変更が入り、デバイスが大量にあると、変更容易性の面で、アーキテクチャ的に課題と判断。
- また、Smart Phone側のアプリを、REST通信とすることでSmart Phone側のアプリの変更容易性が、pub/sub形式よりも良いという判断。
- 以上より、変更後のアーキテクチャでは、REST通信を受けるAPI Gatewayを採用するアーキテクチャとする。

#### 変更点②

- R3の指摘を受け、MEME Logsの永続化に、NoSQL(cassandraを想定)を利用するように変更。
- T3,R1の指摘を受け、データベースの負荷、多重性を考慮して、分散配置が可能なデータベース(cassandraを想定)を利用するように変更。なお、ログを蓄積するという処理であるので、蓄積時の処理は、非同期で書き出す処理に変更。

以上です。

