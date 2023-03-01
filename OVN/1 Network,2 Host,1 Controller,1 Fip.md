# Topology

## 1 Network,2 Host,1 Controller
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
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.7 external_ids:ovn-encap-type="geneve" external_ids:system-id="host1"`

## Compute 1
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.33 external_ids:ovn-encap-type="geneve" external_ids:system-id="host2"`

## Compute 2
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.174 external_ids:ovn-encap-type="geneve" external_ids:system-id="host3"`

## In controller, create logical switch  
- `ovn-nbctl ls-add sw`
- `ovn-nbctl set logical_switch sw other_config:subnet="10.0.0.0/24" other_config:exclude_ips="10.0.0.1"`
- `ovn-nbctl lsp-add sw vm1`
- `ovn-nbctl lsp-set-addresses vm1 "00:00:00:00:02:01 10.0.0.10"`
- `ovn-nbctl lsp-add sw vm2`
- `ovn-nbctl lsp-set-addresses vm2 "00:00:00:00:03:01 10.0.0.20"`
- `ovn-nbctl show`

## Set dhcp
- `ovn-nbctl create dhcp_options cidr=10.0.0.0/24 options='"lease_time"="3600" "router"="10.0.0.1" "server_id"="10.0.0.1" "server_mac"="00:00:00:00:0f:01"'` ## Save the UUID
- `ovn-nbctl lsp-set-dhcpv4-options vm1 $CIDR_UUID`
- `ovn-nbctl lsp-set-dhcpv4-options vm2 $CIDR_UUID`

### In Compute 1, create network linux namespaces vm1
- `ip link add vm1-peer type veth peer name vm1`
- `ovs-vsctl add-port br-int vm1-peer`
- `ovs-vsctl set interface vm1-peer external_ids:iface-id=vm1`
- `ip link set vm1-peer up`
- `ip netns add vm1-ns`
- `ip link set vm1 netns vm1-ns`
- `ip netns exec vm1-ns ip link set dev vm1 address 00:00:00:00:02:01`
- `ip netns exec vm1-ns ip link set vm1 up`
- `ip netns exec vm1-ns dhclient vm1`

### In Compute 1, create network linux namespaces vm1
- `ip link add vm2-peer type veth peer name vm2`
- `ovs-vsctl add-port br-int vm2-peer`
- `ovs-vsctl set interface vm2-peer external_ids:iface-id=vm2`
- `ip link set vm2-peer up`
- `ip netns add vm2-ns`
- `ip link set vm2 netns vm2-ns`
- `ip netns exec vm2-ns ip link set dev vm2 address 00:00:00:00:03:01`
- `ip netns exec vm2-ns ip link set vm2 up`
- `ip netns exec vm2-ns dhclient vm2`

## test
```
ip netns exec vm1-ns ping 10.0.0.20 
PING 10.0.0.20 (10.0.0.20) 56(84) bytes of data.
64 bytes from 10.0.0.20: icmp_seq=1 ttl=64 time=0.318 ms
```

