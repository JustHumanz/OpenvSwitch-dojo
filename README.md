## OpenvSwitch dojo


this repo will contain PoC of openvswitch training

## My lab env
```
                                      ┌────────────┐
                                      │            │
   ┌──────────────────────────────────┤   My PC    ├────────────────────────────────┐
   │                                  │            │                                │
   │           ┌──────────────────────┤            ├─────────────────────┐          │
   │           │                      └──┬───────┬─┘                     │          │
   │           │                         │       │                       │          │
   │           │                         │       │                       │          │
   │           │                         │       │                       │          │
   │           │                         │       │                       │          │
   │           │                         │       │                       │          │
   │           │                         │       │                       │          │
   │enp1s0     │enp7s0             enp7s0│       │enp1s0                 │enp7s0    │enp1s0
 ┌─┴───────────┴─────┐            ┌──────┴───────┴────────┐          ┌───┴──────────┴─────┐
 │                   │            │                       │          │                    │
 │    Compute 1      │            │       Compute 2       │          │ Compute 3          │
 │                   │            │                       │          │                    │
 │                   │            │                       │          │                    │
 └───────────────────┘            └───────────────────────┘          └────────────────────┘
```
enp1s0 = 192.168.122.0/24 for manage network (dhcp)

enp7s0 = 192.168.100.0/24 for floating ip network

For diagram/topology i will use https://asciiflow.com/

### Note
**don't use this on production environment,this all was dumb&dirty workflows**