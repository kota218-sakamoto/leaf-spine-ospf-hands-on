# パラメータシート

## 1. 基本情報

| 項目 | 設定値 |
|---|---|
| 構成 | 2-Spine / 2-Leaf |
| ルーティングプロトコル | OSPF |
| OSPF Process ID | 1 |
| OSPF Area | 0 |
| Leaf-Spine間 Network Type | Point-to-Point |
| Leaf-Spine間サブネット | /30 |
| 検証環境 | Cisco Packet Tracer |

## 2. Spine1

### インターフェース

| Interface | 接続先 | IPアドレス | 種別 | OSPF |
|---|---|---|---|---|
| Fa0/1 | Leaf1 Fa0/1 | 10.0.11.2/30 | Routed Port | Area 0 / Point-to-Point |
| Fa0/2 | Leaf2 Fa0/1 | 10.0.21.2/30 | Routed Port | Area 0 / Point-to-Point |

### OSPF

| 項目 | 設定値 |
|---|---|
| Process ID | 1 |
| Area | 0 |
| Network | 10.0.11.0/30 |
| Network | 10.0.21.0/30 |

## 3. Spine2

### インターフェース

| Interface | 接続先 | IPアドレス | 種別 | OSPF |
|---|---|---|---|---|
| Fa0/1 | Leaf1 Fa0/2 | 10.0.12.2/30 | Routed Port | Area 0 / Point-to-Point |
| Fa0/2 | Leaf2 Fa0/2 | 10.0.22.2/30 | Routed Port | Area 0 / Point-to-Point |

### OSPF

| 項目 | 設定値 |
|---|---|
| Process ID | 1 |
| Area | 0 |
| Network | 10.0.12.0/30 |
| Network | 10.0.22.0/30 |

## 4. Leaf1

### インターフェース

| Interface | 接続先 | IPアドレス / VLAN | 種別 | OSPF |
|---|---|---|---|---|
| Fa0/1 | Spine1 Fa0/1 | 10.0.11.1/30 | Routed Port | Area 0 / Point-to-Point |
| Fa0/2 | Spine2 Fa0/1 | 10.0.12.1/30 | Routed Port | Area 0 / Point-to-Point |
| Fa0/10 | Server | VLAN10 | Access Port | - |
| Vlan10 | Server LAN | 192.168.10.1/24 | SVI | Area 0 |

### OSPF

| 項目 | 設定値 |
|---|---|
| Process ID | 1 |
| Area | 0 |
| Network | 10.0.11.0/30 |
| Network | 10.0.12.0/30 |
| Network | 192.168.10.0/24 |

## 5. Leaf2

### インターフェース

| Interface | 接続先 | IPアドレス / VLAN | 種別 | OSPF |
|---|---|---|---|---|
| Fa0/1 | Spine1 Fa0/2 | 10.0.21.1/30 | Routed Port | Area 0 / Point-to-Point |
| Fa0/2 | Spine2 Fa0/2 | 10.0.22.1/30 | Routed Port | Area 0 / Point-to-Point |
| Fa0/10 | PC | VLAN20 | Access Port | - |
| Vlan20 | PC LAN | 192.168.20.1/24 | SVI | Area 0 |

### OSPF

| 項目 | 設定値 |
|---|---|
| Process ID | 1 |
| Area | 0 |
| Network | 10.0.21.0/30 |
| Network | 10.0.22.0/30 |
| Network | 192.168.20.0/24 |

## 6. エンド端末

| 端末 | IPアドレス | サブネットマスク | デフォルトゲートウェイ | 接続先 |
|---|---|---|---|---|
| Server | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | Leaf1 Fa0/10 |
| PC | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | Leaf2 Fa0/10 |

## 7. VLAN

| VLAN ID | 用途 | ネットワーク | Gateway | Leaf |
|---|---|---|---|---|
| 10 | Server LAN | 192.168.10.0/24 | 192.168.10.1 | Leaf1 |
| 20 | PC LAN | 192.168.20.0/24 | 192.168.20.1 | Leaf2 |

## 8. Leaf-Spine接続一覧

| Link | Network | Leaf IP | Spine IP |
|---|---|---|---|
| Leaf1 - Spine1 | 10.0.11.0/30 | 10.0.11.1 | 10.0.11.2 |
| Leaf1 - Spine2 | 10.0.12.0/30 | 10.0.12.1 | 10.0.12.2 |
| Leaf2 - Spine1 | 10.0.21.0/30 | 10.0.21.1 | 10.0.21.2 |
| Leaf2 - Spine2 | 10.0.22.0/30 | 10.0.22.1 | 10.0.22.2 |

## 9. OSPF広告ネットワーク

| Device | Advertised Network | Area |
|---|---|---|
| Spine1 | 10.0.11.0/30 | 0 |
| Spine1 | 10.0.21.0/30 | 0 |
| Spine2 | 10.0.12.0/30 | 0 |
| Spine2 | 10.0.22.0/30 | 0 |
| Leaf1 | 10.0.11.0/30 | 0 |
| Leaf1 | 10.0.12.0/30 | 0 |
| Leaf1 | 192.168.10.0/24 | 0 |
| Leaf2 | 10.0.21.0/30 | 0 |
| Leaf2 | 10.0.22.0/30 | 0 |
| Leaf2 | 192.168.20.0/24 | 0 |
