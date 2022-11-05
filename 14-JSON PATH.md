```sh
k get pods --sort-by=.metadata.name (hide $.items)
k get pods -o json
k get pods -o=jsonpath='{$.items[0].spec.containers[0].image}'
```

```sh
k get pods -o=jsonpath='{$.items[*].metadata.name} {"\n"} {$.items[*].status.capacity.cpu}'
```

```sh
k get pods -o=custom-columns=NODE:.metadata.name, CPU:.status.capacity.cpu
```

```sh
k get persistentvolume --sort-by=.spec.capacity.storage 
-o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage
```
### Using "" out of jsonpath and '' out of aws-user
```sh
k config view --kubeconfig=my-kube-config -o=jsonpath="{$.contexts[?(@.context.user=='aws-user')].name}"
```

```sh
k get pods -o=jsonpath="{range .items[*]}{.metadata.name}{.spec.containers[*].resources}{'\n'}"
```