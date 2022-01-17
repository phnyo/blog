---
title: "CISCO 操作メモ"
date: 2022-01-16T17:10:15+09:00
tags:
  - memo
---

## 操作方法チートシート

### スイッチ系

特権モードに切り替え
```
enable
```

現在のconfig
```
show running-config / show run
```

copy current configuration file to the startup configuration file
```
copy running-config startup-config
```

terminal configuration
```
configure terminal / config t
```

### configuration 

line console config
```
console line (num)
```

line console password
```
password (password)
login
```

enable password
```
enable password (passwd)
```

enable secret
```
enable secret (passwd)
```

encryption
```
service password-encryption
```

message of the day
```
banner motd "message"
```

interface (vlan)
```
interface vlan (num)
```

複数個の場合
```
interface range (connector)
```

up
```
no shutdown
```

各物理層のデバイスにメッセージ設定
```
description (message)
```

show ip interfaces
```
show ip interface brief / (name)
```
```
show ipv6 interface brief
```

least length of the password
```
security password min-length (num)
```


### IP設定とか

set ipv4 address 
```
ip address (ip address) (subnet musk)
```
サブネットのスイッチには〇〇.1を割り当てる（.0はネットワークアドレス）。

set ipv6 address
```
ipv6 address (ip address)/prefix-length
```
ipv6の下はLLAになる。

```
| GUA (global user? address) | LLA (link local address) |
```
なのでLLAを参照しないとアドレスが出てこないので注意。

```
ipv6 address fe80::(address) link-local
```
LLAも設定できる。

ipv6は削除しないとアドレスがそのままなので、削除するようにする。
```
no ipv6 address
```

set default gateway (cisco product) ipv4

```
ip default-gateway (ip address)
```
v6
```
ipv6 default-gateway (ip address)
```

show connected ip route
```
show ip route
```

ipv6パケット転送可にする
```
ipv6 unicast-routing
```

### 接続ケーブル

- coaxial (同軸)
- fiber (光ファイバー？）なんか点が二つある
ほかはのりで。

### ssh / telnet

vtyとかを設定
```
line vty 0 4 (15?)
password (password)
login / login local (ローカルユーザーの場合)
transport input {ssh | telnet}
```

### DNS

DNS lookup 無効
```
no ip domain-lookup
```

### domain

domain名設定
```
ip domain-name (name)
```

### user

user生成
```
username (user) secret (secret)
```
なんかいかの突撃に対しn秒間ブロック
```
login block-for (secs) attempts (num) within (secs)
```
vtyでn分間放置でログアウト
```
exec-timeout (mins)
```

### rsa

```
crypto key generate rsa
```

