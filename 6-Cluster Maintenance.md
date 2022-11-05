# Cluster Maintenance

## Question
- Upgrade kubeadm cluster to certain version (`cat /etc/*release*, upgrade cluster`)
- Backup the stacked etcd (`etcd backup, backup + restore + change volume in etcd.yaml`)
- Backup the externel etcd 
```sh
cat /etc/k8/man/apiserver.yaml | grep -i etcd
ssh etcd_server
scp backup_file etcd_server:~/backup
systemctl list-unit-files | grep -i etcd
systemctl cat etcd.service 
restore
ls -l data-dir + ls -l backup file
chown -R etcd:etcd backup file
vi etcd.service
reload deamon and etcd.service (as kubeadm upgrade)
```