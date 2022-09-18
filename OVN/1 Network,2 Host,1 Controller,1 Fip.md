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

## Set remote sb&nb
- `ovn-sbctl set-connection ptcp:6642`
- `ovn-nbctl set-connection ptcp:6641`
- `netstat -lntp | grep 664`

## In controller, create logical switch and ports Set openvswitch on to sb&nb: 
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.7 external_ids:ovn-encap-type="geneve" external_ids:system-id="host1"`
- `ovn-nbctl ls-add sw`
- `ovn-nbctl lsp-add sw sp1`
- `ovn-nbctl lsp-set-addresses sp1 "00:00:00:00:00:01 10.0.0.1"`
- `ovn-nbctl lsp-add sw sp2`
- `ovn-nbctl lsp-set-addresses sp2 "00:00:00:00:00:02 10.0.0.2"`
- `ovn-nbctl show`

## In compute node 1, make ovn-controller connect to southbound db, and create test
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.33 external_ids:ovn-encap-type="geneve" external_ids:system-id="host2"`
- `ip link add sp1_l type veth peer name sp1_r`
- `ovs-vsctl add-port br-int sp1_l  #sudo ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure`
- `ovs-vsctl set interface sp1_l external_ids:iface-id=sp1`
- `ip link set sp1_l up`
- `ip netns add sp1`
- `ip link set sp1_r netns sp1`
- `ip netns exec sp1 ip link set sp1_r up`
- `ip netns exec sp1 ip addr add 10.0.0.1/24 dev sp1_r`
- `ip netns exec sp1 ip link set dev sp1_r address 00:00:00:00:00:01`

## or run the following command in compute node 2 machine instead
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.174 external_ids:ovn-encap-type="geneve" external_ids:system-id="host3"`
- `ip link add sp2_l type veth peer name sp2_r`
- `ovs-vsctl add-port br-int sp2_l`
- `ovs-vsctl set interface sp2_l external_ids:iface-id=sp2`
- `ip link set sp2_l up`
- `ip netns add sp2`
- `ip link set sp2_r netns sp2`
- `ip netns exec sp2 ip link set sp2_r up`
- `ip netns exec sp2 ip addr add 10.0.0.2/24 dev sp2_r`
- `ip netns exec sp2 ip link set dev sp2_r address 00:00:00:00:00:02`

## test
```
sudo ip netns exec sp1 ping 10.0.0.2 -c 1
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.318 ms
```
