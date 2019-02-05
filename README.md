# kubernetes-cluster-vagrant

Minikube kubernetes is not cluster and cannot dev/testing as a production cluster. This creates a kubernetes cluster using vagrant. Nodes are based on Ubuntu 18.04, kubernetes version is latest.
1. run:
vagrant up

2. output
```
robert@imac:~/src/vagrant-calico-cluster$ k get nodes -o wide
NAME     STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master   Ready    master   7m13s   v1.13.3   172.17.8.11   <none>        Ubuntu 18.04.1 LTS   4.15.0-45-generic   docker://18.6.0
node1    Ready    <none>   5m32s   v1.13.3   172.17.8.12   <none>        Ubuntu 18.04.1 LTS   4.15.0-45-generic   docker://18.6.0
node2    Ready    <none>   4m      v1.13.3   172.17.8.13   <none>        Ubuntu 18.04.1 LTS   4.15.0-45-generic   docker://18.6.0
robert@imac:~/src/vagrant-calico-cluster$ kubectl get pods -n kube-system -owide
NAME                             READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
calico-node-88whs                2/2     Running   0          7m1s    172.17.8.11   master   <none>           <none>
calico-node-8m77k                2/2     Running   0          4m7s    172.17.8.13   node2    <none>           <none>
calico-node-8prjr                2/2     Running   0          5m39s   172.17.8.12   node1    <none>           <none>
coredns-86c58d9df4-j28q4         1/1     Running   0          7m1s    192.168.0.2   master   <none>           <none>
coredns-86c58d9df4-tc9jh         1/1     Running   0          7m1s    192.168.0.3   master   <none>           <none>
etcd-master                      1/1     Running   0          6m14s   172.17.8.11   master   <none>           <none>
kube-apiserver-master            1/1     Running   0          6m20s   172.17.8.11   master   <none>           <none>
kube-controller-manager-master   1/1     Running   0          6m2s    172.17.8.11   master   <none>           <none>
kube-proxy-b7cg6                 1/1     Running   0          4m7s    172.17.8.13   node2    <none>           <none>
kube-proxy-gfpcf                 1/1     Running   0          5m39s   172.17.8.12   node1    <none>           <none>
kube-proxy-x9nb8                 1/1     Running   0          7m1s    172.17.8.11   master   <none>           <none>
kube-scheduler-master            1/1     Running   0          6m1s    172.17.8.11   master   <none>           <none>
```

3. issues and solutions
* the vms in virtualbox on MAC (NAT) always has the 10.0.2.15 and kubernetes set this as the api listening address, solution is to set an IP for the node and also set apiserver-advertise-address to this IP address.
the master ip is 172.17.8.211, second node is 172.17.8.212. Virtualbox networking setting is as below picture:

![Alt text](images/virtualbox-networking.png "Virtualbox networking settings")
      
* If kubectl exec cannot connect to the pod, this is because the nodes is running on 10.0.2.15 (virtualbox NAT). The kublet should be changed to be binded to eth1, the node ip.
```
robert@imac:~/src/kubernetes-learning/vagrant-cluster-calico$  kubectl exec -it nginx-7db75b8b78-47j9d -- bash 
error: unable to upgrade connection: pod does not exist

robert@imac:~/src/kubernetes-learning/vagrant-cluster-calico$ kubectl get nodes -o wide
NAME               STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kcluster-calico    Ready    master   7h24m   v1.13.3   10.0.2.15     <none>        Ubuntu 18.04.1 LTS   4.15.0-29-generic   docker://18.6.0
kcluster-calico2   Ready    <none>   124m    v1.13.3   10.0.2.15     <none>        Ubuntu 18.04.1 LTS   4.15.0-29-generic   docker://18.6.0
```
solution is adding these to Vagrantfile:
```
sed -i "/KUBELET_EXTRA_ARGS=/c\KUBELET_EXTRA_ARGS=--node-ip=$IPADDR" /etc/default/kubelet
systemctl daemon-reload
systemctl restart kubelet
```

# reference:

https://www.objectif-libre.com/en/blog/2018/07/05/k8s-network-solutions-comparison/

