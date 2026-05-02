# Chapter 1 Packet Forwarding

参照元: `01_Packet_Forwarding.pdf`

この章では、`スイッチやルーターが何を見て、どの表を使い、どう転送するのか` を記事ベースで確認する。

## この章の記事構成

1. `zenn_01_l2_forwarding.md`
2. `zenn_02_l3_forwarding.md`
3. `zenn_03_cef_fib_adjacency.md`
4. `zenn_04_extra_check_packet_forwarding.md`
5. `zenn_packet_forwarding_architecture_overview.md`

## この章のゴール

- L2ではMACアドレスを使って転送することを説明できる
- L3では宛先IPアドレスで次ホップを決め、MACはホップごとに変わることを説明できる
- CEFで `RIB -> FIB -> Adjacency` の関係を説明できる

## Extra Checkの位置づけ

本編記事で拾いきれなかった細かい項目を、`zenn_04_extra_check_packet_forwarding.md` でまとめて確認する。

- できるだけ実際にコマンドを打って確認する
- 1本の細かい記事として読めるようにする
- 複数の小テーマを1記事にまとめてもよい
