# ネットワーク設計書

## 1. 概要

Cisco Packet Tracerを使用し、2台のSpineスイッチと2台のLeafスイッチによるLeaf-Spineネットワークを構築する。

Leaf-Spine間はLayer3 Point-to-Pointリンクとし、OSPF Area 0を使用して動的ルーティングを行う。

Spineを2台配置することでLeaf間に複数経路を確保し、正常時はECMPによる等コストルーティングを使用する。また、Spine1のLeaf向けリンク停止時にも、残存するSpine2経由の経路を利用できる構成とする。

## 2. ネットワーク構成

![Network Topology](images/topology.png)

### 構成機器

| ホスト名 | 役割 |
|---|---|
| Spine1 | Spineスイッチ |
| Spine2 | Spineスイッチ |
| Leaf1 | Leafスイッチ / VLAN10ゲートウェイ |
| Leaf2 | Leafスイッチ / VLAN20ゲートウェイ |
| Server | VLAN10接続端末 |
| PC | VLAN20接続端末 |

## 3. 物理・論理接続

| 接続元 | インターフェース | 接続先 | インターフェース | ネットワーク |
|---|---|---|---|---|
| Leaf1 | Fa0/1 | Spine1 | Fa0/1 | 10.0.11.0/30 |
| Leaf1 | Fa0/2 | Spine2 | Fa0/1 | 10.0.12.0/30 |
| Leaf2 | Fa0/1 | Spine1 | Fa0/2 | 10.0.21.0/30 |
| Leaf2 | Fa0/2 | Spine2 | Fa0/2 | 10.0.22.0/30 |
| Server | NIC | Leaf1 | Fa0/10 | VLAN10 |
| PC | NIC | Leaf2 | Fa0/10 | VLAN20 |

## 4. IPアドレス設計

### Leaf-Spine間

| 機器 | インターフェース | IPアドレス |
|---|---|---|
| Leaf1 | Fa0/1 | 10.0.11.1/30 |
| Spine1 | Fa0/1 | 10.0.11.2/30 |
| Leaf1 | Fa0/2 | 10.0.12.1/30 |
| Spine2 | Fa0/1 | 10.0.12.2/30 |
| Leaf2 | Fa0/1 | 10.0.21.1/30 |
| Spine1 | Fa0/2 | 10.0.21.2/30 |
| Leaf2 | Fa0/2 | 10.0.22.1/30 |
| Spine2 | Fa0/2 | 10.0.22.2/30 |

### LAN

| 機器 | VLAN | IPアドレス | 用途 |
|---|---|---|---|
| Leaf1 | VLAN10 | 192.168.10.1/24 | Server側デフォルトゲートウェイ |
| Server | VLAN10 | 192.168.10.10/24 | 接続端末 |
| Leaf2 | VLAN20 | 192.168.20.1/24 | PC側デフォルトゲートウェイ |
| PC | VLAN20 | 192.168.20.10/24 | 接続端末 |

## 5. OSPF設計

- OSPF Process ID：1
- Area：0
- Leaf-Spine間：Point-to-Point
- Leaf1 LAN：192.168.10.0/24をOSPFで広告
- Leaf2 LAN：192.168.20.0/24をOSPFで広告
- Leaf1 Vlan10：Passive Interface
- Leaf2 Vlan20：Passive Interface

Leaf-Spine間は1対1のLayer3接続であるため、OSPF Network TypeをPoint-to-Pointとする。

これによりDR/BDR選出を行わず、各LeafとSpine間で直接OSPF Neighborを確立する。

## 6. 冗長化設計

Leaf1およびLeaf2は、それぞれSpine1とSpine2の両方へ接続する。

正常時は、Leaf間通信においてSpine1経由とSpine2経由のOSPFコストが等しくなるため、ECMPによる2経路がルーティングテーブルへ登録される。

### 正常時

```text
Leaf1
  ├─ Spine1 ─ Leaf2
  └─ Spine2 ─ Leaf2
```

### Spine1のLeaf向けリンク停止時

```text
Leaf1
  X  Spine1
  └─ Spine2 ─ Leaf2
```

Spine1のLeaf向けリンク停止時はSpine1とのOSPF Neighborが消失し、Spine2経由の経路のみが残る設計とする。

Spine1のLeaf向けリンク復旧後はNeighborを再確立し、再びECMPの2経路へ復帰する。

## 7. 障害試験方針

以下の項目を確認する。

1. 正常時にOSPF NeighborがFULL状態であること
2. 正常時にLeaf間でECMPが成立していること
3. ServerとPC間でEnd-to-End通信が可能であること
4. Spine1のLeaf向けリンク停止時にSpine1経由の経路が消失すること
5. Spine2経由へ経路が切り替わること
6. Spine1のLeaf向けリンク停止後、OSPF収束後もServerとPC間で疎通可能であること
7. Spine1のLeaf向けリンク復旧後にOSPF Neighborが再確立すること
8. 復旧後にECMPの2経路へ戻ること

## 8. 設計上のポイント

- Leaf-Spine間はL3 Routed Portで接続し、ファブリックの冗長化にSTPを使用しない構成とした
- OSPFによって経路情報を動的に交換する
- Point-to-Point Network Typeを使用してDR/BDR選出を不要とした
- 2台のSpineを利用してLeaf間経路を冗長化した
- ECMPにより正常時は複数経路を利用可能とした
- Spine1のLeaf向けリンク停止時にも残存経路で疎通可能であることを検証した
