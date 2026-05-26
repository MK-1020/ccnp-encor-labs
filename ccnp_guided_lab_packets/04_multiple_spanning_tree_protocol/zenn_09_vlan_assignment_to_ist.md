# CCNP ENCOR 試験対策 第26回: Cisco MSTの使い方：VLANをIST / MST0に残すとどう見えるか

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **明示的に `MSTI` へ割り当てない VLAN を `IST` / `MST0` に残す** ところを扱います。  
すべての VLAN を `MSTI 1` や `MSTI 2` に入れるとは限らない、という設計上の扱いを確認します。

## 今回の確認ポイント

- VLAN 40 を作るが、`MSTI 1` / `MSTI 2` には入れないこと
- なぜ `IST` / `MST0` に残す VLAN を確認するのか
- `show spanning-tree mst configuration` と `show spanning-tree mst 0` のどこを見ればよいか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| VLAN Assignment to the IST | 高 | 明示的に割り当てない VLAN が `IST` に残ること | `show spanning-tree mst configuration` / `mst 0` | 本編で検証 | OCG の重要論点のため |
| IST | 高 | `IST` / `MST0` が実際に意味を持つこと | `show spanning-tree mst 0` | 本編で検証 | `MST0` を「余り」と誤解しないため |

## 前回とのつながり

前回までは、`MSTI` ごとの tuning を見てきました。  
今回は、逆に **どの `MSTI` にも明示的に割り当てない VLAN** をどう読むかに注目します。

## 検証ステータス

| 項目 | 状態 |
|---|---|
| EVE-NG構築 | TODO |
| 設定投入 | TODO |
| show確認 | TODO |
| 疎通確認 | 対象外 |
| Wireshark確認 | 対象外 |
| 結果反映 | TODO |

## 検証環境

| 項目 | 内容 |
|---|---|
| 使用イメージ | IOSvL2 |
| 使用ノード | SW1〜SW4 |
| 対象VLAN | 40 |
| 主な確認コマンド | `show spanning-tree mst configuration` / `show spanning-tree mst 0` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、VLAN 40 を追加しつつ、`MSTI 1` / `MSTI 2` には割り当てず、`MST0` / `IST` に残して確認します。

## 初期状態

前回までに VLAN 10 / 20 / 30 の mapping と `MSTI` tuning まで入っている前提で進めます。  
ここでは、VLAN 40 をまだ作っていない状態、または作っていても `MSTI` に割り当てていない状態を確認します。

確認コマンド:

```cisco
show spanning-tree mst configuration
show spanning-tree mst 0
```

期待する結果:

- いま `MSTI 1` / `MSTI 2` に載っている VLAN が確認できること
- これから VLAN 40 を `IST` 側に残す前提が分かること

## 実際の結果

```markdown
TODO: 設定前の `show spanning-tree mst configuration` と `show spanning-tree mst 0` の出力を貼る。
```

## 設定

VLAN 40 を作成しますが、`spanning-tree mst configuration` では `instance 1` や `instance 2` へ割り当てません。

全スイッチで入力:

```cisco
configure terminal
vlan 40
 name LEGACY_40
end
write memory
```

## 設定反映・期待結果の確認

ここでは、VLAN 40 を作成したうえで **`MSTI 1` / `MSTI 2` に載せていないため、`IST` / `MST0` 側で扱われること** を確認します。

確認コマンド:

```cisco
show spanning-tree mst configuration
show spanning-tree mst 0
```

期待する結果:

- VLAN 40 を作成しても、`instance 1` / `instance 2` に入れていないこと
- 明示的に割り当てていない VLAN は `MST0` / `IST` 側に残ること
- `IST` は「余り」ではなく、意味を持つ instance であること

## 実際の結果

```markdown
TODO: 設定後の `show spanning-tree mst configuration` と `show spanning-tree mst 0` の出力を貼る。
```

## 出力の見方

- `show spanning-tree mst configuration` では、VLAN 40 が `instance 1` / `instance 2` に入っていないことを確認します
- `show spanning-tree mst 0` では、`MST0` / `IST` 側を読む意識を持ちます

## 概念整理

### IST は「余り」ではない

明示的に別の `MSTI` へ割り当てていない VLAN は、default で `MST0` / `IST` 側に属します。  
つまり `IST` は「使っていない instance」ではなく、MST の土台として実際に意味を持つ instance です。

## 試験で問われそうな形

- VLAN Assignment to the IST
- 明示的に割り当てない VLAN は `MST0` / `IST`
- `show spanning-tree mst 0` で `IST` 側を確認する

## 読み終えたあとに説明できること

- VLAN を `IST` / `MST0` に残す意味
- VLAN 40 をどこに割り当てていないのか
- `show spanning-tree mst configuration` と `mst 0` の役割

## まとめ

この回では、VLAN 40 を `MSTI` に載せず、`IST` / `MST0` に残す考え方を整理しました。  
次回は、trunk pruning と `MSTI` topology のズレをどう読むかを確認します。

## 関連ファイル

- この回で使った VLAN を IST / MST0 に残す確認用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/09_vlan_assignment_to_ist
