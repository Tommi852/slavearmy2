Vagrant.configure(2) do |config|
  config.vm.box = "http://timonen.me/~tommi/boxit/custom.box"
  config.vm.usable_port_range = (2200..20000)

  r = rand(36**10).to_s(36)

  (1..200).each do |i|
    config.vm.define "slave#{i}" do |slave|
	slave.vm.hostname = "Slave#{i}-#{r}"

    slave.vm.provider "virtualbox" do |vb|
       vb.memory = 150
       vb.linked_clone = true
    end
    end
  end
end
