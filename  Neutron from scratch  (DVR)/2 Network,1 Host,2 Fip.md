# Topology

## 2 Network,1 Host,2 Fip
```
                                                                                                                                    xxx x   xx
                                                                                                                                    x         x
                                                                                                                                    x         xxx   xx
                        Compute Host 1                                                                                           x      xxx  xx     xx
┌─────────────────────────────────────────────────────────────────────────────────┐                                              xxxxx                   x
│                                                                                 │                                              x                      xx
│  ┌──────────────────────────┐                         ┌─────────────────┐       │                                               x                         xx
│  │                          │                         │                 │       │                                              xxx                          x
│  │                          │                         │                 │       │                                          xx x                              x
│  │        router-ns         │169.254.31.238(rfp)      │                 │       │                                         xx                                 x
│  │                          ├─────────────────────────┤                 │       │                                         xx           xx              xxxxxx
│  │ 172.16.18.1  172.16.19.1 │      169.254.31.239(fpr)│    fip-ns       │       │                                           x      x    x       xx      x
│  └───────────────────────┬──┘                         │                 │       │                                                       xx      xxxx
│    ▲                  ▲  │                            │                 │       │                                                         xxxxxx   xx    x
│    │                  │  │vport-router                │ 192.168.100.254 │       │                                                                    xxxxx
│    │                  │  │                            └──────┬──────────┘       │                                                        Internet
│  ┌─┼──────────────────┼──┴──────┐                            ▲                  │                                                           │
│  │ │                  │         │                            │                  │                                                           │
│  │ │                  │         │                            │                  │                                                           │
│  │ │                  │         │                            │                  │                                                           │
│  │ │                  │         │                            │                  │                                                           │
│  │ │                  │         │                            │                  │                                                           │
│  │ │                  │         │            vport-fip       │                  │                                                           │
│  │ │                  │         │◄───────────────────────────┘                  │                                                           │
│  │ │                  │         │                                               │                                                           │
│  │ │   BR-INT (OVS)   │         │                                               │                                                           │
│  │ │                  │         │vport-br-int                                   │                                                           │
│  │ │                  │         │◄──────────┐                                   │                                                           │
│  │ │                  │         │           │                                   │                                                           │
│  │ │                  │         │           │                                   │                                                           │
│  │ │                  │         │           │                                   │                                                           │
│  │ │                  │         │           │                                   │                                                           │
│  │ │                  │         │           │                                   │                                                           │
│  │ │                  │         │           │ peer type                         │                                                           │
│  │ │                  │         │           │                                   │                                                           │
│  │ │                  │         │           │                                   │                                                           │
│  └─┼──────────────────┼─────────┘           │                                   │                                                           │
│    │                  │                     │                                   │                                                           │
│    │                  │                     │       ┌──────────────┐            │                                                           │
│    │                  │                     │       │              │            │                                                           │
│    │                  │                     │       │              │            │                                                           │
│    ▼                  ▼                     │       │              │            ├─────┐                                                     │
│ ┌─────────────────┐ ┌─────────────────┐     └──────►│              │            │     │                                                     │
│ │  172.16.18.100  │ │  172.16.19.100  │ vport-br-ex │    BR-EX     │            │     │              192.168.100.0/24                       │
│ │  192.168.100.150│ │  192.168.100.160│             │              ├────────────┼─────┼─────────────────────────────────────────────────────┘
│ │                 │ │                 │             │              │            │     │
│ │      VM         │ │      VM         │             │              │            ├─────┘
│ │                 │ │                 │             │              │            │ enp7s0
│ │                 │ │                 │             └──────────────┘            │
│ └─────────────────┘ └─────────────────┘                                         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```  

## Walkthrough


#### create ex br
- `ovs-vsctl add-br br-ex`

#### add nic fip to Ovs
- `ovs-vsctl add-port br-ex enp7s0`
- `ifconfig enp7s0 up`

#### create int-br
- `ovs-vsctl add-br br-int`

#### create port for patch type 
- `ovs-vsctl add-port br-int int-ex tag=3 -- set interface int-ex type=patch options:peer=ex-int`
- `ovs-vsctl add-port br-ex ex-int -- set interface ex-int type=patch options:peer=int-ex`

#### create fip & router port in int-br
- `ovs-vsctl add-port br-int v-fip tag=3 -- set interface v-fip type=internal`
- `ovs-vsctl add-port br-int v-router_1 tag=10 -- set interface v-router_1 type=internal`
- `ovs-vsctl add-port br-int v-router_2 tag=20 -- set interface v-router_2 type=internal`

#### create fip & router ns
- `ip netns add fip-ns`
- `ip netns add router-ns`

#### add vport fip&router to ns
- `ip link set v-fip netns fip-ns`
- `ip link set v-router_1 netns router-ns`
- `ip link set v-router_2 netns router-ns`

#### add ip in fip-ns
- `ip netns exec fip-ns ip add add 192.168.100.254/24 dev v-fip`
- `ip netns exec fip-ns ip link set v-fip up`
- `ip netns exec fip-ns ip route add default via 192.168.100.1 dev v-fip`

#### add link from fip to router
- `ip link add fpr netns fip-ns type veth peer name rfp netns router-ns`

#### enable arp proxy
- `ip netns exec fip-ns sysctl net.ipv4.conf.v-fip.proxy_arp=1`

#### add ip in router-ns
- `ip netns exec router-ns ip add add 172.16.18.1/24 dev v-router_1`
- `ip netns exec router-ns ip add add 172.16.19.1/24 dev v-router_2`
- `ip netns exec router-ns ip link set v-router_1 up`
- `ip netns exec router-ns ip link set v-router_2 up`
- `ip netns exec router-ns sysctl -w net.ipv4.ip_forward=1`

#### add ip in rfp
- `ip netns exec router-ns ip add add 169.254.31.238/31 dev rfp`
- `ip netns exec router-ns ip link set rfp up`

#### add ip in fpr
- `ip netns exec fip-ns ip add add 169.254.31.239/31 dev fpr`
- `ip netns exec fip-ns ip link set fpr up`
- `ip netns exec fip-ns ip route add 192.168.100.150 via 169.254.31.238 dev fpr`
- `ip netns exec fip-ns ip route add 192.168.100.160 via 169.254.31.238 dev fpr`

#### set ip route router-ns
- ~~`ip netns exec router-ns ip route add 172.16.18.100 via 169.254.31.239 dev rfp`~~
- `ip netns exec router-ns ip route add default via 169.254.31.239 dev rfp`

#### set nat firewall
- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.150/32 -j DNAT --to-destination 172.16.18.100`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.18.100/32 -j SNAT --to-source 192.168.100.150`
- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.160/32 -j DNAT --to-destination 172.16.19.100`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.19.100/32 -j SNAT --to-source 192.168.100.160`

### VMS
```
virt-install --import --name cirros-vm-1 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --check all=off
```
- `ovs-vsctl set port vnet0 tag=10`
- `virsh console cirros-vm-1`
- `ip add add 172.16.18.100/24 dev eth0`
- `ip link set eth0 up`
- `ip route add default via 172.16.18.1`

```
virt-install --import --name cirros-vm-2 --memory 256 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk-clone.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --check all=off
```
- `ovs-vsctl set port vnet1 tag=20`
- `virsh console cirros-vm-2`
- `ip add add 172.16.19.100/24 dev eth0`
- `ip link set eth0 up`
- `ip route add default via 172.16.19.1`