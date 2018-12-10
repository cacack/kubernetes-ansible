Vagrant.configure("2") do |config|

  cpu_total_cores = %x(grep -c processor /proc/cpuinfo)
  mem_raw = %x(cat /proc/meminfo | grep MemTotal: | \
    gawk '{gsub(/MemTotal:/,\"\");print}' | \
    gawk '{gsub(/kB/,\"*1024\");print}' | \
    gawk '{gsub(/KB/,\"*1024\");print}' | \
    gawk '{gsub(/KiB/,\"*1024\");print}' | \
    gawk '{gsub(/MB/,\"*1048576\");print}' | \
    gawk '{gsub(/MiB/,\"*1048576\");print}' | \
    gawk '{gsub(/GB/,\"*1073741824\");print}' | \
    gawk '{gsub(/GiB/,\"*1073741824\");print}' | \
    gawk '{gsub(/TB/,\"*1099511627776\");print}' | \
    gawk '{gsub(/TiB/,\"*1099511627776\");print}' | \
    gawk '{gsub(/B/,\"*1\");print}' | \
    gawk '{gsub(/[^1234567890*]/,\"\");print}' \
  )
  mem_total_mb = eval(mem_raw) / 1000 / 1000 / 1024 * 1024

  config.vm.define "master" do |master|
    master.vm.box = "ubuntu/bionic64"
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.254.2"
    master.vm.provision :shell, path: "tests/files/provision-script.sh"
    master.vm.provider "virtualbox" do |v|
      v.memory = mem_total_mb.to_i / 8
      v.cpus = cpu_total_cores.to_i / 2
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
    end
  end

  for count in 1..3
    config.vm.define "worker#{count}" do |worker|
      host_ip = count + 2
      worker.vm.box = "ubuntu/bionic64"
      worker.vm.hostname = "worker#{count}"
      worker.vm.network "private_network", ip: "192.168.254.#{host_ip}"
      worker.vm.provision :shell, path: "tests/files/provision-script.sh"
      worker.vm.provider "virtualbox" do |v|
        v.memory = mem_total_mb.to_i / 16
        v.cpus = cpu_total_cores.to_i / 4
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
      end
    end
  end

  config.vm.define "client" do |client|
    client.vm.box = "ubuntu/bionic64"
    client.vm.hostname = "client"
    client.vm.network "private_network", ip: "192.168.254.6"
    client.vm.provision :shell, path: "tests/files/install-pip.sh"
    client.vm.provision :shell, path: "tests/files/provision-script.sh"
    client.vm.provider "virtualbox" do |v|
      v.memory = mem_total_mb.to_i / 16
      v.cpus = cpu_total_cores.to_i / 4
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
    end
  end
end
