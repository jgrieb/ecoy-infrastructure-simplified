Vagrant.require_version ">= 2.0.0"

ssh_username="vagrant"
mount_synced_folder='/vagrant'
ENV['VAGRANT_DEFAULT_PROVIDER'] = "virtualbox"
Vagrant.configure('2') do |config|
  config.ssh.username = "vagrant"

  # Default configuration for virtualbox machines
  config.vm.provider "virtualbox" do |h, override|
    override.vm.box = "bento/ubuntu-20.04"
    override.vm.synced_folder ".", mount_synced_folder, type:"rsync", rsync__auto: true, rsync__exclude: [".git/","/config/"]
  end

  # Script to update the ansible inventory file with the ip assigned to the machine once it has been provisioned
  $update_inventory_script = <<-SCRIPT
    echo start update_inventory_script
    echo $(hostname -I)
    INVENTORY_FILE=$1
    MACHINE_NAME=$2
    SSH_USERNAME=$3
    LINE="$MACHINE_NAME $(echo 'ansible_host=')$(hostname -I)"
    LINE="$LINE"
    sed -i "s/$MACHINE_NAME ansible_host=.*/$LINE/g" "$INVENTORY_FILE"
    sed -i "s/ansible_ssh_user=.*/ansible_ssh_user=$SSH_USERNAME/g" "$INVENTORY_FILE"
  SCRIPT

  cordra_nsidr_server_name = 'first_test_server'
  # Definition of the virtual machine that will be hosting cordra repository server
  config.vm.define cordra_nsidr_server_name, autostart:true do |cordra_nsidr_server|
      machine_name = cordra_nsidr_server_name
      # cordra_nsidr_server.vm.network "private_network", name: "vboxnet0", ip: "172.28.128.8", adapter: 2

      # Specific setup for this virtual machine when using the virtualbox provider
      cordra_nsidr_server.vm.provider "virtualbox" do |h, override|
        h.memory = 1024
        h.cpus = 1
      end

      # Provisioner that run the script that updates the ansible inventory with the IP assigned to this virtual machine
      cordra_nsidr_server.vm.provision "update_inventory", type: "shell" do |s_inventory|
        s_inventory.inline        = $update_inventory_script
        s_inventory.args          = [mount_synced_folder+'/ansible/inventory.ini',machine_name,ssh_username]
        s_inventory.privileged    = false
      end

  end

end
