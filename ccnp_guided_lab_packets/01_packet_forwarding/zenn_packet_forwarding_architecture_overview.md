# CAM・TCAM・Process Switching・CEF・dCEF・SDM Templateをまとめて理解する

## はじめに

CCNP ENCOR Chapter 1 **Packet Forwarding** では、L2 Forwarding、L3 Forwarding、CEFの基本動作だけでなく、ネットワーク機器の内部で何が起きているかというアーキテクチャの話も出てきます。

ただ、この後半部分は少し抽象的で、`show` コマンドで確認しやすい話と、言葉の意味を押さえながら読んだほうが入りやすい話が混ざっています。

第1回から第4回までの記事では、

- L2ではMACアドレスとVLANを使って転送すること
- L3ではルーティングテーブルとARPを使って次ホップへ送ること
- CEFでは `RIB -> FIB -> Adjacency` という役割分担があること

を確認しました。

今回は、第1回から第4回までで見てきた内容を踏まえて、**Packet Forwarding 1章のまとめ** として次の用語を1本の記事で見ていきます。

- **CAM**
- **TCAM**
- **process switching**
- **CEF**
- **software CEF**
- **hardware CEF**
- **centralized forwarding**
- **distributed forwarding**
- **dCEF**
- **SDM Template**

ラボ記事というより、**Chapter 1 全体の仕上げとして、試験に出てくる言葉と実機で見えるコマンドをつなぐまとめ記事** というイメージで読むとちょうどよい内容です。

## この記事で押さえたいこと

- **CAM** と **TCAM** は何が違うのか
- **process switching** と **CEF** は何が違うのか
- **software CEF** と **hardware CEF** はどう違うのか
- **centralized forwarding** と **distributed forwarding** はどう違うのか
- **dCEF** は何をしているのか
- **SDM Template** は何のためにあるのか

## まず全体像

Packet Forwardingの章後半は、ざっくり次の2つに分けて考えるとかなりつかみやすくなります。

1. パケットをどう転送判断するか
2. その判断をどこで、どのハードウェア資源を使って処理するか

前者が **process switching** や **CEF** の話で、後者が **CAM**、**TCAM**、**centralized forwarding**、**distributed forwarding**、**SDM Template** の話です。

つまり、

- **転送ロジックの話**: **process switching** / **CEF**
- **転送を支える実装の話**: **CAM** / **TCAM** / ASIC / 分散転送

という2層に分けると、全体像がかなり見やすくなります。

## CAMとは何か

まず **CAM** は Content Addressable Memory のことです。

スイッチの文脈では、**CAM** という言葉は **MAC address table を高速に引く仕組み** と対応づけて理解すると分かりやすいです。

L2 Forwardingでは、スイッチは受信フレームの宛先MACアドレスを見て、

- そのMACアドレスがどのポートの先にいるか
- どのポートへ転送すべきか

を判断していました。

このときに内部で使われる代表的な高速検索の仕組みが CAM です。

試験勉強では、

- **MAC address table** は CLI で見える表
- **CAM** はその表を高速検索するハードウェア的な仕組み

という対応で押さえておくと分かりやすいです。

### CAMで意識したいこと

- L2転送と結びつきやすい
- MACアドレスベースの検索を高速化する
- `show mac address-table` で見える内容と関連づけて理解する

## TCAMとは何か

次に **TCAM** は Ternary Content Addressable Memory のことです。

CAMよりも柔軟に検索できるメモリで、Layer 2だけでなくLayer 3やACL、QoSなど、**複数条件を同時に評価したい処理** に向いています。

本で出てくるポイントを、学習者目線でかなりざっくり言うとこうなります。

- **CAM**: 主にMACアドレスのような単純な一致検索
- **TCAM**: IPアドレス、プロトコル、ACL条件、QoS分類など、より複雑な条件検索

つまり、**TCAM** は **L2/L3転送やACL判定をハードウェアで高速に処理するための重要な資源** です。

### TCAMで意識したいこと

- `L2/L3 forwarding`
- `ACL`
- `QoS`
- `ポリシー判定`

のような処理を高速化する

そして大事なのは、TCAMは無限ではないということです。

スイッチのハードウェア資源には限りがあるため、何にどれだけ割り当てるかという話が後で出てくる **SDM Template** につながります。

## Process Switchingとは何か

昔のCiscoルーターでは、受信したパケットをCPUがその都度細かく処理して転送判断していました。これが **process switching** です。

**process switching** では、パケットが来るたびにCPUが

- 宛先IPアドレスを見て
- ルーティングテーブルを確認して
- 次ホップを決めて
- ARPテーブルを見て
- 必要なL2ヘッダー情報で書き換えて
- TTLを減らして
- 送信する

という処理を行います。

これが動作として間違いというわけではありませんが、トラフィック量が増えるとCPU負荷が高くなります。

つまり **process switching** は、

- 理屈は分かりやすい
- でも大量のトラフィックをさばくには遅い

という仕組みです。

## CEFとは何か

そこで出てくるのが **CEF**、Cisco Express Forwarding です。

CEFの考え方はシンプルで、毎回CPUが一から考えるのではなく、**転送に必要な情報をあらかじめまとめておき、高速に引けるようにする** というものです。

これが前回のラボで見た

- `RIB`
- `FIB`
- `Adjacency Table`

の関係です。

### 役割のイメージ

- `RIB`: 経路情報の元になる表
- `FIB`: 宛先プレフィックスに対して、どこへ送るかを高速に引く表
- `Adjacency Table`: next-hopへ実際にどうやって送るかを持つ表

つまり CEF は、毎回CPUがルーティングテーブルやARPテーブルを個別に見に行くのではなく、**転送に必要な情報を事前に構造化して持っている** 仕組みです。

## Process SwitchingとCEFの違い

この2つの違いは、次のように見ると分かりやすいです。

- `process switching`: パケットごとにCPUが細かく判断する
- `CEF`: 事前に作ったFIBとAdjacencyを使って高速に転送する

言い換えると、

- `process switching` は都度判断
- `CEF` は事前計算して高速参照

です。

試験対策として大事なのは、**CEFがふだんの高速転送で、process switchingは例外処理やフォールバック側の話** だと理解しておくことです。

※ フォールバックは、通常の方法で処理できないときに、別の方法へ切り替えることです。

## Puntとは何か

CEFで処理できないパケットは、CPU側へ送られます。これを **punt** と考えると分かりやすいです。

※ `punt` は、ラグビーなどでボールを足で蹴って前方へ送るときに使われる言葉です。

たとえば、

- ARP未解決で adjacency が incomplete なとき
- 追加処理が必要な特殊なパケットのとき
- 機器自身が受け取る制御系トラフィックのとき

などは、ハードウェアの高速転送だけでは処理しきれず、CPU側で扱われます。

第3回の記事で `show adjacency detail` を見たときに、

- `incomplete`
- `punt (rate-limited) packets`

のような表示が出ることがありました。

これは、**転送用の情報がまだ完全ではないため、高速転送から外れてCPU側の助けが必要な状態** と理解できます。

ここまで来ると、

- 通常時は CEF
- 例外時は punt
- punt の先で CPU が処理する

という流れが見えてきます。

## Software CEFとは何か

**software CEF** は、CEFのデータ構造を使いつつ、主にソフトウェア側、つまりCPUベースで転送処理を行う考え方です。

ここで大事なのは、**software CEF** でも CEF の考え方自体は同じという点です。

違うのは、

- どこで処理するか
- どのハードウェア資源を使うか

です。

つまり **software CEF** は、

- FIBやAdjacencyを使う
- ただし転送処理の主体はCPU寄り

という理解で大丈夫です。

## Hardware CEFとは何か

一方 **hardware CEF** は、CEFの情報をASICやTCAM、NPUなどの専用ハードウェアに載せて高速転送する考え方です。

ここで重要なのは、

- CPUが毎回全部処理するのではない
- ハードウェアがCEF情報をもとに高速転送する

という点です。

Enterprise向けスイッチや高性能ルーターでは、この **hardware CEF** の考え方が非常に重要です。

つまり、

- **software CEF**: CEFをソフトウェア寄りで使う
- **hardware CEF**: CEFをハードウェア寄りで使う

という違いです。

## Centralized Forwardingとは何か

**centralized forwarding** は、転送判断を中心のフォワーディングエンジンでまとめて処理する方式です。

ざっくりいうと、受信したパケットを中央の頭脳に集めてから、

- どこへ出すか
- どう転送するか

を判断するイメージです。

この方式はイメージしやすい一方で、中央の処理系に負荷が集まりやすくなります。

## Distributed Forwardingとは何か

これに対して **distributed forwarding** は、各ラインカードや各転送モジュール側にも転送判断の仕組みを持たせて、分散的に処理する方式です。

※ ラインカードは、ポート群とその転送機能を受け持つ単位と考えると分かりやすいです。

つまり、全部を中央に集めるのではなく、

- 入口側や各モジュール側で
- できるだけその場で転送判断する

という考え方です。

その結果、

- 高いスループットを出しやすい
- ポート密度や拡張性に有利

というメリットにつながります。

## dCEFとは何か

**dCEF** は distributed Cisco Express Forwarding のことです。

これは、CEF のデータ構造を各ラインカードや各転送エンジン側にも配って、**分散的に CEF 転送を行えるようにする仕組み** です。

ここをつなげて考えると分かりやすいです。

- CEF: 高速転送の仕組み
- distributed forwarding: 転送判断を分散させる考え方
- dCEF: 分散環境でCEFを使う仕組み

つまり dCEF は、**CEF** と **distributed forwarding** が合体したイメージです。

## SDM Templateとは何か

ここで TCAM の話に戻ります。

TCAM などのハードウェア資源は限られているため、

- MACアドレスにどれだけ割り当てるか
- ルートにどれだけ割り当てるか
- ACLやQoSにどれだけ割り当てるか

を、機器の役割に応じて調整したくなります。

そのための考え方が **SDM Template** です。

**SDM** は Switching Database Manager の略で、スイッチ内部のハードウェア資源配分をテンプレートとして持っています。

たとえば、L2中心なのか、L3機能をより多く使うのかで、最適な配分は変わります。

つまり SDM Template は、

- ハードウェア資源の使い道の配分表

のようなものです。

## SDM Templateを実機で確認するコマンド

実機で確認するときは、たとえば Catalyst 系スイッチで次のようなコマンドを使います。

実機で入力:

```cisco
show sdm prefer
show running-config | include sdm
show platform tcam utilization
```

機種によっては使えるコマンドに差がありますが、少なくとも

- 現在どのテンプレートが使われているか
- TCAM利用状況がどうなっているか

を確認する入口としては押さえておきたいところです。

注意点として、`sdm prefer` の変更は reload を伴うことがあります。

そのため、学習目的ではまず

- `show` で現状確認する
- 変更は別日に切り分ける

くらいの運用が安全です。

## これまでの記事とどうつながるか

ここまでの話を、これまでの記事とつなげると次のように見えてきます。

### L2 Forwardingで見ていたもの

- `show mac address-table`
- VLAN
- access port
- trunk port

ここでは主に **CAM** と **MAC address table** の世界を見ていました。

### L3 Forwardingで見ていたもの

- `show ip route`
- `show arp`
- traceroute
- ホップごとのMAC書き換え

ここでは、ルーティングとARPを使って次ホップへ送るという、**process switching** でも **CEF** でも必要になる基本動作を見ていました。

### CEFで見ていたもの

- `show ip cef`
- `show ip cef ... detail`
- `show adjacency detail`
- incomplete adjacency

ここでは、**CEF** の中身である **FIB** と **Adjacency Table** を直接見ていました。

つまり、第1回から第3回までは単発ではなく、

- L2の転送
- L3の転送
- その高速化の仕組み

という流れでつながっています。

## ここまでを一気に振り返ると

Packet Forwardingの章後半は、次のように見ていくと頭に入りやすいです。

- **CAM** は主にMACアドレス検索と結びつく
- **TCAM** はL2/L3転送、ACL、QoSなど複数条件評価に使われる
- **process switching** はCPUが都度転送判断する方式
- **CEF** はFIBとAdjacencyを使って高速転送する方式
- **punt** はCEFだけで処理できないパケットがCPU側へ回ること
- **software CEF** はCPU寄りのCEF
- **hardware CEF** はASICやTCAMなどを使うCEF
- **centralized forwarding** は中央で転送判断する方式
- **distributed forwarding** は分散して転送判断する方式
- **dCEF** は分散環境でCEFを使う仕組み
- **SDM Template** はTCAMなどのハードウェア資源配分を決める考え方

## まとめ

Packet Forwardingの章は、最初はL2とL3の基本動作の話に見えますが、後半に進むと「ネットワーク機器がその転送をどうやって高速に実現しているか」というアーキテクチャの話に広がっていきます。

ここが少し難しく感じるのは自然で、**CAM**、**TCAM**、**CEF**、**dCEF**、**SDM Template** といった用語が、装置内部の実装と結びついているからだと思います。

ただ、今回のように

- 何を判断する仕組みなのか
- どこで処理する話なのか
- どのハードウェア資源と関係するのか

を分けて考えると、かなり理解しやすくなります。

特に試験対策としては、

- **process switching** と **CEF** の違い
- **CAM** と **TCAM** の違い
- **centralized forwarding** と **distributed forwarding** の違い
- **SDM Template** がハードウェア資源配分の話であること

を押さえておくと、Chapter 1 の後半がかなりつかみやすくなるはずです。

## 関連リンク

- L2 Forwarding: https://zenn.dev/mnv/articles/1f9cdc818dcb5d
- L3 Forwarding: https://zenn.dev/mnv/articles/609b88d8ae3a6f
- CEF / FIB / Adjacency: https://zenn.dev/mnv/articles/17e3f4490321fc
- Extra Check: https://zenn.dev/mnv/articles/4f04e38837ba5e
