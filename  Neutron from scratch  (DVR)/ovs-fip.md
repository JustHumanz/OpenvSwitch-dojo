# OVS FIP

#### create ex br
- `ovs-vsctl add-br br-ex`

#### add nic fip to Ovs
- `ovs-vsctl add-port br-ex enp7s0`

#### create int-br
- `ovs-vsctl add-br br-int`

#### create port for patch type 
- `ovs-vsctl add-port br-int vport-br-int -- set interface vport-br-int type=internal`
- `ovs-vsctl add-port br-ex vport-br-ex -- set interface vport-br-ex type=internal`
- `ovs-vsctl set interface vport-br-int type=patch options:peer=vport-br-ex`
- `ovs-vsctl set interface vport-br-ex type=patch options:peer=vport-br-int`

#### create fip & router port in int-br
- `ovs-vsctl add-port br-int vport-fip -- set interface vport-fip type=internal`

- `ovs-vsctl add-port br-int vport-router -- set interface vport-router type=internal`

#### create fip & router ns
- `ip netns add fip-ns`
- `ip netns add router-ns`

#### add vport fip&router to ns
- `ip link set vport-fip netns fip-ns`
- `ip link set vport-router netns router-ns`

#### add ip in fip-ns
- `ip netns exec fip-ns ip add add 192.168.100.254/24 dev vport-fip`
- `ip netns exec fip-ns ip link set vport-fip up`
- `ip netns exec fip-ns ip route add default via 192.168.100.1 dev vport-fip`

#### add link from fip to router
- `ip link add fpr netns fip-ns type veth peer name rfp netns router-ns`

#### enable arp proxy
- `ip netns exec fip-ns sysctl net.ipv4.conf.vport-fip.proxy_arp=1`

#### add ip in router-ns
- `ip netns exec router-ns ip add add 172.16.18.1/24 dev vport-router`
- `ip netns exec router-ns ip link set vport-router up`
- `ip netns exec router-ns sysctl -w net.ipv4.ip_forward=1`

#### add ip in rfp
- `ip netns exec router-ns ip add add 169.254.31.238/31 dev rfp`
- `ip netns exec router-ns ip link set rfp up`

#### add ip in fpr
- `ip netns exec fip-ns ip add add 169.254.31.239/31 dev fpr`
- `ip netns exec fip-ns ip link set fpr up`
- `ip netns exec fip-ns ip route add 192.168.100.150 via 169.254.31.238 dev fpr`

#### set ip route router-ns
- `ip netns exec router-ns ip route add 172.16.18.100 via 169.254.1.10 dev fpr`
- `ip netns exec router-ns ip route add default via 169.254.31.239 dev rfp`

#### set nat firewall
- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.150/32 -j DNAT --to-destination 172.16.18.100`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.18.100/32 -j SNAT --to-source 192.168.100.150`

### VMS
```
virt-install --import --name cirros-vm --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=52:54:00:1b:1e:f7 --check all=off
```

- `virsh console cirros-vm`
- `ip add add 172.16.18.100/24 dev eth0`
- `ip link set eth0 up`
- `ip route add default via 172.16.18.1`



# Ref:
- http://blog.gampel.net/2014/12/openstack-neutron-distributed-virtual.html
- http://blog.gampel.net/2014/12/openstack-dvr2-floating-ips.html
- http://blog.gampel.net/2015/01/openstack-DVR-SNAT.html
- https://assafmuller.com/2015/04/15/distributed-virtual-routing-floating-ips/
- https://blog.scottlowe.org/2012/11/27/connecting-ovs-bridges-with-patch-ports/