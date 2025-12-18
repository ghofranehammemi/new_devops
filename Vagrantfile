Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  # Forward port 9000 de la VM vers ton PC
  config.vm.network "forwarded_port", guest: 9000, host: 9000
end
