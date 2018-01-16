# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.3"
  
  share_path = "/home/vagrant/share/k8s"

  config.vm.synced_folder "k8_cluster", "#{share_path}"

   config.vm.provider "virtualbox" do |vb|
     # Customize the amount of memory on the VM:
     vb.memory = "1024"
   end
  
config.vm.define "master" do |master|
  master.vm.network "private_network", ip: "10.0.0.10"
  master.vm.hostname = "master"
  master.vm.provider "virtualbox" do |vb|
    vb.name = "master"
  end
  master.vm.provision "shell", inline: <<-SHELL
    yum -y install etcd 
    cp #{share_path}/etc/etcd/etcd.conf /etc/etcd/etcd.conf
    cp #{share_path}/etc/kubernetes/apiserver /etc/kubernetes/apiserver
    systemctl enable etcd kube-apiserver kube-controller-manager kube-scheduler 
    systemctl start etcd kube-apiserver kube-controller-manager kube-scheduler 
    sleep 5
    systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler 
  SHELL
end

minions = {
  "minion1" => "10.0.0.11",
  "minion2" => "10.0.0.12",
  "minion3" => "10.0.0.13"
}


minions.each do |v_m, addr|
  config.vm.define "#{v_m}" do |kube_machine|
    kube_machine.vm.network "private_network", ip: "#{addr}"
    kube_machine.vm.hostname = "#{v_m}"
    kube_machine.vm.provider "virtualbox" do |vb|
      vb.name = "#{v_m}"
    end
    kube_machine.vm.provision "shell", inline: <<-SHELL
      cp #{share_path}/etc/kubernetes/kubelet /etc/kubernetes/kubelet
      sed -i "s/SED_WILL_OVERRIDE_HOSTNAME/${HOSTNAME}/" /etc/kubernetes/kubelet 
      systemctl enable kube-proxy kubelet docker
      systemctl start kube-proxy kubelet docker
      sleep 5
      systemctl status kube-proxy kubelet docker 
    SHELL
  end
end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
   config.vm.provision "shell", inline: <<-SHELL
      cp #{share_path}/etc/yum.repos.d/docker.repo /etc/yum.repos.d
      yum clean all
      yum --enablerepo=virt7-docker-common-release -y install rsync wget emacs-nox ntp kubernetes docker
      echo '(setq backup-directory-alist `(("." . "~/.saves")))' >> /home/vagrant/.emacs
      echo '(setq backup-by-copying t)' >> /home/vagrant/.emacs
      chown vagrant:vagrant /home/vagrant/.emacs
      cp /home/vagrant/.emacs /root/
      chown root:root /root/.emacs 
      cp -f #{share_path}/etc/hosts /etc/hosts
      rm -f /etc/localtime
      ln -s /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
      systemctl enable ntpd && systemctl start ntpd && systemctl status ntpd
      # Master config is the same on all k8s hosts
      cp #{share_path}/etc/kubernetes/config /etc/kubernetes/config  
    SHELL
end
