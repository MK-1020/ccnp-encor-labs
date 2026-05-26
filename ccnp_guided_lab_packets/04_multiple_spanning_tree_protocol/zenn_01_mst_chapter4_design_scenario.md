# CCNP ENCOR 試験対策 第18回: Cisco MSTの使い方：Chapter 4共通設計シナリオ

## この章で扱うこと

Chapter 4 では、PVST / RPVST のように VLAN ごとに spanning tree を持つ考え方から一歩進み、**複数 VLAN を `MSTI` に集約して instance 単位で topology を制御する**流れを扱います。

この章では、MST をただ有効化するだけではなく、

- `MST0` / `IST` をどう読むか
- `MST region` をどう成立させるか
- `MSTI` ごとに root / cost / port-priority をどう調整するか
- `IST` に残す VLAN をどう考えるか
- `boundary` や `PVST simulation check` をどう整理するか

を、小さな設計判断ごとに分けて確認していきます。

## 今回の確認ポイント

- この章全体でどんなネットワークを想定するか
- なぜこの共通トポロジを使うのか
- VLAN と `MSTI` をどう割り当てる予定か
- 後続の小記事がどの順で積み上がるか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| Multiple Spanning Tree Protocol | 高 | Chapter 4 全体で何を扱うか | 文章で説明 | 本編で検証 | 章全体の入口になるため |
| MSTI | 高 | VLAN を instance 単位で扱う前提 | 文章で説明 | 本編で検証 | 後続記事の主語になるため |
| IST | 高 | `MST0` / `IST` を後続でどう扱うか | 文章で説明 | 本編で検証 | Chapter 4 の土台になるため |
| CIST | 中 | region 外との接続で意識すること | 文章で説明 | 概念整理のみ | 詳細は後続記事で扱うため |
| MST region | 高 | 同じ設計を共有する単位 | 文章で説明 | 本編で検証 | 後続記事の前提になるため |

## 前章までとのつながり

前章までは、STP / RSTP の基本動作と保護機能を学んできました。  
Chapter 4 ではそこから一歩進み、**VLAN ごとの spanning tree** から **`MSTI` ごとの spanning tree** へ視点を切り替えます。

## 共通設計シナリオ

複数 VLAN を持つ小規模な L2 冗長ネットワークを想定します。  
PVST / RPVST のように VLAN ごとに tree を持たせるのではなく、**関連する VLAN を `MSTI` にまとめ、instance 単位で topology を制御する**ことを目標にします。

この章では、まず同一 `MST region` 内部の動作を固め、そのあと region 外と接する境界の見え方へ進みます。

## 共通トポロジと採用理由

Chapter 4 では、以下の4台構成を共通トポロジとして使います。

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
| SW1-SW2 | SW1 Gi0/0 | SW2 Gi0/0 | `MST region` 内 trunk |
| SW1-SW3 | SW1 Gi0/1 | SW3 Gi0/0 | `MST region` 内 trunk |
| SW2-SW3 | SW2 Gi0/1 | SW3 Gi0/1 | 冗長 link |
| SW2-SW4 | SW2 Gi0/2 | SW4 Gi0/0 | 下流接続 / boundary 候補 |
| SW3-SW4 | SW3 Gi0/2 | SW4 Gi0/1 | 下流接続 / boundary 候補 |

採用理由:

- SW1〜SW3 は同一 `MST region` 内部の動作確認に使う
- SW4 は下流スイッチとして使い、後続記事で `boundary` や外部接続を確認する相手にも使う
- 冗長 link を持たせることで root / role / state / tuning / pruning / boundary の見え方を確認できる

![[スクリーンショット 2026-05-26 210406.png]]

## VLAN / MSTI 設計

- VLAN 10 は `MSTI 1`
- VLAN 20 / 30 は `MSTI 2`
- VLAN 40 は、あえて `MSTI 1` / `MSTI 2` に割り当てず、`IST` / `MST0` に残す確認用として使います

## この章で確認する主な show コマンド

- `show spanning-tree summary`
- `show spanning-tree mst`
- `show spanning-tree mst configuration`
- `show spanning-tree mst <instance-number>`
- `show spanning-tree mst interface <interface-id>`
- `show interfaces trunk`

## 記事分割案

| 回 | タイトル | 扱う設計判断 | 主な設定 | 主な確認コマンド |
|---|---|---|---|---|
| 第19回 | Cisco MST設定：spanning-tree mode mstでMSTを有効化する | mode を MST へ切り替える | `spanning-tree mode mst` | `show spanning-tree summary` |
| 第20回 | Cisco MSTの見方：MST0 / ISTをshow spanning-tree mstで確認する | `MST0` / `IST` を読む | なし | `show spanning-tree mst` / `show spanning-tree mst configuration` |
| 第21回 | Cisco MST設定：MST regionを成立させる3条件 | name / revision をそろえる | `spanning-tree mst configuration` | `show spanning-tree mst configuration` |
| 第22回 | Cisco MST設定：VLAN-to-instance mappingを確認する | VLAN を `MSTI` へ割り当てる | `instance 1 vlan ...` | `show spanning-tree mst configuration` / `mst 1` / `mst 2` |
| 第23回 | Cisco MST設定：MSTIごとにroot bridgeを設計する | `MSTI` ごとの root を分ける | `spanning-tree mst <n> priority ...` | `show spanning-tree mst 1` / `mst 2` |
| 第24回 | Cisco MST設定：spanning-tree mst costで経路を調整する | interface cost で経路を変える | `spanning-tree mst 1 cost ...` | `show spanning-tree mst 1` / `mst interface ...` |
| 第25回 | Cisco MST設定：spanning-tree mst port-priorityで同一コスト時の選択を調整する | port-priority を変える | `spanning-tree mst 1 port-priority ...` | `show spanning-tree mst 1` / `mst interface ...` |
| 第26回 | Cisco MSTの使い方：VLANをIST / MST0に残すとどう見えるか | VLAN を `IST` に残す | `vlan 40` | `show spanning-tree mst configuration` / `mst 0` |
| 第27回 | Cisco MSTとtrunk pruning：VLAN allowedとMSTI topologyのズレを確認する | pruning のズレを確認する | `switchport trunk allowed vlan ...` | `show interfaces trunk` / `show spanning-tree mst 2` |
| 第28回 | Cisco MST設定：revision不一致でMST region boundaryを確認する | revision 不一致を作る | `revision 2` | `show spanning-tree mst configuration` / `mst interface ...` |
| 第29回 | Cisco MSTとPVST連携：PVST simulation checkの考え方 | 境界の概念を整理する | なし | 任意確認のみ |

## 概念整理

### なぜMSTを使うのか

PVST / RPVST は VLAN ごとに spanning tree topology を持てるため分かりやすい一方で、VLAN 数が増えると STP instance の数も増えます。  
MST は、**同じ転送方針で扱える複数 VLAN を `MSTI` にまとめる**ことで、管理する topology 数を減らしつつ、instance 単位で制御するための方式です。

### IST / CIST / MSTI / MST region の位置づけ

- `MSTI`
  複数 VLAN をまとめて載せる spanning tree instance
- `IST`
  `MST0`。同一 `MST region` 内で必ず存在する基本 instance
- `CIST`
  `MST region` の内部だけでなく、region 外との接続も含めて意識する共通木
- `MST region`
  同じ name / revision / VLAN-to-instance mapping を持つスイッチ群

## 試験で問われそうな形

- MST は VLAN ごとの STP ではなく、複数 VLAN を `MSTI` に集約する
- `IST` は `MST0`
- `MST region` は name / revision / VLAN-to-instance mapping で定義される
- `boundary` や `PVST simulation check` は region 外との接続で意識する

## 読み終えたあとに説明できること

- Chapter 4 全体で何を確認するのか
- なぜこの共通トポロジを使うのか
- VLAN 10 / 20 / 30 / 40 をどう使い分けるのか
- 後続の小記事がどの設計判断を扱うのか

## まとめ

この回では、Chapter 4 全体の共通設計シナリオと共通トポロジを確認しました。  
次回は、まず `spanning-tree mode mst` で STP mode を MST へ切り替えるところから始めます。

## 関連ファイル

- この章で使う共通設計シナリオ / 共通トポロジ用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/01_mst_chapter4_design_scenario
