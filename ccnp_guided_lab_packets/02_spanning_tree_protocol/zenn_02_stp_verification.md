# CCNP ENCOR 試験対策: show spanning-treeの出力からSTPの役割を読み取る

# EVE-NGで確認するSTP Verification：Root ID・Bridge ID・Role・Stateの見方

## はじめに

- 前回は、STP が root bridge を基準に木構造を作ることを確認しました。
- 今回は `show spanning-tree` の出力を読んで、どのスイッチが root で、どのポートがどんな役割なのかを確認します。

STP は概念だけ覚えると曖昧になりやすいですが、`show spanning-tree` を読めるようになるとかなり理解しやすくなります。

## 今回確認したいこと

- `Root ID` と `Bridge ID` の違い
- root bridge の見つけ方
- root port / designated port / alternate or blocking port の見分け方
- root path cost の見方

## 検証トポロジ

```markdown
TODO: ここにEVE-NGのトポロジスクリーンショットを貼る
```

## 今回使う設定

今回は前回と同じ三角形トポロジを使います。

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

## 今回使うコマンド

SW1で入力:

```cisco
show spanning-tree vlan 1
```

SW2で入力:

```cisco
show spanning-tree vlan 1
```

SW3で入力:

```cisco
show spanning-tree vlan 1
```

SW1の実行結果:

```markdown
TODO: ここにSW1の `show spanning-tree vlan 1` の結果を貼る
```

SW2の実行結果:

```markdown
TODO: ここにSW2の `show spanning-tree vlan 1` の結果を貼る
```

SW3の実行結果:

```markdown
TODO: ここにSW3の `show spanning-tree vlan 1` の結果を貼る
```

## `show spanning-tree vlan 1` の見方

### `Root ID`

`Root ID` は、その VLAN で root bridge として認識しているスイッチの情報です。

non-root switch では、ここに **自分ではない別のスイッチ** が表示されます。

### `Bridge ID`

`Bridge ID` は、自分自身のスイッチ情報です。

つまり、

- `Root ID`
  - 根元にいる root bridge
- `Bridge ID`
  - 今このコマンドを打っている自分自身

と考えると分かりやすいです。

### `This bridge is the root`

この行が出ていれば、そのスイッチが root bridge です。

まずはこの行の有無を見ると、root bridge を手早く見つけられます。

### `Interface Role Sts`

この表では、各ポートの役割と状態を見ます。

- `Role`
  - `Root` `Desg` `Altn` など
- `Sts`
  - `FWD` `BLK` など

ここを見ると、**どのポートが上流向きで、どのポートが下流向きで、どのポートが止められているか** が分かります。

## 検証1: root bridge を特定する

root bridge では、`This bridge is the root` が表示されます。

また、root bridge のポートは、基本的に designated / forwarding 側として見えるはずです。

ここで大事なのは、**root bridge はネットワーク全体で1台だけ** という点です。

## 検証2: non-root switch の root port を確認する

non-root switch では、root bridge へ最も良い経路で向かうポートが root port になります。

`Role` 列に `Root` と表示されているポートを見つけると、そのスイッチがどちら向きに root を見ているか分かります。

## 検証3: 冗長リンク上の alternate or blocking を確認する

三角形トポロジでは、冗長リンク上のどちらか一方が転送しない側になります。

そのため、

- `Altn`
- `BLK`

のような表示を探すと、**STP がどこでループを止めているか** を把握できます。

## パケットキャプチャしてみるなら

root bridge と non-root switch の間のリンクでBPDUをキャプチャすると、`show spanning-tree` で見ていた情報とつなげやすいです。

- キャプチャ場所
  - root bridge と SW2 の間、または root bridge と SW3 の間
- 表示フィルタ
  - `stp`

キャプチャ結果:

```markdown
TODO: ここにBPDUのキャプチャ画像または確認結果を貼る
```

ここで見たいのは、**root bridge を基準にした情報が実際にBPDUとして流れている** ことです。

`show spanning-tree` の `Root ID` と、実際に流れるBPDUをつなげて見ると、CLIの出力がかなり実感しやすくなります。

## 分かったこと

- `Root ID` は root bridge、`Bridge ID` は自分自身を指す
- `This bridge is the root` を見れば root bridge をすぐ見つけられる
- `Role` と `Sts` を見ると、各ポートの役割が分かる

## まとめ

`show spanning-tree` は最初は行数が多く見えますが、実際には

- root bridge は誰か
- root port はどれか
- どこが forwarding でどこが blocking か

の3点を見るだけでもかなり読みやすくなります。

次は RSTP の見え方を押さえて、802.1D とどう違うのかを確認します。

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/02_spanning_tree_protocol/configs/stp_verification
