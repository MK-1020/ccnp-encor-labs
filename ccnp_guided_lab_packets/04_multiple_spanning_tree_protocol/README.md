# Chapter 4 Multiple Spanning Tree Protocol

参照元: `04_Multiple_Spanning_Tree_Protocol.pdf`

この章は、Chapter 2 で学んだ STP / RSTP の基本と、Chapter 3 で学んだ保護機能を前提に、**Multiple Spanning Tree Protocol（MST）をどう設計し、どう検証するか**を整理する章として扱う。

## この章の記事構成

1. `zenn_01_mst_basics_ist_cist_msti_region.md`
2. `zenn_02_mst_configuration_and_verification.md`
3. `zenn_03_mst_tuning_ist_and_trunk_pruning.md`
4. `zenn_04_mst_region_boundary_and_pvst_simulation.md`

## この章のゴール

- MST が VLAN ごとの STP ではなく、複数 VLAN を `MSTI` に集約する方式だと説明できる
- `IST`、`CIST`、`MSTI`、`MST region` の関係を説明できる
- `MST region` は name / revision / VLAN-to-instance mapping で定義されると説明できる
- `show spanning-tree mst`、`show spanning-tree mst configuration`、`show spanning-tree mst <instance-number>` の読み方を整理できる
- `MSTI` ごとの root 設計、`IST` に残した VLAN、trunk pruning の注意点を説明できる
- `MST region boundary` と `PVST simulation check` の位置づけを説明できる

## この章の基本トポロジ

4章では、まず 3〜4台のシンプルな L2 構成で MST の基本と設定確認を行い、そのあと必要に応じて trunk や別 region 側の条件を加えていく。

### ラボA: MST基本確認用

用途:

- MST mode 有効化前後の確認
- `IST` / `MSTI` / `MST region` の基本整理
- region 一致時の verification

構成イメージ:

```text
        SW1
       /   \
     SW2---SW3
```

- trunk link を1本以上含める
- VLAN 1 / 10 / 20 / 30 を使う

### ラボB: MST設計と境界確認用

用途:

- `MSTI` ごとの root 設計
- `IST` に残した VLAN の確認
- trunk pruning
- `MST region boundary`
- PVST / RPVST 連携の整理

構成イメージ:

```text
          SW1
         /   \
       SW2---SW3
             |
            SW4
```

- 必要に応じて `SW4` を別 region 側、または PVST / RPVST 側の確認役にする
- 実装差異が出やすい箇所は、本文で `TODO:` を残して断定しない

## 各記事の役割

### `zenn_01_mst_basics_ist_cist_msti_region.md`

- MST の基本構造
- `IST`
- `CIST`
- `MSTI`
- `MST region`

### `zenn_02_mst_configuration_and_verification.md`

- MST configuration
- region 一致条件
- `show spanning-tree mst configuration`
- `show spanning-tree mst`
- `show spanning-tree mst <instance-number>`

### `zenn_03_mst_tuning_ist_and_trunk_pruning.md`

- MST tuning
- `MSTI` ごとの root 設計
- VLAN Assignment to the `IST`
- trunk pruning

### `zenn_04_mst_region_boundary_and_pvst_simulation.md`

- `MST region boundary`
- `PVST simulation check`
- `MST Region as the Root Bridge`
- `MST Region Not a Root Bridge for Any VLAN`

## この章で優先して拾うコマンド

- `show spanning-tree mst configuration`
- `show spanning-tree mst`
- `show spanning-tree mst <instance-number>`
- `show spanning-tree mst interface <interface-id>`
- `show interfaces trunk`

## 補足

- 4章は、VLAN 単位の STP から `MSTI` 単位の設計へ視点を切り替える章になる
- 実測していない `show` 出力は本文へ書かず、各記事で `TODO:` を残す
- `MST region boundary` や `PVST simulation check` は、EVE-NG / IOSvL2 で再現できる範囲と概念整理に留める範囲を分けて扱う
