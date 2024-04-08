# Azure で CLUSTERPRO

```powershell
## NIC 追加
# resource-group: 既存
# name: これから作る NIC の名前
# vnet-name: 既存
# subnet: 既存
# network-security-group: 既存

# 仮想NIC vnic2 作成
az network nic create --resource-group exampleRG --name vnic2 --vnet-name vnet2 --subnet default --network-security-group vcom1-nsg

# VM からリソースを deallocate してから NIC を追加する。
az vm deallocate --resource-group myResourceGroup --name vcom1

# VM に NIC を追加するとサブネットがエラー
PS C:\Users\User> az vm nic add --resource-group exampleRG --vm-name vcom1 --nics vnic2
(SubnetsNotInSameVnet) Subnet default referenced by resource /subscriptions/475293d5-229d-463e-922e-5a7a4ffd7d0d/resourceGroups/exampleRG/providers/Microsoft.Network/networkInterfaces/vnic2/ipConfigurations/ipconfig1 is not in the same Virtual Network /subscriptions/475293d5-229d-463e-922e-5a7a4ffd7d0d/resourceGroups/EXAMPLERG/providers/Microsoft.Network/virtualNetworks/VNET1 as the subnets of other VMs in the availability set.
Code: SubnetsNotInSameVnet
Message: Subnet default referenced by resource /subscriptions/475293d5-229d-463e-922e-5a7a4ffd7d0d/resourceGroups/exampleRG/providers/Microsoft.Network/networkInterfaces/vnic2/ipConfigurations/ipconfig1 is not in the same Virtual Network /subscriptions/475293d5-229d-463e-922e-5a7a4ffd7d0d/resourceGroups/EXAMPLERG/providers/Microsoft.Network/virtualNetworks/VNET1 as the subnets of other VMs in the availability set.
PS C:\Users\User>

# 結局、新たな「仮想ネットワーク 192.168.0.0/24、仮想NIC 192.168.0.11/24 を作って VM に追加しようとすると、
# 既存の「仮想ネットワーク 10.0.0.0/16」のサブネットに含まれないアドレスだから、という理由で失敗した。

# そこで、仮想ネットワーク 10.0.0.0/16 に サブネット 10.0.1.0/24 を作り、仮想NIC vnic3 を作る。 
az network nic create --resource-group exampleRG --name vnic3 --vnet-name vnet1 --subnet subnet2 --network-security-group vcom1-nsg

# 10.0.1.4/24 の IP アドレスが割り当てられた。
# この vnic3 を VM に追加してみる。
az vm nic add --resource-group exampleRG --vm-name vcom1 --nics vnic3

# 追加成功した。
# VMを起動して、IPアドレス の構成を確認する。

kaz@vcom1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 60:45:bd:82:a0:3a brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6245:bdff:fe82:a03a/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:22:48:4b:a5:d9 brd ff:ff:ff:ff:ff:ff
    inet 10.0.1.4/24 metric 200 brd 10.0.1.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::222:48ff:fe4b:a5d9/64 scope link
       valid_lft forever preferred_lft forever
kaz@vcom1:~$

# 「2枚の NIC」に「異なるサブネットの IP アドレス」 を割り当てることができた!!

```

## memo

VM のサイズ (Standard_B2s_v2 のような文字列) を取得する方法

```ps1
az vm list-sizes --location eastus2 --output table
```