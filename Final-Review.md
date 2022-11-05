# Final
1. Using `kubectl` in .sh file, location of kubelet config file: `~/.kube/config`, command `sed "s/unix/linux/g"`
4. Liveness, Readiness
7. 
12. podAntiAffinity
13. Environment variable
```sh
valueFrom:
    fieldRef:
        fieldPath: spec.nodeName

command:
 - sh
 - -c
 - 'command'
```
14. weave config in `/etc/cni/net.d/10-weave.conflist`
15. delete the container in the pod `crictl rm 1e020b43c4423`
16. `k api-resources --namespaced -o name > /opt/course/16/resources.txt`
17. `crictl inspect XX, ssh cluster1-worker2 'crictl logs b01edbe6f89ed' &> /opt/course/17/pod-container.log`
23. kubelet have two certificates within on dir

## Extra
1. `watch crictl ps -a, crictl logs containerID, journelctl | grep -i apiserver, /var/logs/containers/`
2. `k exec busybox -it -- sh, k logs XXpod --all-containers=true, ssh cluster1-master1 iptables-save | grep p2-service >> /opt/course/p2/iptables.tx`
3. `kubectl exec pod1 -- env | grep "TREE1=trauerweide", k exec pod1 -- cat /etc/birke`
4. ingress create (ingress controller)
5. Network policy