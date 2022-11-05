## `ip` ([ Link1](https://diego.assencio.com/?index=d71346b8737ee449bb09496784c9b344), [ Link2](https://www.tecmint.com/ip-command-examples/), [ Link3](https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf) )
`ip addr Used to check the network interface with its ip cider, mac address and state ` \
`ip route to check the routing table with gateways`

1. Assign an IP Address to a Specific Interface
    ```sh
    ip addr add 192.168.50.5 dev eth1
    ```
2. Check interfaces
    ```sh
    ip addr 
    ```
3. Remove an IP Address
    ```sh
    ip addr del 192.168.50.5 dev eth1
    ```
4. Enable Network Interface
    ```sh
    ip link set eth1 up  
    ```
5. Disable Network Interface
    ```sh
    ip link set eth1 down 
    ```
6. Check the routing table
    ```sh
    ip route
    ```
    [ Explaination](https://diego.assencio.com/?index=d71346b8737ee449bb09496784c9b344) 
    ```sh
    default via 172.25.0.1 dev eth1 
    10.76.113.0/24 dev eth0 proto kernel scope link src 10.76.113.9 
    10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1 
    10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink 
    172.25.0.0/24 dev eth1 proto kernel scope link src 172.25.0.70
    ```
7. Create new Network Interface
    ```sh
    ip link add docker0 type bridge 
    ```

8. Create new Network Namespace
    ```sh
    ip netns add red
    ```
    view all Network Namespaces
    ```sh
    ip netns 
    ```
9. Check network interfaces
    ```sh
    ip link 
    ```
    Check network interfaces in one network namespace
    ```sh
    ip netns red exec ip link 
    ip -n red link
    ```
