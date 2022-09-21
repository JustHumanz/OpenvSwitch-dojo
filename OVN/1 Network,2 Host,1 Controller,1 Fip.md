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

## In controller, create logical switch and ports Set openvswitch on to sb&nb: 
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.7 external_ids:ovn-encap-type="geneve" external_ids:system-id="host1"`
- `ovn-nbctl ls-add sw`
- `ovn-nbctl lsp-add sw vm1`
- `ovn-nbctl lsp-set-addresses vm1 "00:00:00:00:02:01 10.0.0.1"`
- `ovn-nbctl lsp-add sw vm2`
- `ovn-nbctl lsp-set-addresses vm2 "00:00:00:00:03:01 10.0.0.2"`
- `ovn-nbctl show`

## In compute node 1, make ovn-controller connect to southbound db, and create test
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.33 external_ids:ovn-encap-type="geneve" external_ids:system-id="host2"`
- `ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure`
### VM
```
virt-install --import --name cirros-vm-1 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:02:01 --check all=off

ip addr add 10.0.0.1/24 dev eth0
```
- `ovs-vsctl set interface vnet0 external_ids:iface-id=vm1`

## or run the following command in compute node 2 machine instead
- `ovs-vsctl set open_vswitch . external_ids:ovn-remote="tcp:192.168.122.7:6642" external_ids:ovn-encap-ip=192.168.122.174 external_ids:ovn-encap-type="geneve" external_ids:system-id="host3"`
- `ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure`
### VM
```
virt-install --import --name cirros-vm-1 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:03:01 --check all=off

ip addr add 10.0.0.2/24 dev eth0
```
- `ovs-vsctl set interface vnet0 external_ids:iface-id=vm2`

## test
```
ping 10.0.0.2 
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.318 ms
```

