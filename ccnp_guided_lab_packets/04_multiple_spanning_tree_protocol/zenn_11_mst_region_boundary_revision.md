# CCNP ENCOR 試験対策 第28回: Cisco MST設定：revision不一致でMST region boundaryを確認する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **同じ `MST region` を共有していない相手と接続したときの見え方** を確認します。  
具体的には、SW4 だけ revision を変えて、SW2-SW4 / SW3-SW4 を `boundary` 候補として読みます。

## 今回の確認ポイント

- `MST region` 情報が一致しないとき、同じ内部設計を共有できないこと
- revision 不一致をどの設定で作るか
- `show spanning-tree mst configuration` と `show spanning-tree mst interface` のどこを見るか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| MST Region Boundary | 高 | region 不一致による boundary の考え方 | `show spanning-tree mst configuration` / `mst interface ...` | 本編で検証 | 4章後半の主要論点のため |
| Common MST Misconfigurations | 中 | revision 不一致の例 | `show spanning-tree mst configuration` | 本編で検証 | 実運用のミスと結び付きやすいため |

## 前回とのつながり

前回は、同一 `MST region` 内の trunk pruning を確認しました。  
今回はそこから外に出て、同じ `MST region` として扱えない相手と接したときの見え方を確認します。

## 検証ステータス

| 項目 | 状態 |
|---|---|
| EVE-NG構築 | TODO |
| 設定投入 | TODO |
| show確認 | TODO |
| 疎通確認 | 対象外 |
| Wireshark確認 | 任意 |
| 結果反映 | TODO |

## 検証環境

| 項目 | 内容 |
|---|---|
| 使用イメージ | IOSvL2 |
| 使用ノード | SW1〜SW4 |
| boundary 候補 | SW2 Gi0/2 --- SW4 Gi0/0 / SW3 Gi0/2 --- SW4 Gi0/1 |
| 主な確認コマンド | `show spanning-tree mst configuration` / `show spanning-tree mst interface gi0/2` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、SW1〜SW3 を同一 `MST region` のまま維持し、SW4 だけ revision を変えて `boundary` 候補にします。

boundary 候補:

- SW2 Gi0/2 --- SW4 Gi0/0
- SW3 Gi0/2 --- SW4 Gi0/1

Wireshark で補助確認する場合は、まず SW2-SW4 を観測点にします。

## 初期状態

まずは、SW1〜SW4 が同じ `MST region` としてそろっている状態を確認します。  
そのあと、SW4 だけ revision を変えます。

確認コマンド:

```cisco
show spanning-tree mst configuration
show spanning-tree mst
show spanning-tree mst interface gi0/2
```

期待する結果:

- まだ同じ `MST region` として読めること
- これから revision 不一致を作る前の基準状態が分かること

## 実際の結果

```markdown
TODO: 変更前の `show spanning-tree mst configuration`、`show spanning-tree mst`、`show spanning-tree mst interface gi0/2` の出力を貼る。
```

## 設定

ここでは例として、SW4 側だけ revision を `2` に変えます。

SW4で入力:

```cisco
configure terminal
spanning-tree mst configuration
 revision 2
end
write memory
```

## 設定反映・期待結果の確認

ここでは、`show` を使って **同じ `MST region` の内部設計をそのまま共有できない状態になったか** を確認します。

確認コマンド:

```cisco
show spanning-tree mst configuration
show spanning-tree mst
show spanning-tree mst interface gi0/2
```

期待する結果:

- SW4 だけ revision が変わっていること
- SW2-SW4 / SW3-SW4 のリンクを `boundary` 候補として意識する必要があること
- 同じ `MST region` の内部設計を共有している前提が崩れたと説明できること

## 実際の結果

```markdown
TODO: 変更後の `show spanning-tree mst configuration`、`show spanning-tree mst`、`show spanning-tree mst interface gi0/2` の出力を貼る。
```

## 出力の見方

- `show spanning-tree mst configuration` では revision の差分をまず見ます
- `show spanning-tree mst interface gi0/2` では、boundary 候補として見たい link の interface 観点を確認します
- どのノード / interface を見ているかを正確に意識するのが大事です

## 概念整理

### boundary をどこで意識するか

`MST region boundary` は、単に name / revision / mapping が違うときだけを指すのではなく、**同じ `MST region` の内部設計を共有していない相手と接する場所** と考えると整理しやすいです。

## 試験で問われそうな形

- revision 不一致は `MST region boundary` の例になる
- 同じ `MST region` の内部設計を共有できない link を boundary 候補として読む
- `show spanning-tree mst configuration` と `show spanning-tree mst interface ...` の役割

## 読み終えたあとに説明できること

- revision 不一致で何が変わるのか
- どの link を boundary 候補として見るのか
- どの `show` で region 不一致を確認するのか

## まとめ

この回では、revision 不一致を使って `MST region boundary` を確認する考え方を整理しました。  
次回は、PVST / RPVST との境界で意識する `PVST simulation check` の考え方をまとめます。

## 関連ファイル

- この回で使った MST region boundary 確認用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/11_mst_region_boundary_revision
