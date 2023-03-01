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

## Controller
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.173:6642" external_ids:ovn-nb="tcp:192.168.122.173:6641" external_ids:ovn-encap-ip=192.168.122.173 external_ids:ovn-encap-type="geneve" external_ids:system-id="host1"`

## Compute 1
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.173:6642" external_ids:ovn-nb="tcp:192.168.122.173:6641" external_ids:ovn-encap-ip=192.168.122.91 external_ids:ovn-encap-type="geneve" external_ids:system-id="host2"`

## Compute 2
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.173:6642" external_ids:ovn-nb="tcp:192.168.122.173:6641" external_ids:ovn-encap-ip=192.168.122.181 external_ids:ovn-encap-type="geneve" external_ids:system-id="host3"`

### Create logical switch net-1 
- `ovn-nbctl ls-add net-1`
- `ovn-nbctl set logical_switch net-1 other_config:subnet="10.0.0.0/24" other_config:exclude_ips="10.0.0.1"`
- `ovn-nbctl lsp-add net-1 vm1`
- `ovn-nbctl lsp-set-addresses vm1 "00:00:00:00:01:01 10.0.0.10"`

### Create logical switch net-2
- `ovn-nbctl ls-add net-2`
- `ovn-nbctl set logical_switch net-2 other_config:subnet="172.0.0.0/24" other_config:exclude_ips="172.0.0.1"`
- `ovn-nbctl lsp-add net-2 vm2`
- `ovn-nbctl lsp-set-addresses vm2 "00:00:00:00:02:01 172.0.0.10"`

### Create logical router router1
- `ovn-nbctl lr-add router1`
- `ovn-nbctl lrp-add router1 router1-net1 00:00:00:00:0f:01 10.0.0.1/24`

### Attach logical switch net-1 to router1
- `ovn-nbctl lsp-add net-1 net1-router1`
- `ovn-nbctl lsp-set-addresses net1-router1 00:00:00:00:0f:01`
- `ovn-nbctl lsp-set-type net1-router1 router`
- `ovn-nbctl lsp-set-options net1-router1 router-port=router1-net1`

### Attach logical switch net-2 to router1
- `ovn-nbctl lrp-add router1 router1-net2 00:00:00:00:0f:02 172.0.0.1/24`
- `ovn-nbctl lsp-add net-2 net2-router1`
- `ovn-nbctl lsp-set-addresses net2-router1 00:00:00:00:0f:02`
- `ovn-nbctl lsp-set-type net2-router1 router`
- `ovn-nbctl lsp-set-options net2-router1 router-port=router1-net2`

### Create dhcp server
- `ovn-nbctl create dhcp_options cidr=10.0.0.0/24 options='"lease_time"="3600" "router"="10.0.0.1" "server_id"="10.0.0.1" "server_mac"="00:00:00:00:0f:01"'` ## Save the UUID 1
- `ovn-nbctl create dhcp_options cidr=172.0.0.0/24 options='"lease_time"="3600" "router"="172.0.0.1" "server_id"="172.0.0.1" "server_mac"="00:00:00:00:0f:02"'` ## Save the UUID 2

- `ovn-nbctl lsp-set-dhcpv4-options vm1 $CIDR_UUID_1`
- `ovn-nbctl lsp-set-dhcpv4-options vm2 $CIDR_UUID_2`

### Create network namespace on compute 1
- `ip link add vm1-peer type veth peer name vm1`
- `ovs-vsctl add-port br-int vm1-peer`
- `ovs-vsctl set interface vm1-peer external_ids:iface-id=vm1`
- `ip link set vm1-peer up`
- `ip netns add vm1-ns`
- `ip link set vm1 netns vm1-ns`
- `ip netns exec vm1-ns ip link set dev vm1 address 00:00:00:00:01:01`
- `ip netns exec vm1-ns ip link set vm1 up`
- `ip netns exec vm1-ns dhclient vm1`

### Create network namespace on compute 2
- `ip link add vm2-peer type veth peer name vm2`
- `ovs-vsctl add-port br-int vm2-peer`
- `ovs-vsctl set interface vm2-peer external_ids:iface-id=vm2`
- `ip link set vm2-peer up`
- `ip netns add vm2-ns`
- `ip link set vm2 netns vm2-ns`
- `ip netns exec vm2-ns ip link set dev vm2 address 00:00:00:00:02:01`
- `ip netns exec vm2-ns ip link set vm2 up`
- `ip netns exec vm2-ns dhclient vm2`


### Create bridge mapping for enp7s0 map network name "phyNet" to enp7s0 (run on 'controller1/host1')
- `ovs-vsctl set open_vswitch . external-ids:ovn-bridge-mappings=phyNet:br-ex`
- `ovs-vsctl add-br br-ex`
- `ovs-vsctl add-port br-ex enp7s0`
- `ifconfig enp7s0 up`


### Create a localnet port
- `ovn-nbctl ls-add public`
- `ovn-nbctl lsp-add public ln-public`
- `ovn-nbctl lsp-set-type ln-public localnet`
- `ovn-nbctl lsp-set-addresses ln-public unknown`
- `ovn-nbctl lsp-set-options ln-public network_name=phyNet`

### Attach logical switch public to router1
- `ovn-nbctl lrp-add router1 router1-public 00:00:00:00:00:fe 192.168.100.254/24`
- `ovn-nbctl lsp-add public public-router1`
- `ovn-nbctl lsp-set-type public-router1 router`
- `ovn-nbctl lsp-set-addresses public-router1 router`
- `ovn-nbctl lsp-set-options public-router1 router-port=router1-public`

### Bind port on host1
- `ovn-nbctl lrp-set-gateway-chassis router1-public host1 20`
- `ovn-sbctl show`

### Create fip
- `ovn-nbctl lr-nat-add router1 dnat_and_snat 192.168.100.160 10.0.0.10`
- `ovn-nbctl lr-nat-add router1 dnat_and_snat 192.168.100.170 172.0.0.10`