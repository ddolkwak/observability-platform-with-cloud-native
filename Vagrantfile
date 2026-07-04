# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"  # OS를 Ubuntu 22.04 LTS (Jammy)로 지정
  config.vm.box_check_update = false  # 매번 박스 업데이트를 확인하는 것을 방지하여 부팅 속도 향상

  # VirtualBox 글로벌 고급 설정
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.linked_clone = true
    vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
  end

  # =========================================================
  # [공통 프로비저닝] OS 최적화, admin 사용자 생성 및 SSH Key 배포 (인프라 기초 공사)
  # =========================================================
  $install_default = <<-SHELL
    export DEBIAN_FRONTEND=noninteractive

    echo '======== OS 기본 최적화 및 admin 계정 설정 ========'
    echo '======== [1] Ubuntu 기본 SSH 제한 해제 및 패스워드 인증 허용 ========'
    sed -i 's/^#*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
    sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
    sed -i 's/^PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
    rm -f /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
    
    echo '======== [2] 노드 간 통신 시 SSH 지문 묻기 무시 ========'
    echo -e "Host *\n\tStrictHostKeyChecking no\n" >> /etc/ssh/ssh_config
    systemctl restart sshd

    echo '======== [3] admin 사용자 생성 및 비밀번호(admin./) 설정 ========'
    if ! id "admin" &>/dev/null; then
      useradd -m -s /bin/bash -u 1234 -g admin admin
      echo "admin:admin./" | chpasswd
    fi

    echo '======== [4] admin 사용자에게 비밀번호 없는 sudo 권한 부여 (Ansible 제어용) ========'
    echo "admin ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/admin
    chmod 0440 /etc/sudoers.d/admin

    echo '======== [5] admin 계정 기준의 SSH Key 패스워드리스 환경 구축 ========'
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

    echo '======== [6] 디렉토리 권한 설정 ========'
    chown -R admin:admin /home/admin/.ssh
    chmod 700 /home/admin/.ssh
    chmod 600 /home/admin/.ssh/authorized_keys

    echo '======== [7] Timezone 세팅 ========'
    timedatectl set-timezone Asia/Seoul

    echo '======== [8] 모든 노드간 이름 해소를 위한 /etc/hosts 자동 등록 ========'
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

    # [공유 폴더 연동 설정]
    # 호스트의 "./ansible" 폴더를 VM의 "/home/admin/ansible"로 마운트
    # UID 1234(admin) 및 시스템 그룹 "admin"을 명시적으로 연결
    ansible.vm.synced_folder "./ansible", "/home/admin/ansible",
      owner: 1234,
      group: "admin",
      mount_options: ["dmode=755", "fmode=644"]

    ansible.vm.provision :shell, privileged: true, inline: $install_default
    ansible.vm.provision :shell, privileged: true, inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      apt-add-repository --yes --update ppa:ansible/ansible
      apt-get install -y ansible

      echo '======== [9] 교차 연동 및 기본 파일 이관 작업 ========'
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