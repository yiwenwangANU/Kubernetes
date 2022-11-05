# Networking
## Question
- `ip addr, ip route, netstat -anp, iptables -L -t nat`
- How to check the process and service (`ps aux, systemctl list-unit-file, systemctl cat`)
- Locations of CNI (`cni@docs /opt/cni/bin + /etc/cni/net.d/`)
- How to curl the service `curl http://web-service.kube-system.svc.cluster.local` \
 `curl http://10-224-2-5.kube-system.pod.cluster.local`
- Troubleshooting the network connection problem (wrong host name because of the namespace)
- Create the ingress of the services (`ingress`)
## Establish connection between two ns in one node
1. Using network namespace to isolate the container aviod them to see the other resources on the host
    ```sh
    ip ns add red
    ip ns add blue
    ```
2. Establish the connection between two net namespace by create virtual cable
    ```sh
    ip link add veth-red type veth peer name veth-blue
    ```
    And attach two end to each net namespaces
    ```sh
    ip link set veth-red netns red
    ip link set veth-blue netns blue
    ```
3. Assign IP addresses to each net namespaces
    ```sh
    ip -n red addr add 192.168.15.1 dev veth-red
    ip -n blue addr add 192.168.15.2 dev veth-blue
    ```
    And set the interface up
    ```sh
    ip -n red link set veth-red up 
    ip -n blue link set veth-blue up
    ```
4. Check the connection by
    ```sh
    ip -n red ping 192.168.15.2
    ```

## Establish connections between multiple ns in one node
1. Using network namespace to isolate the container aviod them to see the other resources on the host
    ```sh
    ip ns add red
    ip ns add blue
    ip ns orange
    ```
2. Create an interface of a virtual network switch (Bridge network)
    ```sh
    ip link add v-net-0 type bridge
    ip link set dev v-net-0 up
    ```
3. Establish the connection between Bridge and net namespace by create virtual cable
    ```sh
    ip link add veth-red type veth peer name veth-red-br
    ip link add veth-blue type veth peer name veth-blue-br
    ```
    And attach two end to bridge and net namespace
    ```sh
    ip link set veth-red netns red
    ip link set veth-red-br master v-net-0
    ip link set veth-blue netns blue
    ip link set veth-blue-br master v-net-0
    ```
4.  Assign IP addresses to each net namespaces
    ```sh
    ip -n red addr add 192.168.15.1 dev veth-red
    ip -n blue addr add 192.168.15.2 dev veth-blue
    ```
    And set the interface up
    ```sh
    ip -n red link set veth-red up 
    ip -n blue link set veth-blue up
    ```
5. Assign IP cider to Bridge so the host could connect to the namespace
    ```sh
    ip addr add 192.168.15.5/24 dev v-net-0
    ```
6. Check the connection by
    ```sh
    ip ping 192.168.15.2
    ```

## Let the ns in one node (eth0 ip 192.168.1.2)could reach the IP in another node on 192.168.1.3

7. Add route in the blue namespace to the Bridge
    ```sh
    ip -n blue route add 192.168.1.0/24 via 192.168.15.5
    ip -n blue route add default via 192.168.15.5
    ```
8. Add nat functionality to the host
    ```sh
    iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
    ```
9. Check the connection 
    ```sh
    ip -n blue ping 192.168.1.3
    ```

## Let ping from other node (192.168.1.3) could reach the ns in one node
10. Set the nat by
    ```sh
    iptables -t nat -A POSTROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
    ```
## Set up Pod networking manually
1. Set the bridge
    ```sh
    ip link add v-net-0 type bridge
    ip link set dev v-net-0 up
    ip addr add 192.168.15.5/24 dev v-net-0
    ```
2. Set connection between bridge and ns
    ```sh
    ip ns add red
    ip link add veth-red type veth peer name veth-red-br
    ip -n red link set veth-red up 
    ip -n red addr add 192.168.15.1 dev veth-red
    ip link set veth-red netns red
    ip link set veth-red-br master v-net-0
    ```
3. Set connection between different nodes
    ```sh
    ip -n blue route add 10.244.1.0/24 via 192.168.1.11
    ip -n blue route add 10.244.2.0/24 via 192.168.1.12
    ip -n blue route add 10.244.3.0/24 via 192.168.1.13
    ```

## CNI in kubelets (installing addon)
kubelet using CNI to deploy network solution for us automatically, check by
```sh
ps aux | grep kubelet
```
`Weave can be one of the CNI and deploy its agent on every node as pod`

CNI plugin located in `/bin/cni/bin` \
CNI config located in `/etc/cni/net.d`, this is where kubelet find out which plugin to be used

Check the logs of weave by
```sh
k logs weave-pod-name weave -n=kube-system
```

## Service Networking
Normally we don't access pod directly, but use service instead. When we expose the pod, the kube-proxy will automatically assign an IP to the service and create the mapping in the nat table.  `Service is across nodes`
\
Check the mapping in the nat table
```sh
iptables -L -t nat | grep my-service
``` 
Check the service IP range
```sh
ps aux | grep kube-apiserver | grep service-cluster-ip-range
``` 

## DNS in k8
k8 deploy a build-in DNS server when you set up a cluster, DNS is across nodes.
For service within the same namespace, use DNS directly
```sh
curl http://web-service
``` 
For service within the different namespaces, use DNS + namespace
```sh
curl http://web-service.kube-system
curl http://web-service.kube-system.svc
curl http://web-service.kube-system.svc.cluster.local
``` 
For pods, `must specify the full name`
```sh
curl http://10-224-2-5.kube-system.pod.cluster.local
``` 

## CoreDNS
Created as a `deployment`, a `service` and a `configmap` in k8, check the deployment to see the config file, from the deployment we can find the location of config file and from configmap we can find resolve file
```sh
cat /etc/coredns/Corefile
cat /etc/resolv.conf
``` 

## Ingress
Help the external user to access the application from a single URL with routing in URL path, at the same time implement SSL certificate as well. Just like a Level 7 load balancer.

`Deploy` a `Ingress controller` (use Nginx-ingress-controller) and create a `configmap` for the configs of the deployment. Then create a `service` to link the deployment and a `service account` with roles to get right permissions. 

## Ingress Resources
`Ingress resources` is a set of rules and configs apply on the Ingress controller, all Ingress route to the test service on port 80
```sh
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
        name: minimal-ingress
    spec:
        backend:
            serviceName: test
            servicePort: 80
```
Create rules for routing between different URL paths
```sh
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: ingress-wildcard-host
    spec:
        rules:
        - host: "foo.bar.com"
            http:
            paths:
            - pathType: Prefix
                path: "/bar"
                backend:
                service:
                    name: service1
                    port:
                    number: 80
        - host: "*.foo.com"
            http:
            paths:
            - pathType: Prefix
                path: "/foo"
                backend:
                service:
                    name: service2
                    port:
                    number: 80
```

Without the rewrite-target option, this is what would happen:
```sh
annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```
http://ingress-service:ingress-port/watch --> http://watch-service:port/watch