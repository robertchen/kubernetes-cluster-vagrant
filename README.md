# kubernetes-cluster-vagrant

This creates a kubernetes cluster using vagrant:
1. first running Copy Vagrant-master to a folder and running:
vagrant up
2. create calico networking stuffs:
kubectl create -f ./rbac-kdd.yaml
kubectl create -f ./calico.yaml

3. Copy Vagrant-second-node to a folder and run:
vagrant up



# reference:
https://gist.github.com/lizrice/69d3b28979391287176b3b7155a327b9
