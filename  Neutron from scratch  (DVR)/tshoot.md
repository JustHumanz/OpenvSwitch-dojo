#### Check if vxlan was working or not
- `tcpdump -l -n -i enp1s0 'port 4789'`

- `ip netns exec <ns> tcpdump -i <interface> -w - -U | tee /tmp/avv | tcpdump -r -`