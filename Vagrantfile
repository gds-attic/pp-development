# -*- mode: ruby -*-
# vi: set ft=ruby :

# Node definitions
hosts = [
  { name: 'pp-development-1',  ip: '10.0.0.100' },
]

# Images are built for specific versions of virtualbox guest additions for now.
# It has proven problematic to mix versions of virtualbox and guest additions
# in the past and therefore they are pinned by what is available in this map.
$boxesByVersion = {
  "4.3.6r91406" => {
    name: "pp-ubuntu-12.04-virtualbox-4.3.6r91406",
    url: "https://s3-eu-west-1.amazonaws.com/gds-boxes/pp-ubuntu-12.04-virtualbox-4.3.6r91406.box",
    link: "http://download.virtualbox.org/virtualbox/4.3.6/",
  },
}

def get_box(provider)
  provider    ||= "virtualbox"
  case provider
  when "vmware"
    name  = "puppetlabs-ubuntu-server-12042-x64-vf503-nocm"
    url   = "http://puppet-vagrant-boxes.puppetlabs.com/ubuntu-svr-12042-x64-vf503-nocm.box"
  else
    virtualBoxVersion = `vboxmanage --version`.strip
    box = $boxesByVersion[virtualBoxVersion]
    if box.nil?
      $stderr.puts "Virtualbox version #{virtualBoxVersion} is not supported by pp-development. See README.md.\nSupported: #{$boxesByVersion.keys} --> #{$boxesByVersion.values.map {|item| item[:link]}}"
      exit 1
    end

    name, url = box[:name], box[:url]
  end
  return name, url
end

"#{Vagrant::VERSION}" < "1.1.0" and Vagrant::Config.run do |config|
  hosts.each do |host|
    config.vm.define host[:name] do |c|
      box_name, box_url = get_box("virtualbox")
      c.vm.box = box_name
      c.vm.box_url = box_url

      c.vm.host_name = host[:name] + '.localdomain'
      c.vm.network :hostonly, host[:ip], :netmask => "255.255.255.0"

      modifyvm_args = ['modifyvm', :id]

      # Mitigate boot hangs.
      modifyvm_args << "--rtcuseutc" << "on"

      # Isolate guests from host networking.
      modifyvm_args << "--natdnsproxy1" << "on"
      modifyvm_args << "--natdnshostresolver1" << "on"
      modifyvm_args << "--name" << host[:name]

      c.vm.customize(modifyvm_args)
      c.ssh.forward_agent = true
      c.vm.provision :shell, :path => "tools/bootstrap-vagrant"
      c.vm.share_folder "apps", "/var/apps", ".."
      c.vm.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/apps", "1"]

      c = load_local_vagrant_file(host[:name], c)
    end
  end
end

def load_local_vagrant_file(name, config)
  # Load local overrides
  if File.exist? "#{name}-Vagrantfile.local"
    $stderr.puts "WARNING: Vagrantfile.local is deprecated! Please use Vagrantfile.localconfig instead."
    $stderr.puts ""
  end

  if File.exist? "#{name}-Vagrantfile.localconfig"
    instance_eval File.read("#{name}-Vagrantfile.localconfig"), "#{name}-Vagrantfile.localconfig"
  end

  config
end


"#{Vagrant::VERSION}" >= "1.1.0" and Vagrant.configure("2") do |config|
  if Vagrant.has_plugin? "vagrant-dns"
    config.dns.tld = "perfplat.dev"
    config.dns.patterns = [/^.*perfplat.dev$/]
  end

  hosts.each do |host|
    config.vm.define host[:name] do |c|
      box_name, box_url = get_box("virtualbox")
      c.vm.box = box_name
      c.vm.box_url = box_url

      c.vm.hostname = host[:name] + '.localdomain'
      c.vm.network :private_network, ip: host[:ip], netmask: '255.255.255.0'

      c.vm.provider :virtualbox do |vb, override|
        modifyvm_args = ['modifyvm', :id]
        # Mitigate boot hangs.
        modifyvm_args << "--rtcuseutc" << "on"
        # Isolate guests from host networking.
        modifyvm_args << "--natdnsproxy1" << "on"
        modifyvm_args << "--natdnshostresolver1" << "on"
        modifyvm_args << "--name" << host[:name]
        vb.customize(modifyvm_args)
      end

      c.vm.provider :vmware_fusion do |f, override|
        vf_box_name, vf_box_url = get_box("vmware")
        override.vm.box = vf_box_name
        override.vm.box_url = vf_box_url
        f.vmx["memsize"] = "256"
        f.vmx["numvcpus"] = "1"
        f.vmx["displayName"] = host[:name]
      end

      c.ssh.forward_agent = true
      c.vm.provision :shell, :path => "tools/bootstrap-vagrant"

      c.vm.provider :virtualbox do |vb, override|
        override.vm.synced_folder "..", "/var/apps", :nfs => false
      end

      c.vm.provider :vmware_fusion do |f, override|
        override.vm.synced_folder "..", "/var/apps"
      end

      c = load_local_vagrant_file(host[:name], c)
    end
  end
end
