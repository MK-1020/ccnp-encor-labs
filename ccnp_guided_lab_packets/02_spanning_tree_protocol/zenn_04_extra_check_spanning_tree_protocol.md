# CCNP ENCOR 試験対策 第4回: Spanning Tree Protocol の細かい確認を Extra Check で補う

# EVE-NGで確認するSTP Extra Check：topology change と detail 出力の見方

## はじめに

今回は、Spanning Tree Protocol の第4回として、本編で主役にしにくかった細かい確認をまとめて見ていきます。

ここまでの記事では、STP の基本動作、`show spanning-tree` の見方、RSTP の基本を確認しました。

一方で、STP にはそれ以外にも

- topology change の見え方
- `show spanning-tree ... detail` の読み方
- trunk 上で VLAN ごとにどう見えるか

のように、単体で記事1本にしにくいものの、後で効いてくる確認ポイントがあります。

そこで今回は、細かい確認を1本の記事にまとめる形で Extra Check を作りました。

## 今回確認したいこと

- topology change count をどこで見るか
- 最後の変化がどのポート由来か読めること
- `show spanning-tree interface ... detail` の用途
- trunk 設定と STP の見え方を合わせて見ること

## 検証トポロジ

```markdown
TODO: ここにEVE-NGのトポロジスクリーンショットを貼る
```

## 今回使う設定

今回は前回までの STP 構成をそのまま流用します。

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

## 検証1: topology change の見え方を確認する

SW1で入力:

```cisco
show spanning-tree vlan 1 detail
```

SW1の実行結果:

```markdown
TODO: ここに `show spanning-tree vlan 1 detail` の結果を貼る
```

### `show spanning-tree vlan 1 detail` の見方

- `Number of topology changes`
  - 変化回数
- `last change occurred`
  - 直近の変化からどれくらい経ったか
- `from GigabitEthernet...`
  - どのインターフェース由来の変化か

ここでは、**STP が安定しているか、最近よく揺れていないかを見る入口** として detail 出力を使います。

## 検証2: interface detail でポート単位に見る

SW2で入力:

```cisco
show spanning-tree interface gi0/0 detail
```

SW2の実行結果:

```markdown
TODO: ここに `show spanning-tree interface gi0/0 detail` の結果を貼る
```

このコマンドでは、ポート単位で

- role / state
- path cost
- port priority
- BPDU の送受信

などを絞って見られます。

出力が長い `show spanning-tree vlan ... detail` より、**特定の1ポートに注目したいときに使いやすい** コマンドです。

## 検証3: trunk 側も合わせて見る

SW2で入力:

```cisco
show interfaces trunk
show spanning-tree interface gi0/0
```

SW2の実行結果:

```markdown
TODO: ここに `show interfaces trunk` と `show spanning-tree interface gi0/0` の結果を貼る
```

ここで意識したいのは、STPだけ見ても分からないことがある、という点です。

たとえば VLAN が trunk に通っていなければ、STP の出力以前に、**その VLAN 自体がそのリンクに乗っていない** 可能性があります。

そのため、STP確認では

- `show spanning-tree`
- `show interfaces trunk`

を合わせて見る癖があると、切り分けしやすくなります。

## パケットキャプチャしてみるなら

Extra Check では、トランクリンク上で STP のやり取りを見ると、detail 出力とのつながりが分かりやすいです。

- キャプチャ場所
  - トランクリンク
- 発生させること
  - 可能なら PC 側ポートを shutdown / no shutdown して topology change を起こす
- 表示フィルタ
  - `stp`

キャプチャ結果:

```markdown
TODO: ここにtopology change前後のキャプチャ画像または確認結果を貼る
```

ここで見たいのは、**detail 出力で見た topology change と、実際の制御通信の変化がつながること** です。

`show spanning-tree ... detail` の数字だけ見るより、イベント前後をキャプチャと合わせると理解しやすくなります。

## 分かったこと

- topology change は `show spanning-tree ... detail` で確認できる
- detail 出力では、最近の変化やポート由来の情報が読める
- STP の確認では trunk 設定も合わせて見ると理解しやすい

## まとめ

STP の本編では root bridge やポート役割が中心になりますが、運用やトラブルシュートに近い目線では detail 出力の見方もかなり大事です。

こうした細かい確認を別記事で補っておくと、本編の理解がかなり安定しやすくなります。

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/02_spanning_tree_protocol/configs/extra_check_spanning_tree_protocol
