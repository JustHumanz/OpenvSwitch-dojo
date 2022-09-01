# Topology

## 2 Network,2 Host,4 Fip
```
                                                                                                                                                       xxx x   xx
                                                                                                                                                       x         x
                                                                                                                                                       x         xxx   xx
                                             Compute Host 1                                                                                           x      xxx  xx     xx
                   ┌───────────────────────────────────────────────────────────────────────────────┐                                              xxxxx                   x
                   │                                   (rfp)                                       │                                              x                      xxxx
                   │  ┌───────────────────────────┐169.254.31.238               ┌────────────────┐ │                                               x                         x
                   │  │                           ├─────────────────────────────┤                │ │                                              xxx                         x
                   │  │                           │               169.254.31.239│    fip-ns      │ │                                          xx x                            x
                   │  │                           ├─────────────────────┐(fpr)  │                │ │                                         xx                            xxx
                   │  │         router-ns         │                     │       │ 192.168.100.254│ │                                         xx           xx              xxx
                   │  │172.16.18.1     172.16.19.1├──────┐              │       └─────────┬──────┘ │                                           x      x    x       xx      x
                   │  └─────┬───────┬─────┬───┬───┘      │              │                 │        │                                                       xx      xxxx
                   │        │       │     │   │          │              │                 │        │                                                         xxxxxx   xx    x
                   │        │       │     │   │        ┌─▼─────────┐ ┌──▼────────┐        │        │                                                                    xxxxx
                   │        │       │     │   │        │172.16.18.2│ │172.16.19.2│        │        │                                                        Internet
                   │  ┌─────┴───────┼─────┼───┴─┐      │dhcp-ns    │ │dhcp-ns    │        │        │                                                           │
                   │  │             │     │     │      └───────────┘ └───────────┘        │        │                                                           │
                   │  │             │     │     │                                         │        │                                                           │
                   │  │ br-int(OVS) │     │     ├─────────────────────────────────────────┘        │                                                           │
                   │  │             │     │     │  vport-br-in                                     │                                                           │
                   │  │             │     │     │◄─────────────┐                                   │                                                           │
                   │  └────┬────────┼─────┼─────┘              │                                   │                                                           │
                   │       │        │     │                    │                                   │                                                           │
                   │       │        │     │                    │                                   │                                                           │
                   │       │        │     │                    │                                   │                                                           │
                   │       │        │     │                    │                                   │                                                           │
                   │ ┌─────┴──────┐ │     │                    │                                   │                                                           │
                   │ │            │ │     │                    │                                   │                                                           │
                   │ │br-tun0(OVS)│ │     │                    │                                   │                                                           │
                   │ │            │ │     │                    │                                   │                                                           │
                   │ └─────┬──────┘ │     │                    │                                   │                                                           │
                   │       │        │     │                    │                                   │                                                           │
                   │       │        │     │                    │peer type                          │                                                           │
                   │       │        │     ▼                    │                                   │                                                           │
                   │       │        │  ┌───────────────┐       │                                   │                                                           │
                   │       │        │  │ 172.16.19.100 │       │                                   │                                                           │
                   │       │        │  │192.168.100.160│       │                                   │                                                           │
                   │       │        │  │      VM       │       │          ┌──────────────┐         │                                                           │
                   │       │        │  └───────────────┘       │          │              │         │                                                           │
                   │       │        │                          │          │              │         │                                                           │
            ┌──────┤       │        ▼                          │          │              │         ├─────┐                                                     │
            │      │       │   ┌───────────────┐               │          │              │         │     │                                                     │
            │      │       │   │ 172.16.18.100 │               │          │  br-ex(OVS)  │         │     │                  192.168.100.0/24                   │
┌───────────┤      ├───────┘   │192.168.100.150│               └─────────►│              ├─────────┼─────┼─────────────────────────────────────────────────────┤
│           │      │           │      VM       │               port-br-ex │              │         │     │                                                     │
│           └──────┤           └───────────────┘                          │              │         ├─────┘                                                     │
│            enp1s0│                                                      │              │         │ enp7s0                                                    │
│                  │                                                      └──────────────┘         │                                                           │
│                  │                                                                               │                                                           │
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
│                │ ┌─────────────────────────────────┐                                             │                                                           │
│                │ │                                 │port-br-in                                   │                                                           │
│                │ │     br-int(OVS)                 ├──────────────────────────┐                  │                                                           │
│                │ │                                 │                          │                  │                                                           │
│                │ │                                 │                          │                  │                                                           │
│                │ └──────┬──────────────────────────┘                          │                  │                                                           │
│                │        │           ▲         ▲                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │peer type         │                                                           │
│                │ ┌──────┴─────┐     │         │                               │                  │                                                           │
│                │ │            │     │         │                               │                  │                                                           │
│                │ │br-tun(OVS) │     │         │                               │                  │                                                           │
│                │ │            │     │         │                               │                  │                                                           │
│                │ └──────┬─────┘     │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         │                               │                  │                                                           │
│                │        │           │         ▼                               │                  │                                                           │
│                │        │           │ ┌────────────────┐                      │                  │                                                           │
│                │        │           │ │ 172.16.19.200  │                      │                  │                                                           │
│                │        │           │ │ 192.168.100.170│                      │                  │                                                           │
│                │        │           │ │     VM         │                      │port-br-ex        │                                                           │
│                │        │           │ └────────────────┘                 ┌────┴───────┐          │                                                           │
│                │        │           │                                    │            │          │                                                           │
│         ┌──────┤        │           ▼                                    │            │          ├─────┐                                                     │
│         │      │        │    ┌────────────────┐                          │            │          │     │                                                     │
└─────────┤      │ ◄──────┘    │ 172.16.18.200  │                          │            │          │     │                 192.168.100.0/24                    │
          │      │             │ 192.168.100.180│                          │ br-ex(OVS) ├──────────┼─────┼─────────────────────────────────────────────────────┘
          │      │             │     VM         │                          │            │          │     │
          └──────┤             └────────────────┘                          │            │          ├─────┘
           enp1s0│                                                         │            │          │ enp7s0
                 │                                                         └────────────┘          │
                 │                                                                                 │
                 │                                                                                 │
                 └─────────────────────────────────────────────────────────────────────────────────┘
```

# Walkthrough

## Host 1

#### create br host
- `ovs-vsctl add-br br-tun`
- `ovs-vsctl add-port br-tun vxlan1 -- set interface vxlan1 type=vxlan options:remote_ip=192.168.122.33`


#### create ex br
- `ovs-vsctl add-br br-ex`

#### add nic fip to Ovs
- `ovs-vsctl add-port br-ex enp7s0`

#### create int-br
- `ovs-vsctl add-br br-int`

#### create port for patch type 
- `ovs-vsctl add-port br-int int-ex tag=3 -- set interface int-ex type=patch options:peer=ex-int`
- `ovs-vsctl add-port br-ex ex-int -- set interface ex-int type=patch options:peer=int-ex`

#### create fip & router port in int-br
- `ovs-vsctl add-port br-int vport-fip tag=3 -- set interface vport-fip type=internal`
- `ovs-vsctl add-port br-int vport-router_1 tag=10 -- set interface vport-router_1 type=internal`
- `ovs-vsctl add-port br-int vport-router_2 tag=20 -- set interface vport-router_2 type=internal`

#### create fip & router ns
- `ip netns add fip-ns`
- `ip netns add router-ns`

#### add vport fip&router to ns
- `ip link set vport-fip netns fip-ns`
- `ip link set vport-router_1 netns router-ns`
- `ip link set vport-router_2 netns router-ns`

#### add ip in fip-ns
- `ip netns exec fip-ns ip add add 192.168.100.254/24 dev vport-fip`
- `ip netns exec fip-ns ip link set vport-fip up`
- `ip netns exec fip-ns ip route add default via 192.168.100.1 dev vport-fip`

#### add link from fip to router
- `ip link add fpr netns fip-ns type veth peer name rfp netns router-ns`

#### enable arp proxy
- `ip netns exec fip-ns sysctl net.ipv4.conf.vport-fip.proxy_arp=1`

#### add ip in router-ns
- `ip netns exec router-ns ip add add 172.16.18.1/24 dev vport-router_1`
- `ip netns exec router-ns ip add add 172.16.19.1/24 dev vport-router_2`
- `ip netns exec router-ns ip link set vport-router_1 up`
- `ip netns exec router-ns ip link set vport-router_2 up`
- `ip netns exec router-ns sysctl -w net.ipv4.ip_forward=1`

#### add ip in rfp
- `ip netns exec router-ns ip add add 169.254.31.238/31 dev rfp`
- `ip netns exec router-ns ip link set rfp up`

#### add ip in fpr
- `ip netns exec fip-ns ip add add 169.254.31.239/31 dev fpr`
- `ip netns exec fip-ns ip link set fpr up`
- `ip netns exec fip-ns ip route add 192.168.100.150 via 169.254.31.238 dev fpr`
- `ip netns exec fip-ns ip route add 192.168.100.160 via 169.254.31.238 dev fpr`
- `ip netns exec fip-ns ip route add 192.168.100.170 via 169.254.31.238 dev fpr`
- `ip netns exec fip-ns ip route add 192.168.100.180 via 169.254.31.238 dev fpr`

#### set ip route router-ns
- `ip netns exec router-ns ip route add default via 169.254.31.239 dev rfp`

#### set nat firewall
- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.150/32 -j DNAT --to-destination 172.16.18.100`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.18.100/32 -j SNAT --to-source 192.168.100.150`

- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.160/32 -j DNAT --to-destination 172.16.19.100`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.19.100/32 -j SNAT --to-source 192.168.100.160`

- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.170/32 -j DNAT --to-destination 172.16.19.200`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.19.200/32 -j SNAT --to-source 192.168.100.170`

- `ip netns exec router-ns iptables -t nat -A PREROUTING -d 192.168.100.180/32 -j DNAT --to-destination 172.16.18.200`
- `ip netns exec router-ns iptables -t nat -A POSTROUTING -s 172.16.18.200/32 -j SNAT --to-source 192.168.100.180`

#### set dhcp server
- `ovs-vsctl add-port br-int vport-dhcp_1 tag=10 -- set interface vport-dhcp_1 type=internal`
- `ovs-vsctl add-port br-int vport-dhcp_2 tag=20 -- set interface vport-dhcp_2 type=internal`

- `ip netns add dhcp-1-ns`
- `ip netns add dhcp-2-ns`
- `ip link set vport-dhcp_1 netns dhcp-1-ns`
- `ip link set vport-dhcp_2 netns dhcp-2-ns`

- `ip netns exec dhcp-1-ns ip add add 172.16.18.2/24 dev vport-dhcp_1`
- `ip netns exec dhcp-2-ns ip add add 172.16.19.2/24 dev vport-dhcp_2`

- `ip netns exec dhcp-1-ns ip link set vport-dhcp_1 up`
- `ip netns exec dhcp-2-ns ip link set vport-dhcp_2 up`

- `ip netns exec dhcp-1-ns dnsmasq -p0 --dhcp-range=172.16.18.10,172.16.18.253,1h --dhcp-option=3,172.16.18.1 --dhcp-host=00:00:00:00:01:01,172.16.18.100 --dhcp-host=00:00:00:00:02:01,172.16.18.200 --dhcp-leasefile=/var/lib/misc/dhcp-1-dnsmasq.leases`
- `ip netns exec dhcp-2-ns dnsmasq -p0 --dhcp-range=172.16.19.10,172.16.19.253,1h --dhcp-option=3,172.16.19.1 --dhcp-host=00:00:00:00:01:02,172.16.19.100 --dhcp-host=00:00:00:00:02:02,172.16.19.200 --dhcp-leasefile=/var/lib/misc/dhcp-2-dnsmasq.leases`



### VMS
```
virt-install --import --name cirros-vm-1 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:01:01 --check all=off
```
- `ovs-vsctl set port vnet0 tag=10`
```
virt-install --import --name cirros-vm-2 --memory 256 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk-clone.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:01:02 --check all=off
```
- `ovs-vsctl set port vnet1 tag=20`

## Host 2

#### create br host
- `ovs-vsctl add-br br-tun`
- `ovs-vsctl add-port br-tun vxlan1 -- set interface vxlan1 type=vxlan options:remote_ip=192.168.122.6`

#### create ex br
- `ovs-vsctl add-br br-ex`

#### add nic fip to Ovs
- `ovs-vsctl add-port br-ex enp7s0`

#### create int-br
- `ovs-vsctl add-br br-int`

#### create port for patch type 
- `ovs-vsctl add-port br-int int-ex tag=3 -- set interface int-ex type=patch options:peer=ex-int`
- `ovs-vsctl add-port br-ex ex-int -- set interface ex-int type=patch options:peer=int-ex`

- `ovs-vsctl add-port br-int int-tun -- set interface int-tun type=patch options:peer=tun-int`
- `ovs-vsctl add-port br-tun tun-int -- set interface tun-int type=patch options:peer=int-tun`

### VMS
```
virt-install --import --name cirros-vm-1 --memory 512 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:02:01 --check all=off
```
- `ovs-vsctl set port vnet0 tag=10`

```
virt-install --import --name cirros-vm-2 --memory 256 --vcpus 1 --cpu host \
     --disk cirros-0.3.2-x86_64-disk-clone.img,format=qcow2,bus=virtio \
     -w bridge=br-int,virtualport_type=openvswitch --mac=00:00:00:00:02:02 --check all=off
```
- `ovs-vsctl set port vnet1 tag=20`

