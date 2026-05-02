# Chapter 2 Spanning Tree Protocol

参照元: `02_Spanning_Tree_Protocol.pdf`

この章では、`冗長リンクがあるL2ネットワークで、どうやってループを防ぎながら通信を成立させるか` を記事ベースで確認する。

## この章の記事構成

1. `zenn_01_stp_fundamentals.md`
2. `zenn_02_stp_verification.md`
3. `zenn_03_rstp_basics.md`
4. `zenn_04_extra_check_spanning_tree_protocol.md`
5. `zenn_spanning_tree_protocol_architecture_overview.md`

## この章のゴール

- STPがL2ループを防ぐための仕組みだと説明できる
- root bridge / root port / designated port / alternate or blocking port を見分けられる
- `show spanning-tree` の出力から、どのスイッチがrootで、どのポートがどんな役割なのか読める
- 802.1D と RSTP の違いを大づかみに説明できる

## この章の進め方

- `zenn_01` で STP の基本用語と木構造を確認する
- `zenn_02` で `show spanning-tree` の出力を実際に読む
- `zenn_03` で RSTP の見え方を押さえる
- `zenn_04` で topology change や trunk 上の見え方を補う
- 最後に `zenn_spanning_tree_protocol_architecture_overview.md` で章全体を振り返る
