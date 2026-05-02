# CCNP ENCOR 試験対策: RSTPの基本とSTPとの違いを押さえる

# EVE-NGで確認するRSTP Basics：protocol rstp と port role / state の見え方

## はじめに

- 前回までは、STP の基本動作と `show spanning-tree` の読み方を確認しました。
- 今回は RSTP の基本を押さえて、802.1D ベースの STP とどう違うのかを見ていきます。

CCNP の学習では、STP と RSTP を完全に別物として覚えるより、**同じループ防止の仕組みを、より速く動けるようにしたもの** と考えると入りやすいです。

## 今回確認したいこと

- `show spanning-tree` で protocol が `rstp` と見えること
- role / state の見え方が分かること
- 802.1D より RSTP のほうが収束を意識した仕組みになっていること

## 検証トポロジ

```markdown
TODO: ここにEVE-NGのトポロジスクリーンショットを貼る
```

## 今回使う設定

今回は前回までと同じ三角形トポロジを使い、RSTP ベースで動いていることを確認します。

SW1で入力:

```cisco
enable
configure terminal
hostname SW1
spanning-tree mode rapid-pvst
interface range gi0/0 - 2
 no shutdown
end
write memory
```

SW2で入力:

```cisco
enable
configure terminal
hostname SW2
spanning-tree mode rapid-pvst
interface range gi0/0 - 2
 no shutdown
end
write memory
```

SW3で入力:

```cisco
enable
configure terminal
hostname SW3
spanning-tree mode rapid-pvst
interface range gi0/0 - 2
 no shutdown
end
write memory
```

## 実行コマンド

SW1で入力:

```cisco
show spanning-tree summary
show spanning-tree vlan 1
```

SW2で入力:

```cisco
show spanning-tree summary
show spanning-tree vlan 1
```

SW3で入力:

```cisco
show spanning-tree summary
show spanning-tree vlan 1
```

SW1の実行結果:

```markdown
TODO: ここにSW1の `show spanning-tree summary` と `show spanning-tree vlan 1` の結果を貼る
```

SW2の実行結果:

```markdown
TODO: ここにSW2の `show spanning-tree summary` と `show spanning-tree vlan 1` の結果を貼る
```

SW3の実行結果:

```markdown
TODO: ここにSW3の `show spanning-tree summary` と `show spanning-tree vlan 1` の結果を貼る
```

## `protocol rstp` の見方

`show spanning-tree vlan 1` の冒頭で、`Spanning tree enabled protocol rstp` のように表示されていれば、現在のSTP動作が RSTP ベースであることが分かります。

ここではまず、**自分のスイッチが今どのモードで動いているか** を確認します。

## STP と RSTP の違いをざっくり押さえる

この章でまず押さえたいのは、次の違いです。

- 802.1D
  - 古いSTPの基本形
- RSTP
  - より速く収束しやすいように改善された形

細かい内部動作まで一気に覚えるより、まずは **RSTP は STP の改善版** と考えると十分です。

## role / state の見え方

RSTP では、`Role` と `Sts` の表示がまとまって見やすく感じられることがあります。

- `Role`
  - `Root` `Desg` `Altn` など
- `Sts`
  - `FWD` `BLK` など

このあたりは前回の `show spanning-tree` の読み方とつながっています。

つまり、読み方そのものが大きく変わるというより、**同じ出力の中で RSTP として動いていることを確認する** 感覚です。

## パケットキャプチャしてみるなら

スイッチ間リンクで STP 系のフレームを見ておくと、`protocol rstp` の表示と実際の制御通信がつながりやすくなります。

- キャプチャ場所
  - SW1-SW2 間などのスイッチ間リンク
- 表示フィルタ
  - `stp`

キャプチャ結果:

```markdown
TODO: ここにRSTP系BPDUのキャプチャ画像または確認結果を貼る
```

ここで見たいのは、**CLIでは RSTP と見えていても、実際には STP 系の制御フレームが継続して流れている** ことです。

こうして見ると、RSTP も STP の延長線上にある仕組みだとつかみやすくなります。

## 分かったこと

- `protocol rstp` の表示から、現在のSTP動作が RSTP ベースだと確認できる
- RSTP は STP の改善版として押さえると理解しやすい
- root / designated / alternate などの役割の読み方は引き続き重要

## まとめ

RSTP は、STP の考え方を捨てるものではなく、**同じループ防止をより速く扱いやすくした仕組み** と考えるとつかみやすいです。

次は Extra Check として、topology change や `show spanning-tree ... detail` の見方を補っていきます。

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/02_spanning_tree_protocol/configs/rstp_basics
