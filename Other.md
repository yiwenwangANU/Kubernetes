## Set Alias and Auto Completion

k8 doc search cheat

## Get pod with certain label
```sh
k get pods --selector="env=dev"
```

## Count the outcome lines
```sh
k get pods -no-headers| wc -l
```

## Use base64 encoder in one line
```sh
cat jane.csr | base64 -w 0
```

## Check the process
```sh
ps aux | grep -i something
```

## Create service
```sh
k expose pod httpd --port=80 --name=httpd-service
k run httpd --image=httpd:alpine --port=80 -expose
```

## View something inside a pod
```sh
k exec my-pod -- cat /log/app.log
```

