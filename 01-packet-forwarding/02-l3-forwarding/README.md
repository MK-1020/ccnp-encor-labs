# L3 Forwarding Lab

## 概要

CCNP ENCORの学習として、EVE-NGでL3 Forwardingの基本動作を検証したラボです。

## 検証内容

- ルーティングテーブルによる次ホップ決定
- ARPによる次ホップMACアドレス解決
- pingによる疎通確認
- tracerouteによるL3経路確認
- WiresharkによるIPヘッダとEthernetヘッダの確認

## 構成

PC1 --- R1 --- R2 --- R3 --- PC2

## IPアドレス

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

## 設定ファイル

- configs/R1.cfg
- configs/R2.cfg
- configs/R3.cfg
