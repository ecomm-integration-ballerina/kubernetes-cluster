# Kubernetes cluster
A vagrant script for setting up a Kubernetes cluster using Kubeadm

## Pre-requisites

 * **[Vagrant 2.1.4+](https://www.vagrantup.com)**
 * **[Virtualbox 5.2.18+](https://www.virtualbox.org)**

## How to Run

Execute the following vagrant command to start a new Kubernetes cluster, this will start two masters and two nodes. This can be customized by reviewing the server array in the Vagrantfile. When you run the following command all servers defined in the list will be provisioned :

```
vagrant up
```

You can also start invidual machines by vagrant up followed by the machine, vagrant status will let you see the name's of the machines.

If more than two nodes are required, you can edit the servers array in the Vagrantfile

```
servers = [
    {
        :name => "k8s-node-3",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.13",
        :mem => "2048",
        :cpu => "2"
    }
]
 ```

As you can see above, you can also configure IP address, memory and CPU in the servers array. 

There will be 2 masters and 2 nodes of which the master is running nginx as reverse proxy for kubelets and kubectl. The set up is fairly simple to get up and running.

Make sure you install the network components to allow all nodes to be ready. This is because they will not enter the ready state without network services being provisioned accordingly:

```
kubectl apply -f /vagrant-service/network/
```

## Clean-up

Execute the following command to remove the virtual machines created for the Kubernetes cluster.
```
vagrant destroy -f
```

You can destroy individual machines by vagrant destroy k8s-node-1 -f

## Licensing

[Apache License, Version 2.0](http://opensource.org/licenses/Apache-2.0).
