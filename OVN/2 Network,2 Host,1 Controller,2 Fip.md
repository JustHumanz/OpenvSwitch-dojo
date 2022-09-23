# Topology

## 2 Network,2 Host,1 Controller,2 Fip
```
TODO topology
```

## All nodes
- `sudo add-apt-repository cloud-archive:ussuri`
- `sudo apt install -y openvswitch-common openvswitch-switch ovn-common ovn-host`

## Controller
- `sudo apt install -y ovn-central`

## Set remote sb&nb on Controller
- `ovn-sbctl set-connection ptcp:6642`
- `ovn-nbctl set-connection ptcp:6641`
- `netstat -lntp | grep 664`

## In controller, create logical switch and ports Set openvswitch on to sb&nb: 
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.7 external_ids:ovn-encap-type="geneve" external_ids:system-id="host1"`
- `ovn-nbctl ls-add net-1`
- `ovn-nbctl set logical_switch net-1 other_config:subnet="10.0.0.0/24" other_config:exclude_ips="10.0.0.1"`
- `ovn-nbctl lsp-add net-1 vm1`
- `ovn-nbctl lsp-set-addresses vm1 "00:00:00:00:02:01 10.0.0.10"`

- `ovn-nbctl lsp-add net-1 vm3`
- `ovn-nbctl lsp-set-addresses vm3 "00:00:00:00:02:02 10.0.0.20"`

- `ovn-nbctl ls-add net-2`
- `ovn-nbctl set logical_switch net-2 other_config:subnet="172.0.0.0/24" other_config:exclude_ips="172.0.0.1"`
- `ovn-nbctl lsp-add net-2 vm2`
- `ovn-nbctl lsp-set-addresses vm2 "00:00:00:00:03:01 172.0.0.10"`

- `ovn-nbctl lsp-add net-2 vm4`
- `ovn-nbctl lsp-set-addresses vm4 "00:00:00:00:03:02 172.0.0.20"`

- `ovn-nbctl lr-add router1`
- `ovn-nbctl lrp-add router1 router1-net1 00:00:00:00:0f:01 10.0.0.1/24`
- `ovn-nbctl lsp-add net-1 net1-router1`
- `ovn-nbctl lsp-set-addresses net1-router1 00:00:00:00:0f:01`
- `ovn-nbctl lsp-set-type net1-router1 router`
- `ovn-nbctl lsp-set-options net1-router1 router-port=router1-net1`

- `ovn-nbctl lrp-add router1 router1-net2 00:00:00:00:0f:02 172.0.0.1/24`
- `ovn-nbctl lsp-add net-2 net2-router1`
- `ovn-nbctl lsp-set-addresses net2-router1 00:00:00:00:0f:02`
- `ovn-nbctl lsp-set-type net2-router1 router`
- `ovn-nbctl lsp-set-options net2-router1 router-port=router1-net2`

- `ovn-nbctl show`

## Set dhcp
- `ovn-nbctl create dhcp_options cidr=10.0.0.0/24 options='"lease_time"="3600" "router"="10.0.0.1" "server_id"="10.0.0.1" "server_mac"="00:00:00:00:0f:01"'` ## Save the UUID
- `ovn-nbctl create dhcp_options cidr=172.0.0.0/24 options='"lease_time"="3600" "router"="172.0.0.1" "server_id"="172.0.0.1" "server_mac"="00:00:00:00:0f:02"'` ## Save the UUID

- `ovn-nbctl lsp-set-dhcpv4-options vm1 $CIDR_UUID`
- `ovn-nbctl lsp-set-dhcpv4-options vm3 $CIDR_UUID`

- `ovn-nbctl lsp-set-dhcpv4-options vm2 $CIDR_UUID`
- `ovn-nbctl lsp-set-dhcpv4-options vm4 $CIDR_UUID`

## In compute node 1, make ovn-controller connect to southbound db, and create test
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.33 external_ids:ovn-encap-type="geneve" external_ids:system-id="host2"`
- `ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure`
### VMS
```
virt-install --import --name cirros-vm-1 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:02:01 --check all=off

```
- `ovs-vsctl set interface vnet0 external_ids:iface-id=vm1`

```
virt-install --import --name cirros-vm-2 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:02:02 --check all=off

```
- `ovs-vsctl set interface vnet1 external_ids:iface-id=vm3`

## or run the following command in compute node 2 machine instead
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.174 external_ids:ovn-encap-type="geneve" external_ids:system-id="host3"`
- `ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure`
### VMS
```
virt-install --import --name cirros-vm-1 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:03:01 --check all=off

```
- `ovs-vsctl set interface vnet0 external_ids:iface-id=vm2`

```
virt-install --import --name cirros-vm-2 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:03:02 --check all=off

```
- `ovs-vsctl set interface vnet1 external_ids:iface-id=vm4`

## Create router 'gw1'
- `ovn-nbctl create Logical_Router name=gw1 options:chassis=host1`

## Create new logical switch and connect it to 'gw1'
- `ovn-nbctl ls-add fipNet`
- `ovn-nbctl lsp-add fipNet fipNet172-gw1`
- `ovn-nbctl lsp-set-type fipNet172-gw1 router`
- `ovn-nbctl lsp-set-addresses fipNet172-gw1 00:00:00:f1:00:00`
- `ovn-nbctl lsp-set-options fipNet172-gw1 router-port=gw1-fipNet172`

- `ovn-nbctl lsp-add fipNet fipNet10-gw1`
- `ovn-nbctl lsp-set-type fipNet10-gw1 router`
- `ovn-nbctl lsp-set-addresses fipNet10-gw1 00:00:00:f2:00:00`
- `ovn-nbctl lsp-set-options fipNet10-gw1 router-port=gw1-fipNet10`

## Create new port on router 'gw1'
- `ovn-nbctl lrp-add gw1 gw1-fipNet172 00:00:00:f1:00:00 192.168.100.254/24`
- `ovn-nbctl lrp-add gw1 gw1-fipNet10 00:00:00:f2:00:00 192.168.100.253/24`

## Create a new logical switch for connecting the 'gw1' and 'router1' routers
- `ovn-nbctl ls-add join`

## Connect 'gw1' to the 'join' switch
- `ovn-nbctl lrp-add gw1 gw1-join 00:00:00:00:fa:00 192.168.255.1/24`
- `ovn-nbctl lsp-add join join-gw1`
- `ovn-nbctl lsp-set-type join-gw1 router`
- `ovn-nbctl lsp-set-addresses join-gw1 00:00:00:00:fa:00`
- `ovn-nbctl lsp-set-options join-gw1 router-port=gw1-join`

## 'router1' to the 'join' switch
- `ovn-nbctl lrp-add router1 router1-join 00:00:00:00:fb:00 192.168.255.2/24`
- `ovn-nbctl lsp-add join join-router1`
- `ovn-nbctl lsp-set-type join-router1 router`
- `ovn-nbctl lsp-set-addresses join-router1 00:00:00:00:fb:00`
- `ovn-nbctl lsp-set-options join-router1 router-port=router1-join`

## Add static routes
- `ovn-nbctl lr-route-add gw1 "172.0.0.0/24" 192.168.255.2`
- `ovn-nbctl lr-route-add gw1 "10.0.0.0/24" 192.168.255.2`
- `ovn-nbctl lr-route-add router1 "0.0.0.0/0" 192.168.255.1`

## Create bridge mapping for eth1. map network name "phyNet" to br-eth1 (run on 'controller1/host1')
- `ovs-vsctl set open_vswitch . external-ids:ovn-bridge-mappings=phyNet:br-ex`

## Create localnet port on 'outside'. set the network name to "phyNet"
- `ovn-nbctl lsp-add fipNet fipNet-localnet`
- `ovn-nbctl lsp-set-addresses fipNet-localnet unknown`
- `ovn-nbctl lsp-set-type fipNet-localnet localnet`
- `ovn-nbctl lsp-set-options fipNet-localnet network_name=phyNet`

## Fip
- `ovn-nbctl lr-nat-add gw1 snat 192.168.100.254 10.0.0.0/24`
- `ovn-nbctl lr-nat-add gw1 snat 192.168.100.253 172.0.0.0/24`

- `ovn-nbctl lr-nat-add gw1 dnat_and_snat 192.168.100.160 10.0.0.10`
- `ovn-nbctl lr-nat-add gw1 dnat_and_snat 192.168.100.170 172.0.0.10`

## Test on Host2
**virsh console cirros-vm-1**
- `ping -c 3 172.0.0.10`
- `ping -c 3 192.168.100.1` #fip network

**virsh console cirros-vm-2**
- `ping -c 3 172.0.0.20`
- `ping -c 3 192.168.100.1` #fip network

## Test on Host3
**virsh console cirros-vm-1**
- `ping -c 3 10.0.0.10`
- `ping -c 3 192.168.100.1` #fip network

**virsh console cirros-vm-2**
- `ping -c 3 10.0.0.20`
- `ping -c 3 192.168.100.1` #fip network

## Test on laptop
- `ping -c 3 192.168.100.160`
- `ping -c 3 192.168.100.170`
