# CCNP ENCOR 試験対策 第29回: Cisco MSTとPVST連携：PVST simulation checkの考え方

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **`MST region` の外側と接したときに、PVST / RPVST 側との整合性をどう考えるか** を整理します。  
このテーマは EVE-NG / IOSvL2 で再現性に差が出やすいため、概念整理を主に扱います。

## 今回の確認ポイント

- `MSTI` が PVST 側へそのまま個別に出るわけではないこと
- `MST region` は外部から 1 つのまとまりのように見えること
- `PVST simulation check` が何のためにあるのか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| PVST simulation check | 高 | MST と PVST / RPVST 境界での整合性の考え方 | 文章で説明 | 概念整理のみ | EVE-NG で再現性に差が出やすいため |
| MST Region as the Root Bridge | 中 | MST region 側が root の設計 | 文章で説明 | 概念整理のみ | 境界での見え方理解に必要なため |
| MST Region Not a Root Bridge for Any VLAN | 中 | MST region 側が root ではない設計 | 文章で説明 | 概念整理のみ | 比較で理解しやすいため |
| CIST | 中 | `IST` / `CIST` を境界でどう考えるか | 文章で説明 | 概念整理のみ | `PVST simulation check` の前提になるため |

## 前回とのつながり

前回は、revision 不一致を使って `MST region boundary` を確認しました。  
今回はその続きとして、boundary の相手が PVST / RPVST 側だった場合の考え方を整理します。

## 検証ステータス

| 項目 | 状態 |
|---|---|
| EVE-NG構築 | 対象外または任意 |
| 設定投入 | 対象外または任意 |
| show確認 | TODO |
| 疎通確認 | 対象外 |
| Wireshark確認 | 任意 |
| 結果反映 | TODO |

## 検証環境

| 項目 | 内容 |
|---|---|
| 使用イメージ | IOSvL2 |
| 主題 | 概念整理中心 |
| 注意 | 実測できない部分は断定しない |

## トポロジ

この回では、前回までの `boundary` の考え方を引き継ぎつつ、**相手が PVST / RPVST 側だったらどう考えるか** を整理します。  
実際のトポロジは、Chapter 4 の共通トポロジに「region 外の相手」を重ねて考えます。

## 初期状態

この回では、実測可能な環境があれば boundary 状態を利用して確認してよいですが、主眼は概念整理です。  
そのため、必須の初期状態確認は置かず、必要に応じて前回の `boundary` 記事を参照します。

確認コマンド:

```cisco
show spanning-tree mst
show spanning-tree mst 0
```

期待する結果:

- `IST` / `CIST` を主語に boundary を考える意識が持てること
- 実測できない部分を断定しない構成になっていること

## 実際の結果

```markdown
TODO: 実測できる環境がある場合のみ、`show spanning-tree mst` や `show spanning-tree mst 0` の出力を貼る。
```

## 設定

この回の主題は概念整理なので、必須の設定は置きません。  
必要なら、前回の `boundary` 確認構成を使って補助確認します。

## 設定反映・期待結果の確認

この回では、実測よりも **何を主語に考えるべきか** を整理することが目的です。  
show や Wireshark は、実測できる場合だけ補助確認として使います。

確認の観点:

- `MSTI` がそのまま PVST 側へ個別に出るわけではないこと
- `MST region` は外部から 1 つのまとまりのように見えること
- 境界では `IST` / `CIST` の情報をもとに整合性を取ること
- その整合性が崩れると、`PVST simulation check` により root inconsistent になる可能性があること

## 概念整理

### MSTI がそのまま外へ出るわけではない

`MSTI` は `MST region` 内部で VLAN を集約して扱うための単位です。  
したがって、PVST 側へ出るときに **各 `MSTI` がそのまま個別に外へ出ていく** と考えるのは適切ではありません。

### MST region は外部から 1 つのまとまりのように見える

`MST region` は内部では複数の instance を持ちますが、外部と接するときは 1 つのまとまりとして扱う意識が大切です。  
このとき、境界では `IST` / `CIST` を主語に整合性を考えます。

### PVST simulation check

PVST / RPVST 側との境界では、`IST` / `CIST` の情報をもとに整合性を取ります。  
その整合性が崩れると、`PVST simulation check` により boundary port が root inconsistent になる可能性があります。

### MST region が root の場合と root ではない場合

- `MST Region as the Root Bridge`
  - MST region 側が root として振る舞う設計
- `MST Region Not a Root Bridge for Any VLAN`
  - MST region 側が root にならず、外側を root として受ける設計

この違いによって、boundary でどちら側の情報を強く意識するかが変わります。  
ただし、ここは実装差異や再現条件の影響も受けやすいため、詳細挙動は未確認なら断定しません。

PVST / RPVST 側との接続を再現できる場合は、境界リンクで BPDU をキャプチャし、MSTI がそのまま PVST 側へ出ていくわけではないことを補助的に確認します。  
ただし、IOSvL2 環境では再現性に差が出る可能性があるため、この記事では詳細挙動は概念整理を主とし、キャプチャは任意確認として扱います。

## 試験で問われそうな形

- `PVST simulation check` は MST と PVST / RPVST の境界で整合性を取る考え方
- `MSTI` がそのまま外へ個別に出るわけではない
- 境界では `IST` / `CIST` を主語に考える
- root inconsistent の可能性を理解しておく

## 読み終えたあとに説明できること

- `PVST simulation check` が何のためにあるのか
- なぜ `MSTI` をそのまま外側の主語にしないのか
- `MST region` が root になる場合とならない場合の違い

## まとめ

この回では、`PVST simulation check` を中心に、`MST region` の外側と接するときの考え方を整理しました。  
Chapter 4 全体としては、MST の基本構造、region 設定、tuning、boundary までを小さな設計判断ごとに回収した形になります。

## 関連ファイル

- この回で使う PVST simulation check 概念整理用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/12_pvst_simulation_check
