# Security
## Questions
- Create key and csr for user1, approve the csr and generate the crt (`openssl + csr`)
- Controlplane components use the wrong ca/crt/key and how to find out
- Use certain config context to run some command, or set the default context (`kube config + kube config use context`)
- Create the necessary roles and role bindings required for the dev-user to create, list and delete pods in the default namespace (`rbac`)
- Create a service account, create its role and its bind them using rbac. Link the service account to a pod in pod definition file. Link the service account to a third-party software by create token from service account.
(`service account pod, k create token service-account`)
- How to use private image on the docker hub (`private image`)
- How to run a pod as a particular user and with particular capabliity (`security context`)
- How to control the ingress and egress of a pod (`network policy`)
## Certificates of controlplane components
### apiserver
- client-ca-file apiserver.crt apiserver.key 
- etcd-ca-file apiserver-etcd-client.crt apiserver-etcd-client.key
- kubelet-certificate-authority apiserver-etcd-client.crt apiserver-etcd-client.key
### etcd
- trusted-ca-file etcdserver.key etcdserver.crt
- peer-trusted-ca-file etcdpeer1.crt peer.key (For communication between different etcd server in high avilibility environment)
### controller
- root-ca-file ca.crt ca.key
## How to generate key and crt by Openssl
1. Using openssl to generate key
2. Using openssl to generate certificate signing request file in XX.csr format from the key
3. Create a certificate signing request in k8 using the content in XX.csr file with base64
4. Admin in k8 could aprove/ deny the request, if aproved the request now have the certificate field
5. Copy the certificate field and decode with base64 finally store in a XX.crt file as the certificate.  

Check to Doc@"openssl" to get how to generate key & crt and view the crt. 
- Information in crt includes issuer, CN, validate data, alternative name
- Be care there are two CN in crt.
##  Certificate Signing Request ([Certificate Signing Request](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/) )
For normal user to get their certificate for authorization \
Generate key for Jane
   ```sh
   openssl genrsa -out jane.key 2048
   ```
Certificate Signing Request
   ```sh
   openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
   ```
Encode with base64
   ```sh
   cat jane.csr | base64 -w 0
   ```
Put in the request jane-csr.yaml file and send the request
   ```sh
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
    name: myuser  # name
spec:
    request:  # request                              
    signerName: kubernetes.io/kube-apiserver-client
    expirationSeconds: 86400  # one day
    usages:
    - client auth
   ```
Admin view and aprove the request by 
   ```sh
   kubectl get csr
   kubectl get csr jane -o yaml
   kubectl certificate approve jane
   ```
or deny the request by
   ```sh
   kubectl certificate deny jane
   kubectl delete csr jane
   ```
Get the certificate, copy the certificate part
   ```sh
   kubectl get csr/jane -o yaml
   ```
Decode with base64
   ```sh
   kubectl get csr jane -o jsonpath='{.status.certificate}'| base64 -d > jane.crt
   ```
   or
   ```sh
   echo “LS0...Qo=” | base64 --decode 
   ```

##  kube Config use-context ([kube config & kube Config use-context](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/))
To authorize I am the right person to access the kube cluster, I need to provide 4 information: key, crt, ca.crt and server. 
   ```sh
    kubectl get pods
        --client-key admin.key
        --client-certificate admin.crt
        --certificate-authority ca.crt
        --server my-kube-playground:6443
   ```

We are lazy, so we will use `kubectl config use-context prod-user@production` instead. The kube system will use the config file in `$HOME/.kube/config` by default if not specified.

Create the config file as following:
   ```sh
    apiVersion: v1
    kind: Config
    clusters:
    - name: my-kube-playground
      cluster:
        certificate-authority: ca.crt
        server: https://my-kube-playground:6443

    contexts:
    - name: my-kube-admin@my-kube-playground
      context: 
        cluster: my-kube-playground
        user: my-kube-admin
        namespace: finance
    
    users:
    - name: my-kube-admin
    user:
        client-certificate: admin.crt
        client-key: admin.key
   ```
We can multiple users&cluster pairs in the config file
   ```sh
   apiVersion: v1
   kind: Config
   current-context: my-kube-admin@my-kube-playground
   clusters:
   - name: my-kube-playground  (values hidden...)
   - name: development
   - name: production
   - name: google
   contexts:
   - name: my-kube-admin@my-kube-playground
   - name: admin@development
   - name: dev-user@production
   - name: prod-user@google
   users:
   - name: my-kube-admin  
   - name: admin
   - name: dev-user
   - name: prod-user
   ```

Kubectl create is not needed, view the config by
   ```sh
   kubectl config view
   ```
Use the other context by specify the config file location and use context
   ```sh
   kubectl config --kubeconfig=/root/my-kube-config use-context user@cluster
   kubectl config --kubeconfig=/root/my-kube-config current-context
   kubectl config use-context prod-user@production
   ```
can use `certificate-authority-data:` instead of `certificate-authority:` with base64 encoded data

##  Role based access control ([Role](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) )
IAM role in k8, create by role and role binding \
Check API Group
   ```sh
   curl http://localhost:6443 -k
      --key admin.key
      --cert admin.crt
      --cacert ca.crt
   curl http://localhost:6443/apis -k | grep “name”
      --key admin.key
      --cert admin.crt
      --cacert ca.crt
   ```
Start kubectl proxy so we can access API Group without providing key&crt
   ```sh
   kubectl proxy
   curl http://localhost:8001 -k
   ```
Authorization Mode, set in the kube-apiserver file
   ```sh
   --authorization-mode=Node,RBAC,Webhook 
   ```
RBAC, 1st create role by
   ```sh
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
      name: developer
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["list", "get", "create", "update", "delete"]
     resourceNames: ["blue", "orange"]
   - apiGroups: [""]
     resources: ["ConfigMap"]
     verbs: ["create"]
   ```
2rd, bind the role to the user
   ```sh
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
      name: devuser-developer-binding
   subjects:
   - kind: User
     name: dev-user
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: Role
     name: developer
     apiGroup: rbac.authorization.k8s.io
   ```

check role binding
   ```sh
   kubectl get roles
   kubectl get rolebindings
   kubectl describe role developer
   kubectl describe rolebinding devuser-developer-binding
   kubectl auth can-i create pods --as dev-user --namespace test
   ```
## Cluster Role and Binding ([RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) )
Using Cluster role and binding for cluster scoped resources like nodes, PV or for those namespaced resources but within all namespaces

##  Service Account ([Service Account](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) )
IAM Roles for the services, after create the service account, we need to generate a token as credential as well. Export the token for third-party authentication. Create the service account and its token by 
   ```sh
   k create serviceaccount jenkins
   k create token jenkins
   ```
Specify the service account in a deployment, if not sepcified default will be used
   ```sh
   apiVersion: apps/v1 
   kind: Deployment
   metadata:
     name: nginx-deployment
     namespace: default
   spec:
     replicas: 3
     template:
       metadata:
       # ...
       spec:
         serviceAccountName: bob-the-bot 
         # Sepcify yhe service account here and service account will be automatically mounted 
         containers:
         - name: nginx
           image: nginx:1.14.2
   ```
Token will be mounted to the pod volumn
## Image Security ([Image Security](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) )
Using private image on Docker hub by create the secret file:
   ```sh
   kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
   ```
And config the pod yaml to use the secret
   ```sh
   apiVersion: v1
   kind: Pod
   metadata:
     name: private-reg
   spec:
     containers:
     - name: private-reg-container
       image: <your-private-image>
     imagePullSecrets:
     - name: regcred
   ```
##  Security Context ([Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) )

Run the pod as particular user
   ```sh
   apiVersion: v1
   kind: Pod
   metadata:
   name: security-context-demo-2
   spec:
     securityContext:
       runAsUser: 1000
     containers:
     - name: sec-ctx-demo-2
       image: gcr.io/google-samples/node-hello:1.0
       securityContext:
         runAsUser: 2000
         capabilities:
           add: ["NET_ADMIN", "SYS_TIME"]
   ```

## Network Policy ([Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/) )

Act like Security Group in AWS