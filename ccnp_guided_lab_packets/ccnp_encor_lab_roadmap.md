# CCNP ENCOR Lab Roadmap

この一覧は、`1ラボごとにEVE-NGで構成を組んで検証する` ための学習ロードマップです。

各ラボは、次の状態を目標にしています。

- 構成を自分でEVE-NGに再現できる
- コンフィグを手打ちできる
- `show` コマンドのどこを見ればよいか分かる
- 検証結果をそのままZenn記事にできる

本数は固定しません。

55本に収めることよりも、

- 読者が段階的に理解しやすいこと
- 自分で再現しやすいこと
- 既出記事と自然につながること

を優先します。

また、Chapter 1 を進めた結果、1章あたりの進め方は次の粒度がちょうどよさそうだと分かってきました。

- 本編ラボを2〜3本に分ける
- 必要なら Extra Check を1本作る
- 章末にまとめ記事を1本作る

このファイルでは、**Chapter 1 は実績ベース**、**Chapter 2 は次に作る具体案**、**Chapter 3 以降は暫定ロードマップ** として扱います。

## Chapter 1 Packet Forwarding

1. L2 Forwarding
2. L3 Forwarding
3. CEF / FIB / Adjacency
4. Extra Check
5. Chapter 1 Architecture Overview

実績ベースのファイル構成イメージ:

- `zenn_01_l2_forwarding.md`
- `zenn_02_l3_forwarding.md`
- `zenn_03_cef_fib_adjacency.md`
- `zenn_04_extra_check_packet_forwarding.md`
- `zenn_packet_forwarding_architecture_overview.md`

## Chapter 2 Spanning Tree Protocol

Chapter 2 は、Chapter 1 と同じ粒度で切るなら次の形が進めやすそうです。

1. STP Fundamentals
内容:
- ループが起きる理由
- BPDU
- root bridge
- root port / designated port / blocking port
- 802.1D の port state

2. STP Verification
内容:
- `show spanning-tree vlan ...` の見方
- root の確認
- root port / designated port / alternate or blocking の確認
- trunk 上で VLAN ごとに STP がどう見えるか

3. RSTP Basics
内容:
- 802.1D と RSTP の違い
- port state の違い
- 収束が速くなる考え方
- `show spanning-tree` で protocol が `rstp` と見えること

4. Extra Check
内容:
- topology change の見方
- `show spanning-tree ... detail` の確認
- trunk 上の VLAN 抜けや STP mismatch を見る入口

5. Chapter 2 Overview
内容:
- STP / RSTP の全体像
- root bridge を中心にどう木構造を作るか
- 802.1D と RSTP の違いの整理

想定ファイル構成:

- `02_spanning_tree_protocol/README.md`
- `02_spanning_tree_protocol/zenn_01_stp_fundamentals.md`
- `02_spanning_tree_protocol/zenn_02_stp_verification.md`
- `02_spanning_tree_protocol/zenn_03_rstp_basics.md`
- `02_spanning_tree_protocol/zenn_04_extra_check_spanning_tree_protocol.md`
- `02_spanning_tree_protocol/zenn_spanning_tree_protocol_architecture_overview.md`

補足:
- `zenn_` を主原稿兼検証メモとして使う
- コマンドと結果はノードごとにコードブロックを分ける
- 各章の下に `configs/` を置き、記事ごとにノード別 `.cfg` を分けてGitHubから取得できるようにする
- 今後は、各記事で使う全ノードのコンフィグを `configs/` 配下にそろえる。スイッチ、ルーター、VPCS を含め、配布漏れがない状態を基本形にする
- VPCS の設定ファイルは `set pcname ...` を先に書いてから IP / GW を入れ、最後に `save` を入れる
- `関連ファイル` はローカルファイル案内ではなく、GitHub上の `configs/` フォルダや `.cfg` へのリンクを載せる。記事Markdownへのリンクは原則載せない
- `関連ファイル` は、実記事に合わせて短い説明文とGitHubリンク1本を基本形にする
- 可能なラボでは、キャプチャ場所と `何が見えると勉強になるか` も入れる
- 必要な記事では、キャプチャ結果を本文中で解説する節も用意する
- キャプチャ画像を貼るときは、注目箇所を示したうえで、画像の下に `どこを見て` `それが何を意味して` `どう読めるか` を短く添える
- `show` 結果の直後には、`どこを見るか` `それが何を意味するか` `この結果から何が言えるか` を短く添える
- コマンド結果の説明は必要な箇所だけ短く入れる
- 既出の説明は繰り返しすぎない

## Chapter 3 Advanced STP Tuning

以下は、現時点では暫定の粒度です。Chapter 2 を進めたあとで同じように分解していく前提です。

- STP root primary / secondary
- port cost / port priority
- BPDU Guard / BPDU Filter
- Root Guard
- Extra Check
- Chapter Overview

## Chapter 4 Multiple Spanning Tree Protocol

- MST Region
- Instance
- VLAN Mapping
- 境界動作
- Extra Check
- Chapter Overview

## Chapter 5 VLAN Trunks and EtherChannel

- VTP / DTP / 802.1Q trunk
- LACP EtherChannel
- Extra Check
- Chapter Overview

## Chapter 6 IP Routing Essentials

- Static Route / Default Route / Floating Static
- PBR
- VRF Lite
- Extra Check
- Chapter Overview

## Chapter 7 EIGRP

- EIGRP Neighbor and Route Exchange
- Metric / Successor / Feasible Successor
- Extra Check
- Chapter Overview

## Chapter 8 OSPF

- OSPF Hello / Neighbor / DR / BDR
- OSPF Cost / Timer / Network Type / Loopback
- Extra Check
- Chapter Overview

## Chapter 9 Advanced OSPF

- Multi Area OSPF and LSA Flow
- OSPF Summary / Filtering / Path Selection
- Extra Check
- Chapter Overview

## Chapter 10 OSPFv3

- IPv6 OSPFv3
- IPv4 Address Family
- Extra Check
- Chapter Overview

## Chapter 11 BGP

- eBGP and iBGP Neighbor
- Route Advertisement / Next Hop / Aggregate
- Extra Check
- Chapter Overview

## Chapter 12 Advanced BGP

- Prefix List and Route Map
- Path Attribute Control
- Community
- Extra Check
- Chapter Overview

## Chapter 13 Multicast

- IGMP and IGMP Snooping
- PIM Sparse Mode / RP / RPF
- Extra Check
- Chapter Overview

## Chapter 14 Quality of Service

- Classification and Marking
- Policing / Shaping / Queuing
- Extra Check
- Chapter Overview

## Chapter 15 IP Services

- NTP
- FHRP
- NAT / PAT
- Extra Check
- Chapter Overview

## Chapter 16 Overlay Tunnels

- GRE
- GRE over IPsec or VTI
- DMVPN or VXLAN Basics
- Extra Check
- Chapter Overview

## Chapter 17 Wireless Signals and Modulation

- dBm / RSSI / Channel / Signal Strength Observation
- Extra Check
- Chapter Overview

## Chapter 18 Wireless Infrastructure

- AP / WLC / CAPWAP / Deployment Model
- Extra Check
- Chapter Overview

## Chapter 19 Wireless Roaming and Location Services

- L2 / L3 Roaming Flow
- Extra Check
- Chapter Overview

## Chapter 20 Authenticating Wireless Clients

- Open / PSK
- 802.1X / RADIUS / MAB
- Extra Check
- Chapter Overview

## Chapter 21 Troubleshooting Wireless Connectivity

- Client Connectivity Troubleshooting Flow
- Extra Check
- Chapter Overview

## Chapter 22 Enterprise Network Architecture

- 2-Tier / 3-Tier / L2 Access / Routed Access
- Extra Check
- Chapter Overview

## Chapter 23 Fabric Technologies

- SD-Access Underlay / Overlay
- SD-WAN Control Plane
- Extra Check
- Chapter Overview

## Chapter 24 Network Assurance

- ping / traceroute / debug
- SNMP / Syslog / NetFlow
- SPAN / IP SLA
- Extra Check
- Chapter Overview

## Chapter 25 Secure Network Access Control

- 802.1X / MAB / NAC
- TrustSec / MACsec
- Extra Check
- Chapter Overview

## Chapter 26 Network Device Access Control

- ACL / PACL / VACL / RACL
- SSH / AAA
- ZBFW / CoPP
- Extra Check
- Chapter Overview

## Chapter 27 Virtualization

- Virtual Switch / VM / Container / NFV
- Extra Check
- Chapter Overview

## Chapter 28 Foundational Network Programmability

- REST API / JSON / XML
- YANG / NETCONF / RESTCONF
- Python Basics for Network Verification
- Extra Check
- Chapter Overview

## Chapter 29 Introduction to Automation Tools

- EEM
- Ansible
- Extra Check
- Chapter Overview

## 進め方

- Chapter 1 の進め方をベースに、Chapter 2 も本編 + Extra Check + 章まとめで組む
- Chapter 2 が固まったら、Chapter 3 以降も同じ粒度へ見直す
- 本数を先に固定せず、進める章から順に具体化する
