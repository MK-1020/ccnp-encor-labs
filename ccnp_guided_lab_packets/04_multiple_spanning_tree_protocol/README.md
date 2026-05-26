# Chapter 4 Multiple Spanning Tree Protocol 記事設計

参照元: `04_Multiple_Spanning_Tree_Protocol.pdf`

## この章で扱うこと

Chapter 4 では、PVST / RPVST のように VLAN ごとに spanning tree を持つ考え方から一歩進み、**複数 VLAN を `MSTI` に集約して instance 単位で設計する**流れを扱う。

この章では、単に `spanning-tree mode mst` を入れて終わるのではなく、次の流れで小さな設計判断を積み上げる。

1. MST mode へ切り替える
2. `MST0` / `IST` の見え方を確認する
3. `MST region` を成立させる条件をそろえる
4. VLAN-to-instance mapping を入れる
5. `MSTI` ごとの root / cost / port-priority を調整する
6. `IST` に残す VLAN の意味を確認する
7. trunk pruning と `MSTI` topology のズレを考える
8. `MST region boundary` と `PVST simulation check` を整理する

## 共通設計シナリオ

複数 VLAN を持つ小規模な L2 冗長ネットワークを想定する。  
PVST / RPVST のように VLAN ごとに別の tree を持たせるのではなく、**関連する VLAN を `MSTI` にまとめ、instance 単位で topology を制御する**。

この章では、まず同一 `MST region` 内部で `MSTI` ごとの設計を確認し、そのあと region 外と接する境界の考え方へ進む。

## 共通トポロジと採用理由

Chapter 4 では、以下の4台構成を共通トポロジとして使う。

```text
             SW1
          Gi0/0 Gi0/1
           /       \
        Gi0/0     Gi0/0
          SW2-----SW3
        Gi0/1   Gi0/1
          \       /
        Gi0/2   Gi0/2
           \     /
             SW4
          Gi0/0 Gi0/1
```

| Link | SW-A / IF | SW-B / IF | 用途 |
|---|---|---|---|
| SW1-SW2 | SW1 Gi0/0 | SW2 Gi0/0 | MST region 内 trunk |
| SW1-SW3 | SW1 Gi0/1 | SW3 Gi0/0 | MST region 内 trunk |
| SW2-SW3 | SW2 Gi0/1 | SW3 Gi0/1 | region 内冗長 link |
| SW2-SW4 | SW2 Gi0/2 | SW4 Gi0/0 | 下流接続 / boundary 候補 |
| SW3-SW4 | SW3 Gi0/2 | SW4 Gi0/1 | 下流接続 / boundary 候補 |

採用理由:

- SW1〜SW3 は同一 `MST region` 内部の動作確認に使う
- SW4 は下流スイッチとして使い、後続記事では `boundary` 確認用の相手にも使う
- 冗長 link を持たせることで、root / role / state / tuning / pruning / boundary の見え方を確認できる

## VLAN / MSTI 設計

- VLAN 10 は `MSTI 1`
- VLAN 20 / 30 は `MSTI 2`
- VLAN 40 はあえて `MSTI 1` / `MSTI 2` に割り当てず、`IST` / `MST0` に残す確認用として使う

## この章で確認する show コマンド

- `show spanning-tree summary`
- `show spanning-tree mst`
- `show spanning-tree mst configuration`
- `show spanning-tree mst <instance-number>`
- `show spanning-tree mst interface <interface-id>`
- `show interfaces trunk`

## マスター回収表

| OCG論点 | 扱う記事 | 状態 | 扱い方 | 備考 |
|---|---|---|---|---|
| Multiple Spanning Tree Protocol | 第18回〜第20回 | 回収予定 | 本編で検証 | 基本構造と mode 切替 |
| MSTI | 第18回 / 第22回 / 第23回 | 回収予定 | 本編で検証 | mapping と tuning |
| IST | 第18回 / 第20回 / 第26回 | 回収予定 | 本編で検証 | `MST0` の見え方 |
| CIST | 第18回 / 第29回 | 回収予定 | 概念整理のみ | 詳細挙動は概念中心 |
| MST region | 第18回 / 第21回 | 回収予定 | 本編で検証 | 成立条件の確認 |
| MST Configuration | 第21回 / 第22回 | 回収予定 | 本編で検証 | name / revision / mapping |
| MST Verification | 第20回〜第23回 | 回収予定 | 本編で検証 | `show spanning-tree mst` 系 |
| MST Tuning | 第23回〜第25回 | 回収予定 | 本編で検証 | priority / cost / port-priority |
| MSTI priority / root bridge | 第23回 | 回収予定 | 本編で検証 | `MSTI` ごとの root |
| MST interface cost | 第24回 | 回収予定 | 本編で検証 | 実測結果は TODO 予定 |
| MST port priority | 第25回 | 回収予定 | 本編で検証 | 実測結果は TODO 予定 |
| VLAN Assignment to the IST | 第26回 | 回収予定 | 本編で検証 | VLAN 40 を利用 |
| Trunk Link Pruning | 第27回 | 回収予定 | 本編で検証 | VLAN allowed と `MSTI` のズレ |
| MST Region Boundary | 第28回 | 回収予定 | 本編で検証 | revision 不一致の例 |
| PVST simulation check | 第29回 | 回収予定 | 概念整理のみ | 必要なら任意確認 |
| MST Region as the Root Bridge | 第29回 | 回収予定 | 概念整理のみ | 境界設計の整理 |
| MST Region Not a Root Bridge for Any VLAN | 第29回 | 回収予定 | 概念整理のみ | 境界設計の整理 |

状態は次を使う。

- 未着手
- 回収予定
- 検証中
- 回収済み
- 対象外

## 記事分割案

| 回 | ファイル | タイトル | 扱う設計判断 | 主な設定 | 主な確認コマンド |
|---|---|---|---|---|---|
| 第18回 | `zenn_01_mst_chapter4_design_scenario.md` | Cisco MSTの使い方：Chapter 4共通設計シナリオ | 章全体の地図を作る | なし | なし |
| 第19回 | `zenn_02_mst_enable_mode.md` | Cisco MST設定：spanning-tree mode mstでMSTを有効化する | mode を MST へ切り替える | `spanning-tree mode mst` | `show spanning-tree summary` |
| 第20回 | `zenn_03_mst0_ist_show.md` | Cisco MSTの見方：MST0 / ISTをshow spanning-tree mstで確認する | `MST0` / `IST` を読む | なし | `show spanning-tree mst` / `show spanning-tree mst configuration` |
| 第21回 | `zenn_04_mst_region_three_conditions.md` | Cisco MST設定：MST regionを成立させる3条件 | name / revision をそろえる | `spanning-tree mst configuration` | `show spanning-tree mst configuration` |
| 第22回 | `zenn_05_mst_vlan_to_instance_mapping.md` | Cisco MST設定：VLAN-to-instance mappingを確認する | VLAN を `MSTI` にまとめる | `instance 1 vlan ...` | `show spanning-tree mst configuration` / `mst 1` / `mst 2` |
| 第23回 | `zenn_06_msti_root_bridge_design.md` | Cisco MST設定：MSTIごとにroot bridgeを設計する | `MSTI` ごとに root を変える | `spanning-tree mst <n> priority ...` | `show spanning-tree mst 1` / `mst 2` |
| 第24回 | `zenn_07_mst_interface_cost_tuning.md` | Cisco MST設定：spanning-tree mst costで経路を調整する | interface cost を変える | `spanning-tree mst 1 cost ...` | `show spanning-tree mst 1` / `mst interface ...` |
| 第25回 | `zenn_08_mst_port_priority_tuning.md` | Cisco MST設定：spanning-tree mst port-priorityで同一コスト時の選択を調整する | port-priority を変える | `spanning-tree mst 1 port-priority ...` | `show spanning-tree mst 1` / `mst interface ...` |
| 第26回 | `zenn_09_vlan_assignment_to_ist.md` | Cisco MSTの使い方：VLANをIST / MST0に残すとどう見えるか | VLAN を `IST` に残す | `vlan 40` | `show spanning-tree mst configuration` / `mst 0` |
| 第27回 | `zenn_10_mst_trunk_pruning.md` | Cisco MSTとtrunk pruning：VLAN allowedとMSTI topologyのズレを確認する | pruning のズレを読む | `switchport trunk allowed vlan ...` | `show interfaces trunk` / `show spanning-tree mst 2` |
| 第28回 | `zenn_11_mst_region_boundary_revision.md` | Cisco MST設定：revision不一致でMST region boundaryを確認する | revision 不一致を作る | `revision 2` | `show spanning-tree mst configuration` / `mst interface ...` |
| 第29回 | `zenn_12_pvst_simulation_check.md` | Cisco MSTとPVST連携：PVST simulation checkの考え方 | 境界の概念を整理する | なし | 任意確認のみ |

## 補足

- 実測していない `show` / `ping` / `traceroute` / Wireshark 結果は本文に書かず、各記事で `TODO:` を残す
- `MST region boundary` や `PVST simulation check` は、EVE-NG / IOSvL2 で再現できる範囲と概念整理に留める範囲を分けて扱う
- config 配布フォルダは新しい 12 分割構成に合わせて更新済み
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs
