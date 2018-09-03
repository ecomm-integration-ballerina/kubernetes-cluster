servers = [
    {
        :name => "k8s-master",
        :box => "centos/7",
        :box_version => "1804.02",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "k8s-node-1",
        :box => "centos/7",
        :box_version => "1804.02",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "k8s-node-2",
        :box => "centos/7",
        :box_version => "1804.02",
        :eth1 => "192.168.205.12",
        :mem => "1024",
        :cpu => "1"
    }
]

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|

            config.vm.box = opts[:box]
            config.vm.box_version = opts[:box_version]
            config.vm.hostname = opts[:name]
            config.vm.network :private_network, ip: opts[:eth1]

             config.vm.provider "virtualbox" do |v|

                v.name = opts[:name]
            	v.customize ["modifyvm", :id, "--groups", "/Ballerina Development"]
                v.customize ["modifyvm", :id, "--memory", opts[:mem]]
                v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]

             end

        end

    end

end 