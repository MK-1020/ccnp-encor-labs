# CCNP ENCOR 試験対策: Spanning Tree Protocol 2章のまとめとして全体像を押さえる

# STP・RSTP・root bridge・port role・topology changeをまとめて理解する

## はじめに

CCNP ENCOR Chapter 2 **Spanning Tree Protocol** では、単に `show spanning-tree` の見方を覚えるだけでなく、**冗長リンクがあるL2ネットワークで、どうやってループを防ぐか** という考え方を押さえることが大事です。

ここまでの記事では、

- STP が root bridge を基準に木構造を作ること
- root port / designated port / blocking or alternate の役割
- `show spanning-tree` の読み方
- RSTP の基本的な見え方

を確認しました。

今回は、それらを踏まえて、Chapter 2 全体を1本の記事で振り返ります。

## この記事で押さえたいこと

- なぜSTPが必要なのか
- root bridge を中心にどう木構造を作るのか
- root port / designated port / blocking or alternate の違い
- STP と RSTP の違い
- topology change をどこで見るのか

## なぜSTPが必要なのか

L2ネットワークでは、冗長リンクを張ると安心感がありますが、そのままではループの危険があります。

特に、

- ブロードキャスト
- unknown unicast flooding

のようなトラフィックは、TTLのような仕組みを持たないため、ループすると止まりません。

STP は、その冗長性を残したまま、**どこかのリンクを論理的に止めて木構造にする** ことでループを防ぎます。

## root bridge を中心に考える

STP の理解でいちばん大事なのは、**まず root bridge が1台選ばれる** という点です。

そこから各スイッチが、

- root へ向かう最もよいポート
- 下流へBPDUを流すポート
- 冗長なので止めるポート

を決めていきます。

つまり、STP は「全部のポートを平等に扱う仕組み」ではなく、**root bridge を基準に役割を決める仕組み** と考えると分かりやすいです。

## port role の見方

`show spanning-tree` を読むときは、まず次の役割を押さえると見やすくなります。

- **root port**
  - non-root switch が root bridge へ向かうためのポート
- **designated port**
  - セグメント代表として forwarding するポート
- **blocking or alternate port**
  - 冗長なので通常は転送しないポート

この役割分担によって、物理的にはループしていても、論理的には木構造になります。

## `show spanning-tree` では何を見るか

この章でいちばんよく使うのは `show spanning-tree vlan 1` です。

ここでは主に次を見ます。

- `This bridge is the root`
  - root bridge かどうか
- `Root ID`
  - root bridge の情報
- `Bridge ID`
  - 自分自身の情報
- `Role` / `Sts`
  - 各ポートの役割と状態

この4点を追うだけでも、かなり読みやすくなります。

## STP と RSTP の違い

STP の基本形は 802.1D ですが、CCNP の学習では RSTP もあわせて押さえる必要があります。

まず大づかみに言うと、

- **STP**
  - 基本形
- **RSTP**
  - より速く収束しやすいよう改善された形

です。

そのため、試験対策では細かい差分を全部暗記する前に、**RSTP は STP の改善版** と理解しておくと入りやすいです。

## topology change も見ておく

STP は安定して動いていても、リンクのアップダウンなどがあると topology change が発生します。

これは `show spanning-tree vlan 1 detail` のようなコマンドで確認できます。

ここでは、

- 変化回数
- 最後にいつ変化したか
- どのポート由来か

を見ることで、ネットワークが安定しているかの手がかりになります。

## ここまでを一気に振り返ると

Chapter 2 では、次の流れで理解するとつかみやすいです。

- 冗長リンクがあるとL2ループの危険がある
- STP は root bridge を基準に木構造を作る
- root port / designated port / blocking or alternate が決まる
- `show spanning-tree` でその役割が見える
- RSTP はその仕組みをより速く扱いやすくしたもの
- detail 出力では topology change も追える

## まとめ

Spanning Tree Protocol の章は、最初は用語が多く感じられますが、実際には

- なぜ止める必要があるのか
- 誰を基準にするのか
- どのポートが forwarding でどのポートが止まるのか

の3つが見えてくると、かなり理解しやすくなります。

特に試験対策としては、

- root bridge
- root port / designated port / blocking or alternate
- `show spanning-tree` の読み方
- STP と RSTP の違い

を押さえておくと、Chapter 2 全体がかなりつかみやすくなるはずです。

## 関連記事

- STP Fundamentals
- STP Verification
- RSTP Basics
- Extra Check
