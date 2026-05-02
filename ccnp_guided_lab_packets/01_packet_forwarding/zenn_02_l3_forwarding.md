# CCNP ENCOR 試験対策: ルーティングテーブルとARPでL3 Forwardingを理解する

# EVE-NGで確認するL3 Forwarding：IPアドレスは変わらずMACアドレスはホップごとに変わる

## はじめに

- CCNP ENCOR official Cert Guideの内容に従って、ラボを構成して学習しています。
- 前回はL2 Forwardingとして、VLAN、MAC Address Table、802.1Qタグの動きを確認しました。
- 今回はL3に注目し、ルーターがパケットをどのように次のネットワークへ転送するのかをラボで検証、確認していきます。

L2では、スイッチがMACアドレステーブルを使ってフレームを転送していました。

一方、L3ではルーターがルーティングテーブルを使って、宛先ネットワークへの次ホップを判断します。

今回の検証では、PC1からPC2へpingを行い、ルーターをまたいだ通信で、

- IPアドレスは変わらない
- MACアドレスはホップごとに変わる
- ルーターはARPで次ホップのMACアドレスを解決する
- tracerouteでL3の通過経路を確認できる

という点を確認します。

## 今回確認したいこと

- ルーターがルーティングテーブルを見て次ホップを決定すること
- 次ホップへ転送するためにARPテーブルを使うこと
- ルーターをまたいでも送信元IPアドレスと宛先IPアドレスは変わらないこと
- ルーターをまたぐたびに送信元MACアドレスと宛先MACアドレスが変わること
- tracerouteでL3の通過経路を確認できること
- ルートがない場合、途中のルーターからDestination host unreachableが返ること

## 検証トポロジ

![](https://static.zenn.studio/user-upload/bace7e215605-20260426.png)

## IPアドレス設計

| Device | Interface | IP Address |
|---|---|---|
| PC1 | e0 | 192.168.10.10/24 |
| R1 | Gi0/0 | 192.168.10.1/24 |
| R1 | Gi0/1 | 10.12.12.1/30 |
| R2 | Gi0/0 | 10.12.12.2/30 |
| R2 | Gi0/1 | 10.23.23.1/30 |
| R3 | Gi0/0 | 10.23.23.2/30 |
| R3 | Gi0/1 | 192.168.30.1/24 |
| PC2 | e0 | 192.168.30.10/24 |

## ルーターの設定

今回は、OSPFなどの動的ルーティングプロトコルは使わず、スタティックルートでL3 Forwardingの基本動作を確認します。

### R1の設定

```cisco
hostname R1

interface GigabitEthernet0/0
 description To_PC1
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 description To_R2
 ip address 10.12.12.1 255.255.255.252
 no shutdown

ip route 192.168.30.0 255.255.255.0 10.12.12.2
```

R1では、PC2側ネットワークである `192.168.30.0/24` への通信をR2へ向けます。

### R2の設定

```cisco
hostname R2

interface GigabitEthernet0/0
 description To_R1
 ip address 10.12.12.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 description To_R3
 ip address 10.23.23.1 255.255.255.252
 no shutdown

ip route 192.168.10.0 255.255.255.0 10.12.12.1
ip route 192.168.30.0 255.255.255.0 10.23.23.2
```

R2は中継ルーターなので、PC1側ネットワークとPC2側ネットワークの両方へのルートを持たせます。

### R3の設定

```cisco
hostname R3

interface GigabitEthernet0/0
 description To_R2
 ip address 10.23.23.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 description To_PC2
 ip address 192.168.30.1 255.255.255.0
 no shutdown

ip route 192.168.10.0 255.255.255.0 10.23.23.1
```

R3では、PC1側ネットワークである `192.168.10.0/24` への通信をR2へ向けます。

## 検証1: インターフェース状態の確認

まず、各ルーターでインターフェースがupしていることを確認します。

```cisco
show ip interface brief
```

確認ポイントは以下です。

- IPアドレスが想定通り設定されていること
- Statusがupであること
- Protocolもupであること

```cisco
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.10.1    YES manual up                    up
GigabitEthernet0/1     10.12.12.1      YES manual up                    up
```

```cisco
R2#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     10.12.12.2      YES manual up                    up
GigabitEthernet0/1     10.23.23.1      YES manual up                    up
```

```cisco
R3#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     10.23.23.2      YES manual up                    up
GigabitEthernet0/1     192.168.30.1    YES manual up                    up
```

これで、物理・L2的なリンクは問題なさそうです。

## 検証2: ルーティングテーブルの確認

次に、ルーティングテーブルを確認します。

```cisco
show ip route
```

R1では、PC2側ネットワークへのスタティックルートが入っていることを確認します。

```cisco
R1#show ip route 192.168.30.10
Routing entry for 192.168.30.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.12.12.2
      Route metric is 0, traffic share count is 1
```

R1は、`192.168.30.0/24` へ行くには、次ホップ `10.12.12.2`、つまりR2へ送ればよいと判断しています。

R2では、PC1側とPC2側の両方へのルートを確認します。

```cisco
R2#show ip route 192.168.10.10
Routing entry for 192.168.10.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.12.12.1
      Route metric is 0, traffic share count is 1
```

```cisco
R2#show ip route 192.168.30.10
Routing entry for 192.168.30.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.23.23.2
      Route metric is 0, traffic share count is 1
```

R3では、PC1側ネットワークへのルートを確認します。

```cisco
R3#show ip route 192.168.10.10
Routing entry for 192.168.10.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.23.23.1
      Route metric is 0, traffic share count is 1
```

これで、各ルーターが宛先ネットワークへの経路を持っていることを確認できました。

## 検証3: ARPテーブルの確認

次にARPテーブルを確認します。

```cisco
show arp
```

ルーターはルーティングテーブルで次ホップを決めますが、実際に同一リンク上の次ホップへフレームを送るには、次ホップのMACアドレスが必要です。

そのため、ルーターはARPによって次ホップのIPアドレスに対応するMACアドレスを解決します。

R1では、R2のMACアドレスをARPテーブルに持っていることを確認します。

```cisco
R1#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.12.12.1              -   5000.0001.0001  ARPA   GigabitEthernet0/1
Internet  10.12.12.2             31   5000.0002.0000  ARPA   GigabitEthernet0/1
Internet  192.168.10.1            -   5000.0001.0000  ARPA   GigabitEthernet0/0
Internet  192.168.10.10           7   0050.7966.6804  ARPA   GigabitEthernet0/0
```

R2では、R1側とR3側のMACアドレスを確認します。

```cisco
R2#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.12.12.1             34   5000.0001.0001  ARPA   GigabitEthernet0/0
Internet  10.12.12.2              -   5000.0002.0000  ARPA   GigabitEthernet0/0
Internet  10.23.23.1              -   5000.0002.0001  ARPA   GigabitEthernet0/1
Internet  10.23.23.2             24   5000.0003.0000  ARPA   GigabitEthernet0/1
```

R3では、R2とPC2のMACアドレスを確認します。

```cisco
R3#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.23.23.1             26   5000.0002.0001  ARPA   GigabitEthernet0/0
Internet  10.23.23.2              -   5000.0003.0000  ARPA   GigabitEthernet0/0
Internet  192.168.30.1            -   5000.0003.0001  ARPA   GigabitEthernet0/1
Internet  192.168.30.10          13   0050.7966.6805  ARPA   GigabitEthernet0/1
```

ここで重要なのは、R1がPC2のMACアドレスを知っているわけではないという点です。

R1が知っていればよいのは、次ホップであるR2のMACアドレスです。

つまり、L3転送では、

- ルーティングテーブルで次ホップを決める
- ARPテーブルで次ホップのMACアドレスを確認する
- そのMACアドレス宛にEthernetフレームを作り直して送信する

という流れになります。

## 検証4: pingによる疎通確認

PC1からPC2へpingを実行します。

```text
PC1> ping 192.168.30.10

84 bytes from 192.168.30.10 icmp_seq=1 ttl=61 time=7.799 ms
84 bytes from 192.168.30.10 icmp_seq=2 ttl=61 time=3.176 ms
84 bytes from 192.168.30.10 icmp_seq=3 ttl=61 time=4.257 ms
84 bytes from 192.168.30.10 icmp_seq=4 ttl=61 time=4.346 ms
84 bytes from 192.168.30.10 icmp_seq=5 ttl=61 time=4.611 ms
```

PC1からPC2へpingが成功しました。

これにより、PC1からR1、R2、R3を経由してPC2までL3転送できていることが確認できました。

## 検証5: tracerouteで通過経路を確認

次に、tracerouteを実行します。

```text
PC1> trace 192.168.30.10
trace to 192.168.30.10, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   2.706 ms  2.590 ms  2.403 ms
 2   10.12.12.2   3.421 ms  3.163 ms  4.200 ms
 3   10.23.23.2   5.102 ms  4.597 ms  4.722 ms
 4   *192.168.30.10   6.232 ms (ICMP type:3, code:3, Destination port unreachable)
```

tracerouteでは、PC1からPC2までの通信が、どのL3機器を通過しているかを確認できます。

今回の結果では、以下の順番で通信していることが分かります。

```text
PC1 → R1 → R2 → R3 → PC2
```

pingは「宛先まで届くか」を確認するコマンドですが、tracerouteは「どのルーターを経由して届いたか」を確認するコマンドです。

今回のようなL3 Forwardingの検証では、show ip routeだけでなく、tracerouteも使うことで、実際の通過経路を確認できます。

## 検証6: ルートがない場合の動き

検証中、R2に `192.168.30.0/24` へのルートが入っていない状態でpingを実行すると、以下のような結果になりました。

```text
PC1> ping 192.168.30.10

192.168.30.10 icmp_seq=1 timeout
*10.12.12.2 icmp_seq=2 ttl=254 time=3.780 ms (ICMP type:3, code:1, Destination host unreachable)
*10.12.12.2 icmp_seq=3 ttl=254 time=3.547 ms (ICMP type:3, code:1, Destination host unreachable)
*10.12.12.2 icmp_seq=4 ttl=254 time=3.564 ms (ICMP type:3, code:1, Destination host unreachable)
*10.12.12.2 icmp_seq=5 ttl=254 time=3.429 ms (ICMP type:3, code:1, Destination host unreachable)
```

ここで注目するのは、エラーを返している送信元が `10.12.12.2` である点です。

`10.12.12.2` はR2のR1側インターフェースです。

つまり、PC1から送信されたパケットはR2までは届いています。

しかし、R2が `192.168.30.0/24` へのルートを持っていなかったため、R2が `Destination host unreachable` を返しています。

この結果から、pingが失敗した場合でも、

- どの機器がエラーを返しているか
- どこまでは通信が届いているか
- どのルーターのルーティングテーブルを確認すべきか

を判断できることが分かりました。

## パケットキャプチャしてみた

次にWiresharkで、実際のパケットを確認します。

今回は以下の2か所でキャプチャしました。

- R1-R2間
- R2-R3間

PC1からPC2へpingを実行し、ICMP Echo Requestの中身を確認します。

### R1-R2間のキャプチャ

![](https://static.zenn.studio/user-upload/6e9cd128b960-20260426.png)

R1-R2間では、IPヘッダは以下のようになっていました。

| Field | Value |
|---|---|
| Source IP | 192.168.10.10 |
| Destination IP | 192.168.30.10 |

送信元IPアドレスはPC1、宛先IPアドレスはPC2です。

一方で、Ethernetヘッダでは以下のようになっていました。

| Field | Value |
|---|---|
| Source MAC | R1のMACアドレス |
| Destination MAC | R2のMACアドレス |

つまり、R1-R2間では、R1がR2へフレームを転送していることが分かります。

### R2-R3間のキャプチャ

![](https://static.zenn.studio/user-upload/4c681cdc4e43-20260426.png)

R2-R3間でも、IPヘッダは変わりません。

| Field | Value |
|---|---|
| Source IP | 192.168.10.10 |
| Destination IP | 192.168.30.10 |

送信元IPアドレスと宛先IPアドレスは、R1-R2間と同じです。

しかし、Ethernetヘッダは以下のように変わっていました。

| Field | Value |
|---|---|
| Source MAC | R2のMACアドレス |
| Destination MAC | R3のMACアドレス |

つまり、R2は受け取ったIPパケットをR3へ転送する際に、Ethernetヘッダを作り直していることが分かります。

### IPアドレスとMACアドレスの違い

今回のキャプチャから、以下のことが確認できました。

```text
IPヘッダ:
送信元IP = PC1
宛先IP   = PC2
→ ルーターをまたいでも基本的に変わらない

Ethernetヘッダ:
送信元MAC = そのリンクで送信する機器
宛先MAC   = そのリンクで次に受け取る機器
→ ルーターをまたぐたびに変わる
```

L3では、IPアドレスを使って最終的な宛先を判断します。

しかし、実際に次の機器へ届けるためには、同一リンク上で使うL2のMACアドレスが必要です。

そのため、ルーターはパケットを転送するたびにEthernetヘッダを作り直します。

## まとめ

今回の検証で、以下を確認できました。

- ルーターはルーティングテーブルを見て次ホップを判断する
- 次ホップへ転送するにはARPでMACアドレスを解決する必要がある
- 送信元IPアドレスと宛先IPアドレスは、ルーターをまたいでも基本的に変わらない
- 送信元MACアドレスと宛先MACアドレスは、ホップごとに変わる
- tracerouteを使うと、L3の通過経路を確認できる
- ルートがない場合、途中のルーターからDestination host unreachableが返る

前回のL2 Forwardingでは、スイッチがMACアドレステーブルを使って同一VLAN内でフレームを転送する動きを確認しました。

今回のL3 Forwardingでは、ルーターがルーティングテーブルとARPテーブルを使って、異なるネットワーク間でパケットを転送する動きを確認しました。

特に重要なのは、IPアドレスとMACアドレスの役割の違いです。

IPアドレスは、最終的な送信元と宛先を表します。

一方、MACアドレスは、同一リンク上で次の機器へ届けるために使われます。

この違いをWiresharkで確認すると、L3 Forwardingの動きがかなり具体的に理解できました。

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/01_packet_forwarding/configs/l3_forwarding
