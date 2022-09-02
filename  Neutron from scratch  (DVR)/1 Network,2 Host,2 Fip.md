# Topology

## 1 Network,2 Host,2 Fip
```
                                                                                                                                                           xx x   xx
                                                                                                                                                          x         x
                                                                                                                                                          x         xxx   xx
                                                Compute Host 1                                                                                           x      xxx  xx     xx
                      ┌───────────────────────────────────────────────────────────────────────────────┐                                              xxxxx                   x
                      │                                                                               │                                              x                      xx
                      │  ┌────────────────────────┐                         ┌─────────────────┐       │                                               x                         xx
                      │  │                        │                         │                 │       │                                              xxx                          x
                      │  │             router-ns  │                         │                 │       │                                          xx x                              x
                      │  │                        │169.254.31.238           │                 │       │                                         xx                                 x
                      │  │                        ├─────────────────────────┤                 │       │                                         xx           xx              xxxxxx
                      │  │ 172.16.18.1            │           169.254.31.239│    fip-ns       │       │                                           x      x    x       xx      x
                      │  └─────────────────────┬──┘                         │                 │       │                                                       xx      xxxx
                      │                      ▲ │                            │                 │       │                                                         xxxxxx   xx    x
                      │                      │ │                            │ 192.168.100.254 │       │                                                                    xxxxx
                      │                      │ │                            └─────────────────┘       │                                                        Internet
                      │  ┌───────────────────┼─┴──────┐                            ▲                  │                                                           │
                      │  │                   │        │                            │                  │                                                           │
                      │  │    BR-INT (OVS)   │        │◄───────────────────────────┘                  │                                                           │
                      │  │                   │        │                                               │                                                           │
                      │  │                   │        │◄──────────┐                                   │                                                           │
                      │  └───────────────────┼────────┘           │                                   │                                                           │
                      │          ▲           │                    │                                   │                                                           │
                      │          │           │                    │                                   │                                                           │
                      │          │           │                    │                                   │                                                           │
                      │          │           │                    │                                   │                                                           │
                      │          ▼           │                    │                                   │                                                           │
                      │   ┌────────────────┐ │                    │                                   │                                                           │
                      │   │                │ │                    │                                   │                                                           │
                      │   │br-host-1(OVS)  │ │                    │                                   │                                                           │
                      │   │                │ │                    │                                   │                                                           │
                      │   └────────────────┘ │                    │                                   │                                                           │
                      │         ▲            │                    │                                   │                                                           │
                      │         │            │                    │                                   │                                                           │
                      │         │            │                    │                                   │                                                           │
                      │         │            │                    │                                   │                                                           │
                      │         │            │                    │                                   │                                                           │
                      │         │            │                    │         ┌──────────────┐          │                                                           │
                      │         │            │                    │         │              │          │                                                           │
                      │         │            │                    │         │              │          │                                                           │
               ┌──────┤         │            ▼                    │         │              │          ├─────┐                                                     │
               │      │         │    ┌─────────────────┐          └────────►│              │          │     │                                                     │
               │      │         │    │  172.16.18.100  │                    │ BR-EX(OVS)   │          │     │                  192.168.100.0/24                   │
   ┌───────────┤      │◄────────┘    │  192.168.100.150│                    │              ├──────────┼─────┼─────────────────────────────────────────────────────┤
   │           │      │              │                 │                    │              │          │     │                                                     │
   │           └──────┤              │      VM         │                    │              │          ├─────┘                                                     │
   │            enp1s0│              └─────────────────┘                    │              │          │ enp7s0                                                    │
   │                  │                                                     └──────────────┘          │                                                           │
   │192.168.122.6     │                                                                               │                                                           │
   │                  │                                                                               │                                                           │
   │                  └───────────────────────────────────────────────────────────────────────────────┘                                                           │
   │                                                                                                                                                              │
   │                                                                                                                                                              │
   │                                                                                                                                                              │
   │                                                                                                                                                              │
   │                                                                                                                                                              │
   │                                                                                                                                                              │
   │                                                                                                                                                              │
   │                                          Compute Host 2                                                                                                      │
   │                ┌─────────────────────────────────────────────────────────────────────────────────┐                                                           │
   │                │                                                                                 │                                                           │
   │                │                                                                                 │                                                           │
   │                │                                                                                 │                                                           │
   │                │                                                                                 │                                                           │
   │                │                                                                                 │                                                           │
   │                │  ┌──────────────────────────────┐                                               │                                                           │
   │                │  │                              │                                               │                                                           │
   │                │  │                              │                                               │                                                           │
   │                │  │     BR-INT (OVS)             │vport-br-in                                    │                                                           │
   │                │  │                              ├───────────┐                                   │                                                           │
   │                │  │                              │           │                                   │                                                           │
   │                │  └──────────────────────────────┘           │                                   │                                                           │
   │                │          ▲            ▲                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          ▼            │                     │                                   │                                                           │
   │                │   ┌────────────────┐  │                     │                                   │                                                           │
   │                │   │                │  │                     │ peer type                         │                                                           │
   │                │   │                │  │                     │                                   │                                                           │
   │                │   │ br-host-2(OVS) │  │                     │                                   │                                                           │
   │                │   │                │  │                     │                                   │                                                           │
   │                │   └────────────────┘  │                     │                                   │                                                           │
   │                │          ▲            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │                                   │                                                           │
   │                │          │            │                     │       ┌──────────────┐            │                                                           │
   │                │          │            │                     │       │              │            │                                                           │
   │192.168.122.33  │          │            │                     │       │              │            │                                                           │
   │         ┌──────┤          │            ▼                     │       │              │            ├─────┐                                                     │
   │         │      │          │        ┌─────────────────┐       └───────┤              │            │     │                                                     │
   └─────────┤      │ ◄────────┘        │  172.16.18.100  │    port-br-ex │ BR-EX(OVS)   │            │     │                 192.168.100.0/24                    │
             │      │                   │  192.168.100.150│               │              ├────────────┼─────┼─────────────────────────────────────────────────────┘
             │      │                   │                 │               │              │            │     │
             └──────┤                   │      VM         │               │              │            ├─────┘
              enp1s0│                   └─────────────────┘               │              │            │ enp7s0
                    │                                                     └──────────────┘            │
                    │                                                                                 │
                    │                                                                                 │
                    └─────────────────────────────────────────────────────────────────────────────────┘
```  
# Walkthrough


## Host 1

#### create br host
- `ovs-vsctl add-br br-tun`
- `ovs-vsctl add-port br-tun vxlan1 -- set interface vxlan1 type=vxlan options:remote_ip=192.168.122.33 options:local_ip=192.168.122.6 options:egress_pkt_mark=0 options:df_default=true out_key=flow options:in_key=flow`

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

- `ovs-vsctl add-port br-int int-tun tag=10 -- set interface int-tun type=patch options:peer=tun-int`
- `ovs-vsctl add-port br-tun tun-int -- set interface tun-int type=patch options:peer=int-tun`

#### create fip & router port in int-br
- `ovs-vsctl add-port br-int vport-fip tag=3 -- set interface vport-fip type=internal`
- `ovs-vsctl add-port br-int vport-router tag=10 -- set interface vport-router type=internal`

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
- `ip netns exec fip-ns ip route add 192.168.100.160 via 169.254.31.238 dev fpr`

#### set ip route router-ns
- `ip netns exec router-ns ip route add default via 169.254.31.239 dev rfp`

#### set nat firewall
- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.150/32 -j DNAT --to-destination 172.16.18.100`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.18.100/32 -j SNAT --to-source 192.168.100.150`

- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.160/32 -j DNAT --to-destination 172.16.18.200`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.18.200/32 -j SNAT --to-source 192.168.100.160`

#### set openflow 
- `ovs-ofctl add-flow br-tun "table=0,priority=1,in_port=3 actions=resubmit(,1)"` #Make sure port 3 is vxlan interface
- `ovs-ofctl add-flow br-tun "table=0,priority=1,in_port=1 actions=resubmit(,4)"` #Make sure port 1 is tun-int interface
- `ovs-ofctl add-flow br-tun "table=0,priority=0 actions=drop"`
- `ovs-ofctl add-flow br-tun "table=1,priority=3,arp,dl_vlan=10,arp_tpa=172.16.18.100 actions=drop"`
- `ovs-ofctl add-flow br-tun "table=1,priority=2,dl_vlan=10,dl_dst=00:00:00:00:01:01 actions=drop"`
- `ovs-ofctl add-flow br-tun "table=1,priority=0 actions=resubmit(,2)"`
- `ovs-ofctl add-flow br-tun "table=2,priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)"`
- `ovs-ofctl add-flow br-tun "table=2,priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)"`
- `ovs-ofctl add-flow br-tun "table=4,priority=1,tunnel_id=0x2 actions=mod_vlan_vid:10,resubmit(,9)"`
- `ovs-ofctl add-flow br-tun "table=4,priority=1,tunnel_id=0x3 actions=mod_vlan_vid:20,resubmit(,9)"`
- `ovs-ofctl add-flow br-tun "table=4,priority=0 actions=drop"`
- `ovs-ofctl add-flow br-tun "table=9,priority=1,dl_src=00:00:00:00:02:01 actions=output:3"`
- `ovs-ofctl add-flow br-tun "table=9,priority=0 actions=resubmit(,10)"`
- `ovs-ofctl add-flow br-tun "table=10,priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x0,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:OXM_OF_IN_PORT[]),output:3"`
- `ovs-ofctl add-flow br-tun "table=20,priority=2,dl_vlan=10,dl_dst=00:00:00:00:02:01 actions=strip_vlan,load:0x2->NXM_NX_TUN_ID[],output:1"`
- `ovs-ofctl add-flow br-tun "table=20,priority=0 actions=resubmit(,22)"`
- `ovs-ofctl add-flow br-tun "table=22,priority=1,dl_vlan=10 actions=strip_vlan,load:0x2->NXM_NX_TUN_ID[],output:1"`
- `ovs-ofctl add-flow br-tun "table=22,priority=0 actions=drop"`

### VM
```
virt-install --import --name cirros-vm --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:01:01 --check all=off
```

- `ovs-vsctl set port vnet0 tag=10`
- `virsh console cirros-vm`
- `ip add add 172.16.18.100/24 dev eth0`
- `ip link set eth0 up`
- `ip route add default via 172.16.18.1`

## Host 2

#### create br host
- `ovs-vsctl add-br br-tun`
- `ovs-vsctl add-port br-tun vxlan1 -- set interface vxlan1 type=vxlan options:remote_ip=192.168.122.6 options:local_ip=192.168.122.33 options:egress_pkt_mark=0 options:df_default=true out_key=flow options:in_key=flow`

#### create ex br
- `ovs-vsctl add-br br-ex`
- `ifconfig enp7s0 up`

#### add nic fip to Ovs
- `ovs-vsctl add-port br-ex enp7s0`

#### create int-br
- `ovs-vsctl add-br br-int`

#### create port for patch type 
- `ovs-vsctl add-port br-int int-ex -- set interface int-ex type=patch options:peer=ex-int`
- `ovs-vsctl add-port br-ex ex-int -- set interface ex-int type=patch options:peer=int-ex`

- `ovs-vsctl add-port br-int int-tun tag=10 -- set interface int-tun type=patch options:peer=tun-int`
- `ovs-vsctl add-port br-tun tun-int -- set interface tun-int type=patch options:peer=int-tun`

#### set openflow 

- `ovs-ofctl add-flow br-tun "table=0,priority=1,in_port=3 actions=resubmit(,1)"` #Make sure port 3 is vxlan interface
- `ovs-ofctl add-flow br-tun "table=0,priority=1,in_port=1 actions=resubmit(,4)"` #Make sure port 1 is tun-int interface
- `ovs-ofctl add-flow br-tun "table=0,priority=0 actions=drop"`
- `ovs-ofctl add-flow br-tun "table=1,priority=3,arp,dl_vlan=10,arp_tpa=172.16.18.200 actions=dro`
- `ovs-ofctl add-flow br-tun "table=1,priority=2,dl_vlan=10,dl_dst=00:00:00:00:02:01 actions=drop`
- `ovs-ofctl add-flow br-tun "table=1,priority=0 actions=resubmit(,2)"  `
- `ovs-ofctl add-flow br-tun "table=2,priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20`
- `ovs-ofctl add-flow br-tun "table=2,priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22`
- `ovs-ofctl add-flow br-tun "table=4,priority=1,tunnel_id=0x2 actions=mod_vlan_vid:10,resubmit(,9)"  `
- `ovs-ofctl add-flow br-tun "table=4,priority=0 actions=drop"`
- `ovs-ofctl add-flow br-tun "table=9,priority=1,dl_src=00:00:00:00:01:01 actions=output:3" `
- `ovs-ofctl add-flow br-tun "table=9,priority=0 actions=resubmit(,10)" `
- `ovs-ofctl add-flow br-tun "table=10,priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x0,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:OXM_OF_IN_PORT[]),output:3"`
- `ovs-ofctl add-flow br-tun "table=20,priority=2,dl_vlan=10,dl_dst=00:00:00:00:01:01 actions=strip_vlan,load:0x2->NXM_NX_TUN_ID[],output:1"  `
- `ovs-ofctl add-flow br-tun "table=20,priority=0 actions=resubmit(,22)"`
- `ovs-ofctl add-flow br-tun "table=22,priority=1,dl_vlan=10 actions=strip_vlan,load:0x2->NXM_NX_TUN_ID[],output:1"  `
- `ovs-ofctl add-flow br-tun "table=22,priority=0 actions=drop`

### VM
```
virt-install --import --name cirros-vm --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:02:01 --check all=off
```

- `ovs-vsctl set port vnet0 tag=10`
- `virsh console cirros-vm`
- `ip add add 172.16.18.200/24 dev eth0`
- `ip link set eth0 up`
- `ip route add default via 172.16.18.1`

