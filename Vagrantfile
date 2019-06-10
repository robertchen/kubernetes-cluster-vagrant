# -*- mode: ruby -*-
# vi: set ft=ruby :

$configureBox = <<-SHELL

  apt-get update
  apt-get upgrade -y

  # Docker prerequisites
  apt-get install -y apt-transport-https ca-certificates curl software-properties-common
  # install docker
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt-get update
  #apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
  apt-get install -y apt-transport-https docker-ce=18.06.0~ce~3-0~ubuntu
  apt-mark hold docker-ce
  # add vagrant to docker group
  usermod -aG docker vagrant

  # kubelet requires swap off, no needed because no swap
  #swapoff -a
  # keep swap off after reboot
  #sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

  # Kubernetes prerequisites
  # apt-get update
  # apt-get install -y apt-transport-https curl
  #install kubernetes
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
  deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
  # install kubeadm、kubelet、kubectl
  apt-get update
  apt-get install -y kubelet kubeadm kubectl
  apt-mark hold kubelet kubeadm kubectl

  # get enp0s8 ip addr, eth0 is 10.0.2.15 for NAT
  IPADDR=$(ip a show enp0s8 | grep inet | grep -v inet6 | awk '{print $2}' | cut -f1 -d/)
  # bind kubelet to the private IP addr
  #sed -i "/KUBELET_EXTRA_ARGS=/c\KUBELET_EXTRA_ARGS=--node-ip=$IPADDR" /etc/default/kubelet
  echo "KUBELET_EXTRA_ARGS=--node-ip=$IPADDR" > /etc/default/kubelet
  # kubelet reload
  systemctl daemon-reload
  systemctl restart kubelet

SHELL

$configureMaster = <<-SHELL

  echo "This is the master"

  # get enp0s8 ip addr, eth0 is 10.0.2.15 for NAT
  IPADDR=$(ip a show enp0s8 | grep inet | grep -v inet6 | awk '{print $2}' | cut -f1 -d/)
  HOSTNAME=$(hostname -s)

  # kubeadm init
  # for Flannel:
  #kubeadm init --apiserver-advertise-address=$IPADDR --apiserver-cert-extra-sans=$IPADDR --node-name $HOSTNAME --pod-network-cidr=10.244.0.0/16
  # for Calico:
  kubeadm init --apiserver-advertise-address=$IPADDR --apiserver-cert-extra-sans=$IPADDR --node-name $HOSTNAME --pod-network-cidr=192.168.0.0/16

  # for vagrant user running kubectl
  sudo --user=vagrant mkdir -p /home/vagrant/.kube
  cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

  # Flannel
  #export KUBECONFIG=/etc/kubernetes/admin.conf
  #kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
  # Calico
  export KUBECONFIG=/etc/kubernetes/admin.conf
  kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
  kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

  # save kubectl join command
  kubeadm token create --print-join-command > /etc/kubeadm_join_cmd.sh
  chmod +x /etc/kubeadm_join_cmd.sh

  # allow ssh password authentication
  sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
  systemctl restart sshd

SHELL

$configureNode = <<-SHELL

  echo "This is a worker"

  apt-get install -y sshpass
  #sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@192.168.33.11:/etc/kubeadm_join_cmd.sh .
  sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@172.17.8.11:/etc/kubeadm_join_cmd.sh .
  sh ./kubeadm_join_cmd.sh

SHELL

Vagrant.configure(2) do |config|

  (1..3).each do |i|

    if i == 1 then
      vm_name = "master"
    else
      vm_name = "node#{i-1}"
    end

    config.vm.define vm_name do |s|

      # hostname
      s.vm.hostname = vm_name
      # node OS
      s.vm.box = "ubuntu/bionic64"
      # set netowrk
      # do not overlap with pod-network-cidr
      private_ip = "172.17.8.#{i+10}"
      s.vm.network "private_network", ip: private_ip

      # specify node specs
      s.vm.provider "virtualbox" do |v|
        v.gui = false        
        if i == 1 then
          v.cpus = 2
          v.memory = 1024
        else
          v.cpus = 1
          v.memory = 1024
        end
      end

      # common
      s.vm.provision "shell", inline: $configureBox

      if i == 1 then
        # Master
        s.vm.provision "shell", inline: $configureMaster
      else
        # Node
        s.vm.provision "shell", inline: $configureNode
      end

    end
  end
end

