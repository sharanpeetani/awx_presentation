Vagrant.configure("2") do |config|
  servers=[

    {
      :host => "ubuntu1",
      :hostname => "ubuntu1.lab.com",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.56.107",
      :ssh_port => '2230',
      :cpu => '2',
      :memory => '2048'
    },
    {
      :host => "ubuntu2",
      :hostname => "ubuntu2.lab.com",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.56.108",
      :ssh_port => '2231',
      :cpu => '1',
      :memory => '1024'
    },
    {
      :host => "rocky1",
      :hostname => "rocky1.lab.com",
      :box => "rockylinux/8",
      :ip => "192.168.56.109",
      :ssh_port => '2232',
      :cpu => '1',
      :memory => '1024'
    },
    {
      :host => "rocky2",
      :hostname => "rocky2.lab.com",
      :box => "rockylinux/8",
      :ip => "192.168.56.110",
      :ssh_port => '2233',
      :cpu => '1',
      :memory => '1024'
    },

  ]

  servers.each do |machine|

    config.vm.define machine[:host] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
      node.vm.synced_folder '.', '/vagrant', disabled: true

      node.vm.network :private_network, ip: machine[:ip]
      node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", machine[:memory]]
        v.customize ["modifyvm", :id, "--cpus", machine[:cpu]]
        v.customize ["modifyvm", :id, "--name", machine[:hostname]]
      end
    end
  end

  id_rsa_key_pub = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))

  config.vm.provision :shell,
        :inline => "echo 'appending SSH public key to ~vagrant/.ssh/authorized_keys' && echo '#{id_rsa_key_pub }' >> /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys"

  config.ssh.insert_key = false
  config.vbguest.auto_update = false
end
