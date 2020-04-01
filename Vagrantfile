# -*- mode: ruby -*-
# vi: set ft=ruby :

servers = [
    {
        :name => "k8s-mstr-0",
        :type => "master",
        :box => "ubuntu/xenial64",
        :eth1 => "192.168.205.10",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-mstr-1",
        :type => "master",
        :box => "ubuntu/xenial64",
        :eth1 => "192.168.205.11",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node-1",
        :type => "node",
        :box => "ubuntu/xenial64",
        :eth1 => "192.168.205.12",
        :mem => "4096",
        :cpu => "2"
    },
    {
        :name => "k8s-node-2",
        :type => "node",
        :box => "ubuntu/xenial64",
        :eth1 => "192.168.205.13",
        :mem => "4096",
        :cpu => "2"
    }
]

# This script to install k8s using kubeadm will get executed after a box is provisioned
$configureBox = <<-SCRIPT
    apt-get update
    apt-get install -y apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
    apt-get update && apt-get install -y docker-ce

    # run docker commands as vagrant user (sudo not required)
    usermod -aG docker vagrant

    # install kubeadm
    apt-get install -y apt-transport-https curl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
    cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
    sysctl --system
    apt-get update
    apt-get install -y kubelet kubeadm kubectl
    apt-mark hold kubelet kubeadm kubectl

    # kubelet requires swap off
    swapoff -a

    # keep swap off after reboot
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

    # ip of this box
    IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`
    # set node-ip
    sudo systemctl restart kubelet
SCRIPT

$configureMaster = <<-SCRIPT
    echo "This is master"
    # ip of this box
    IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`
    
    # install k8s master
    HOST_NAME=$(hostname -s)
    if [ "$HOST_NAME" = "k8s-mstr-0" ]; then
        apt-get install -y nginx
        cat <<EOF > /etc/nginx/kube_api.conf
stream {
    server {
        listen 192.168.205.10:123;
        proxy_pass api_server;
    }

    upstream api_server {
        server 192.168.205.10:6443;
        server 192.168.205.11:6443;
    }
}
EOF
        grep kube_api.conf /etc/nginx/nginx.conf
        if [ $? -eq 1 ]; then   echo 'include /etc/nginx/kube_api.conf;' >> /etc/nginx/nginx.conf; fi
        systemctl enable nginx
        systemctl restart nginx
        kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=172.24.0.0/16 --control-plane-endpoint $IP_ADDR:123 --upload-certs
        sleep 3
        kubeadm init phase upload-certs --upload-certs | grep -v upload-certs > /vagrant-service/cert-key
    else
        CA_CERT_HASH=$(cat /vagrant-service/cert-key)
        JOIN_COMMAND=$(cat /vagrant-service/kubeadm_join_cmd.sh)
        $JOIN_COMMAND --control-plane --apiserver-advertise-address=$IP_ADDR --node-name $HOST_NAME --certificate-key $CA_CERT_HASH 
    fi

    #copying credentials to regular user - vagrant
    sudo --user=vagrant mkdir -p /home/vagrant/.kube
    cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
    chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

    if [ "$HOST_NAME" = "k8s-mstr-0" ]; then
        kubeadm token create --print-join-command > /vagrant-service/kubeadm_join_cmd.sh
        export KUBECONFIG=/etc/kubernetes/admin.conf
        kubectl completion bash > /home/vagrant/.kube/completion.bash.inc
        echo "source '/home/vagrant/.kube/completion.bash.inc'" >> /home/vagrant/.bashrc
        chown vagrant:vagrant /home/vagrant/.kube/completion.bash.inc
        chmod +x /vagrant-service/kubeadm_join_cmd.sh
    fi

SCRIPT

$configureNode = <<-SCRIPT
    echo "This is worker"
    sh /vagrant-service/kubeadm_join_cmd.sh
SCRIPT

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|

            config.vm.box = opts[:box]
            config.vm.hostname = opts[:name]
            config.vm.network :private_network, ip: opts[:eth1]
            config.vm.synced_folder ".", "/vagrant-service"
            config.vm.provision :hosts, :sync_hosts => true

            config.vm.provider "virtualbox" do |v|

                v.name = opts[:name]
            	v.customize ["modifyvm", :id, "--groups", "/Ballerina Development"]
                v.customize ["modifyvm", :id, "--memory", opts[:mem]]
                v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]

            end

            # we cannot use this because we can't install the docker version we want - https://github.com/hashicorp/vagrant/issues/4871
            #config.vm.provision "docker"

            config.vm.provision "shell", inline: $configureBox
            if opts[:type] == "master"
                config.vm.provision "shell", inline: $configureMaster
            elsif opts[:type] == "node"
                config.vm.provision "shell", inline: $configureNode
            end
        end
    end
end 