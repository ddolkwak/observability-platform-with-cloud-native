# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"  # OS Version Designation
  config.vm.box_check_update = false  # Prevent Box Check

  # VirtualBox Global Configuration
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.linked_clone = true
    vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
    vb.customize ["modifyvm", :id, "--nictype1", "virtio"] # Adapter 1 (Vagrant Default NAT)
    vb.customize ["modifyvm", :id, "--nictype2", "virtio"] # Adapter 2 (K8s Host-only Network)
  end

  # =========================================================
  # [Infra Provisioning] OS Optimazation, Create admin account, SSH Key Deployment
  # =========================================================
  $install_default = <<-SHELL
    export DEBIAN_FRONTEND=noninteractive

    echo '======== OS Optimazation & Admin Account Settings ========'
    echo '======== [1] Resolve Ubuntu Default SSH Restriction & Allow Password Authentication ========'
    sed -i 's/^#*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
    sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
    sed -i 's/^PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
    rm -f /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
    
    echo '======== [2] Resolve SSH HostKeyChecking between nodes ========'
    echo -e "Host *\n\tStrictHostKeyChecking no\n" >> /etc/ssh/ssh_config
    systemctl restart sshd

    echo '======== [3] Admin Account Settings ========'
    if ! id "admin" &>/dev/null; then
      useradd -m -s /bin/bash -u 1234 -g admin admin
      echo "admin:admin./" | chpasswd
      chown admin:admin /home/admin
      cp /etc/skel/.bashrc /home/admin/
      cp /etc/skel/.profile /home/admin/
      chown admin:admin /home/admin/.bashrc /home/admin/.profile
    fi

    echo '======== [4] Add admin to sudoers ========'
    echo "admin ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/admin
    chmod 0440 /etc/sudoers.d/admin

    echo '======== [5] Change SSH Directory Owner to admin ========'
    mkdir -p /home/admin/.ssh
    chown -R admin:admin /home/admin/.ssh
    
    if [ "$HOSTNAME" = "ansible-node" ]; then
      if [ ! -f /home/admin/.ssh/id_rsa ]; then
        su - admin -c 'ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa'
      fi
      cp /home/admin/.ssh/id_rsa.pub /vagrant/admin_rsa.pub
    fi
    
    sleep 5
    if [ -f /vagrant/admin_rsa.pub ]; then
      if ! grep -q -f /vagrant/admin_rsa.pub /home/admin/.ssh/authorized_keys 2>/dev/null; then
        cat /vagrant/admin_rsa.pub >> /home/admin/.ssh/authorized_keys
      fi
    fi

    echo '======== [6] Directory Authorization Setings ========'
    chown -R admin:admin /home/admin/.ssh
    chmod 700 /home/admin/.ssh
    chmod 600 /home/admin/.ssh/authorized_keys

    echo '======== [7] Timezone Settings ========'
    timedatectl set-timezone Asia/Seoul

    echo '======== [8] Register Nodename to /etc/hosts ========'
    if ! grep -q "k8s-master" /etc/hosts; then
      chmod 777 /etc/hosts
      cat << EOF >> /etc/hosts
192.168.56.133 ansible-node
192.168.56.130 k8s-master
192.168.56.131 k8s-worker-1
192.168.56.132 k8s-worker-2
EOF
      chmod 644 /etc/hosts
    fi
  SHELL

  # =========================================================
  # 1. Ansible Control Plane (ansible-node)
  # =========================================================
  config.vm.define "ansible-node" do |ansible|
    ansible.vm.hostname = "ansible-node"
    ansible.vm.network "private_network", ip: "192.168.56.133"
    ansible.vm.provider "virtualbox" do |vb|
      vb.name = "ansible-node"
      vb.memory = 1024
      vb.cpus = 1
    end

    # [Shared Folder Settings (for Github Connection)]
    ansible.vm.synced_folder "./ansible", "/home/admin/ansible",
      owner: 1234,
      group: "admin",
      mount_options: ["dmode=755", "fmode=644"]

    ansible.vm.provision :shell, privileged: true, inline: $install_default
    ansible.vm.provision :shell, privileged: true, inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      apt-add-repository --yes --update ppa:ansible/ansible
      apt-get install -y ansible

      echo '======== [9] Cross-connection & Default File Transfer ========'
      cp -r /etc/ansible/* /home/admin/ansible/ 2>/dev/null || true
      rm -rf /etc/ansible
      ln -s /home/admin/ansible /etc/ansible
      chown -R admin:admin /home/admin/ansible/
    SHELL
  end

  # =========================================================
  # 2. Kubernetes Master Node (k8s-master)
  # =========================================================
  config.vm.define "k8s-master" do |master|
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", ip: "192.168.56.130"
    master.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-master"
      vb.memory = 4096
      vb.cpus = 2
    end
    master.vm.provision :shell, privileged: true, inline: $install_default
  end

  # =========================================================
  # 3. Kubernetes Worker Nodes (k8s-worker-1, k8s-worker-2)
  # =========================================================
  (1..2).each do |i|
    config.vm.define "k8s-worker-#{i}" do |worker|
      worker.vm.hostname = "k8s-worker-#{i}"
      worker.vm.network "private_network", ip: "192.168.56.13#{i}"
      worker.vm.provider "virtualbox" do |vb|
        vb.name = "k8s-worker-#{i}"
        vb.memory = 2048
        vb.cpus = 2
      end
      worker.vm.provision :shell, privileged: true, inline: $install_default
    end
  end
end