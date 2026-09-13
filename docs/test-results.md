# 試験結果

## 1. 試験概要

Leaf-Spine構成において、OSPF Neighbor、ECMP、End-to-End疎通、Spine1のLeaf向けリンク停止時の経路切り替え、および復旧後のECMP再形成を確認した。

## 2. 試験環境

| 項目 | 内容 |
|---|---|
| 検証環境 | Cisco Packet Tracer |
| 構成 | 2-Spine / 2-Leaf |
| ルーティングプロトコル | OSPF |
| OSPF Area | 0 |
| Leaf-Spine間 | Layer3 Point-to-Point |
| Server | 192.168.10.10/24 |
| PC | 192.168.20.10/24 |

## 3. 試験結果一覧

| No. | 試験項目 | 期待結果 | 結果 |
|---|---|---|---|
| 1 | OSPF Neighbor確認 | 各LeafがSpine1・Spine2とFULLになる | OK |
| 2 | Leaf1 ECMP確認 | 192.168.20.0/24への2経路が登録される | OK |
| 3 | Leaf2 ECMP確認 | 192.168.10.0/24への2経路が登録される | OK |
| 4 | 正常時End-to-End疎通 | ServerからPCへPing成功 | OK |
| 5 | Spine1のLeaf向けリンク停止時Neighbor確認 | Spine1とのNeighborが消失する | OK |
| 6 | Spine1のLeaf向けリンク停止時経路確認 | Spine2経由の1経路へ切り替わる | OK |
| 7 | Spine1のLeaf向けリンク停止時疎通確認 | OSPF収束後もServerからPCへ疎通可能 | OK |
| 8 | Spine1のLeaf向けリンク復旧確認 | Spine1とのNeighborが再確立する | OK |
| 9 | ECMP再形成確認 | 復旧後に2経路へ戻る | OK |

## 4. OSPF Neighbor確認

### Leaf1

`show ip ospf neighbor` を実行し、Spine1およびSpine2とのNeighborがともに `FULL/-` であることを確認した。

```text
Neighbor ID     Pri   State           Address         Interface
10.0.21.2         0   FULL/  -        10.0.11.2       FastEthernet0/1
10.0.22.2         0   FULL/  -        10.0.12.2       FastEthernet0/2
```

### Leaf2

```text
Neighbor ID     Pri   State           Address         Interface
10.0.21.2         0   FULL/  -        10.0.21.2       FastEthernet0/1
10.0.22.2         0   FULL/  -        10.0.22.2       FastEthernet0/2
```

判定：**OK**

## 5. ECMP確認

### Leaf1

Leaf1からLeaf2配下の `192.168.20.0/24` への経路を確認した。

```text
O    192.168.20.0 [110/3] via 10.0.11.2, FastEthernet0/1
                  [110/3] via 10.0.12.2, FastEthernet0/2
```

Spine1・Spine2を経由する2つの等コスト経路が登録されていることを確認した。

### Leaf2

Leaf2からLeaf1配下の `192.168.10.0/24` への経路を確認した。

```text
O    192.168.10.0 [110/3] via 10.0.21.2, FastEthernet0/1
                  [110/3] via 10.0.22.2, FastEthernet0/2
```

こちらも2つの等コスト経路が登録されていることを確認した。

判定：**OK**

## 6. 正常時End-to-End疎通確認

ServerからPCへPingを実施した。

```text
Server : 192.168.10.10
PC     : 192.168.20.10
```

Serverから `192.168.20.10` へのPingが成功し、Leaf1とLeaf2をまたぐEnd-to-End通信が可能であることを確認した。

判定：**OK**

## 7. Spine1のLeaf向けリンク停止試験

Spine1のLeaf向けインターフェースを停止し、Spine1を経由できない状態を作成した。

```text
interface range FastEthernet0/1 - 2
 shutdown
```

### OSPF Neighbor

障害発生後、Leaf1ではSpine1とのNeighborが消失し、Spine2とのNeighborのみが残ることを確認した。

```text
Neighbor ID     Pri   State           Address         Interface
10.0.22.2         0   FULL/  -        10.0.12.2       FastEthernet0/2
```

### ルーティングテーブル

正常時に2経路存在していた `192.168.20.0/24` が、Spine2経由の1経路へ切り替わったことを確認した。

```text
O    192.168.20.0 [110/3] via 10.0.12.2, FastEthernet0/2
```

### 障害時End-to-End疎通

Spine1のLeaf向けリンク停止中にServerからPCへPingを実施した。

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

Spine1のLeaf向けリンク停止後、OSPF収束後もSpine2経由でEnd-to-End疎通が可能であることを確認した。

判定：**OK**

## 8. Spine1のLeaf向けリンク復旧試験

Spine1のLeaf向けインターフェースを復旧した。

```text
interface range FastEthernet0/1 - 2
 no shutdown
```

復旧後、Leaf1でSpine1・Spine2とのOSPF Neighborが再び `FULL/-` となることを確認した。

また、`192.168.20.0/24` への経路が再び2つの等コスト経路となり、ECMPが再形成されたことを確認した。

判定：**OK**

## 9. 総合結果

OSPFによる動的ルーティングが正常に動作し、正常時にはSpine1・Spine2を利用したECMPが成立することを確認した。

また、Spine1のLeaf向けリンク停止時にはOSPFがSpine2経由の経路へ収束し、その後もEnd-to-End疎通が可能であることを確認した。

Spine1のLeaf向けリンク復旧後にはOSPF Neighborが再確立され、ECMPの2経路へ正常に復帰することも確認した。

総合判定：**OK**
