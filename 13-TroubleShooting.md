# Trouble Shooting (link)
## Those wrong
- Cluster maintenance 4
- Security 1 (csr change name and request)
## Question
- DB not connected (`curl web-service, check deployment env, port number, label/selector`)
- Control Plane component problems(`k get pods -A, k logs XX pod`)
- Node01 is not ready ==> Kubelet is not working
  0. `service kubelet status, service kubelet restart`
  1. `journalctl -u kubelet | grep -i err/unable`
  2. `systemctl list-unit-files | grep -i kubelet`
  3. `systemctl cat kubelet.service | grep -i --color conf`
  4. `vi XX.conf` kubelet has two config files
  5. `service kubelet restart`
- Network problems
  1. kube-proxy failure (`k logs kube-proxy, k describe configmap XX. k edit ds kube-proxy`) kube-proxy has two config files
  2. DNS failure 
- Other commands
  1. `crictl ps -a, crictl logs XX`
### k get pods -A
4 static pods: etcd, apiserver, controller, scheduler \
1 deployment on each node: coreDNS \
2 deamonsets on each node: kube-proxy, add-on (weave, flannel)
### Application Failure
`k config set-context --current --namespace=beta` \
`alias and auto completion in cheat`\
`Check deployment instead of pod first` \
If the user cannot access the application, what step should we use to find out failure?
```sh
curl http://web-service.ip:port
k describe service web-service #check endpoint, nodeport, port, target port and selector
k describe deployment/pod web / k logs web -f or -previous # check DB env valuables
k describe service db-service
k describe pod db / k logs db -f or -previous
```

### Control Plane Failure
0. when k get pods not working
    ```sh
    crictl ps -a  # To get container ID
    crictl logs container-id  # To view the logs
    ```
1. check the status of nodes & pods
    ```sh
    k get nodes/pods
    k logs kube-apiserver
    ```
2. check the status of pods in kube-system
3. check service logs
    ```sh
    k logs kube-apiserver-master -n kube-system
    ```
4. modify the static pods

### Working node Failure
1. ssh node01
2. service kubelet status
3. service kubelet restart
4. journelctl -u kubelet 
5. cat /var/lib/kubelet/config.yaml
6. service kubelet restart

### Network Failure
1. Reinstall CNI
2. kube-proxy (is a deamonset, a configmap)
- check kube-proxy pod
- check kube-proxy logs
- check configmap
- config file located in `/var/lib/kube-proxy/config.conf` for deamonset and `/var/lib/kube-proxy/kubeconfig.conf` for configmap in the kube-proxy pod
3. CoreDNS (is a service [ service account ] , a deployment )