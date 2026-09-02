# Next-Generation Cloud-Native Infrastructure & Observability Platform Based on eBPF (Cilium) and OpenTelemetry

### Overview
_This is an Infrastructure as Code (IaC) Repository that automatically provisions a Kubernetes-based cluster in a local development environment(with Resource Constraints of 16GB RAM & 250GB disk) using Vagrant and Ansible._

**Related Repositories** :
* **Cluster Infra Provisioning** : _[observability-platform-with-cloud-native.git](https://github.com/ddolkwak/observability-platform-with-cloud-native)_
* **GitOps Infrastructure & K8s Manifests** : _[observability-platform-cluster-desired-state.git](https://github.com/ddolkwak/observability-platform-cluster-desired-state)_
* **Spring Boot Application Development** : _[observability-spring-boot-app-development.git](https://github.com/ddolkwak/observability-spring-boot-app-development)_

**Additional Links**
* **Project Scheduling & Management** : [Jira Dashboard](https://ddolkwakpro.atlassian.net/jira/software/projects/OPDWCN/boards/3)
* **Investigation & Study** : [Velog.io/@daankwak/posts](https://velog.io/@daankwak/posts)

### Objectives
_By converging_
* Infrastructure as Code (IaC)
* Modern DevOps Observability Standards (OpenTelemetry)
* Kernel-level Networking (eBPF),

_the goal is to build cloud-native architecture and an observability platform within the resource constraints of local host environment._

_1) Move beyond traditional monitoring architectures and resource-intensive sidecar proxy approaches by establishing a global-standard OpenTelemetry (OTel) telemetry pipeline._

_2) Utilize the Cilium CNI with eBPF capabilities to provide kube-proxy-free observability and control over microservice network topologies and metrics._

_3) Applying Strict Resource Optimization, delve into cluster components, connection principles, etc.(Both Kernel-level & application-level)._

---
### System Architecture
![Diagram1](./architecture-diagram/system-architecture.svg)

### Cluster Provisioning
![Diagram2](./architecture-diagram/k8s-cluster-provisioning.svg)

### GitOps Deployment Pipelines
![Diagram3](./architecture-diagram/gitops-deployment-pipeline.svg)

### Application & Network Routing
![Diagram4](./architecture-diagram/application-and-network-routing.svg)

### Observability & Monitoring
![Diagram5](./architecture-diagram/observability-and-monitoring.svg)

---
### Core Technical Stack (Overall)
* **Infrastructure/IaC** : _Vagrant, Ansible, VirtualBox, Ubuntu (CLI-only)_
* **Orchestration/Runtime** : _Upstream K8s (kubeadm), CRI-O, NFS Provisioner_
* **Networking/eBPF** : _Cilium(Strict mode), MetalLB, CoreDNS_
* **K8s App Definition/Packaging** : _Helm, Kustomize_
* **GitOps/CD** : _ArgoCD Core, GitHub, GHCR(GitHub Container Registry)_
* **Application/Containerization** : _Java Spring Boot, PostgreSQL, Docker_
* **Observability** : _OpenTelemetry Collector, Micrometer, Grafana Cloud(Mimir)_

---
### Key Features
* **IaC Automation** : _Automated the entire infrastructure provisioning and Kubernetes bootstrapping process from scratch using Vagrant and Ansible, achieving fully reproducible local environments._
* **Strict Resource Optimization** : _Custom-tuned OS Kernel, Upstream Kubernetes cluster, JVM Heap Memory, etc. to maximize efficiency within a highly constrained 9GB RAM._
* **Lightweight Container Runtime** : _Implemented CRI-O as the container runtime, for less resource footprint and fit on Kubernetes API model._
* **Declarative Infrastructure** : _Managed the entire cluster infra utilizing GitOps principles with ArgoCD and Kustomize to ensure fully automated and declarative infrastructure lifecycle management._
* **eBPF-powered Advanced Networking** : _Replaced legacy iptables with Cilium (Kube-proxy-less strict mode) for high-performance eBPF-based direct routing, and MetalLB (L2 mode) for external traffic ingress._
* **Minimal-Agent Observability Pipeline** : _Designed a lightweight monitoring architecture using OpenTelemetry Collector as a DaemonSet. Offloaded the heavy TSDB backend to Grafana Cloud (Mimir) to strictly maintain the local resource budget while ensuring full observability._
  * _TSDB Offload to Grafana Cloud : Original plan was to deploy monitoring stacks(prometheus, grafana) on the cluster, fell through due to local host environment resource constraint._
* **Microservice-Oriented Repository Design** : _Logically divided the project into three distinct repositories (Cluster Infrastructure, GitOps Infra & K8s Manifests, and Application Source Code) to separate concerns and independent deployment lifecycles._

---
### Directory Structure
**1) Cluster Infrastructure Provisioning**
```bash
# GitHub Repository (observability-platform-with-cloud-native.git)
Vagrantfile                               # VM Provisioning
/home/admin/                              # Home Directory
      └── ansible/                        # Ansible Home Directory(Symlinked to /etc/ansible)
          ├── ansible.cfg                 # Ansible Configuration
          ├── collections/                # Prerequisite Ansible Collections
          │   └── requirements.yml
          ├── group_vars/                 # Global Variables Mgmt.
          │   └── all.yml
          ├── hosts                       # Inventory
          ├── roles/                      # Deployment Playbooks Based on Ansible Roles
          │   ├── common/
          │   ├── master/
          │   ├── worker/
          │   ├── argocd-cli/
          │   ├── argocd-core/
          │   ├── helm/
          │   ├── metallb/
          │   └── metrics-server/
          │
          ├── k8s_deploy.yml              # K8s Deployment Playbook
          ├── k8s_startup.yml             # Cluster Init
          ├── k8s_shutdown.yml            # Cluster Graceful Shutdown
          └── argocd_deploy.yml           # ArgoCD Deployment Playbook
```
**2) GitOps Infrastructure & K8s Manifests**
```bash
# GitHub Repository (observability-platform-cluster-desired-state.git)
gitops/
├── bootstrap/
│   ├── gitops-project.yaml    # Project Definition to ArgoCD
│   └── root-app.yaml          # ArgoCD Entrypoint
├── clusters/
│   ├── dev/
│   │   ├── infra-apps.yaml    # infra App-of-Apps
│   │   └── work-apps.yaml     # workloads App-of-Apps
│   └── prod/
│       ├── infra-apps.yaml
│       └── work-apps.yaml
├── apps/                      # ArgoCD Application Definition
│   ├── infra/
│   │   ├── priority-class.yaml
│   │   ├── storage.yaml
│   │   └── otel-collector.yaml
│   └── workloads/
│       ├── postgres.yaml
│       └── spring-app.yaml
└── manifests/                 # Kubernetes Resources
    ├── infra/
    │   ├── priority-class/
    │   │   └── base/
    │   │       ├── kustomization.yaml
    │   │       └── priority-classes.yaml
    │   └── otel-collector/
    │       ├── base/
    │       │   ├── kustomization.yaml
    │       │   ├── otel-rbac.yaml
    │       │   ├── otel-collector-config.yaml
    │       │   ├── otel-collector-daemonset.yaml
    │       │   ├── otel-collector-service.yaml
    │       │   └── cilium-local-redirect-policy.yaml
    │       └── overlays/
    │           ├── dev/
    │           └── prod/
    └── workloads/
        ├── postgres/
        │   ├── base/
        │   │   ├── kustomization.yaml
        │   │   ├── deployment.yaml
        │   │   ├── pvc.yaml
        │   │   └── service.yaml
        │   └── overlays/
        │       ├── dev/
        │       └── prod/
        └── spring-app/
            ├── base/
            │   ├── kustomization.yaml
            │   ├── deployment.yaml
            │   └── service.yaml
            └── overlays/
                ├── dev/
                └── prod/
```

---
### How to run

### Prerequisites
For configuring system infrastructure, the following tools should be installed in local environment:
* `VirtualBox` _(v7.2.4)_,
* `Vagrant` _(v2.4.9)_,
* _Latest Version of_ `Git`,
* and IDE(`VSCodium` or whatever)

`winget` can be used in PowerShell, or Comand Prompt to install the required tools.
```bash
# Oracle VirtualBox 7.2.4 Installation
winget install --id Oracle.VirtualBox --version 7.2.4

# Vagrant 2.4.9 Installation
winget install --id Hashicorp.Vagrant --version 2.4.9

# Git (Latest Version) Installation
winget install --id Git.Git

# VSCodium Installation
winget install --id VSCodium.VSCodium
```
_Note: You may need to restart your terminal or reboot your computer after installing VirtualBox and Vagrant for the system path and network drivers to apply correctly._

After installation, verify that the tools are installed correctly and check their versions:
```bash
# Check VirtualBox version
VBoxManage --version
# Check Vagrant version
vagrant --version
# Check Git version
git --version
# Check VSCodium version
codium --version
```
The expected versions are:
```bash
VirtualBox: 7.2.4
Vagrant: 2.4.9
Git: Latest version
```

### Provisioning VMs
**1. Create Workspace**
```bash
# 1) Clone Repository & Create Workspace on Local Environment
git clone https://github.com/ddolkwak/observability-platform-with-cloud-native.git workspace
cd workspace

# 2) Disconnect Git Repository
# Powershell
Remove-Item -Path ".\.git" -Recurse -Force -ErrorAction SilentlyContinue
# Git Bash / Linux
rm -rf ./.git
```

**2. Provision VMs via Vagrant**

Before provisioning VMs on local envirionment, customize VM settings below:

_(i) Hostname_

_(ii) IP Addresses_

_(iii) Allocatable Resouces_ for each VMs.

```bash
vi workspace/Vagrantfile
```

Here is **_default settings_** for Hostname and Allocatable resources.

| VM | CPU | Memory |
|----|-----|------------------|
| `ansible-node` | 1 Core | 1GB (1024MB) |
| `k8s-master` | 2 Core | 3GB (3072MB) |
| `k8s-worker-1` | 2 Core | 2.5GB (2560MB) |
| `k8s-worker-2` | 2 Core | 2.5GB (2560MB) |

Here is an example for Vagrantfile customization.
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  ...

  ...
  # =========================================================
  # [Infra Provisioning] OS Optimazation, Create admin account, SSH Key Deployment
  # =========================================================
    ...

    ...
    echo '======== [8] Register Nodename to /etc/hosts ========'
    if ! grep -q "k8s-master" /etc/hosts; then
      chmod 777 /etc/hosts
      cat << EOF >> /etc/hosts
000.000.000.000 ansible-node    # Set IP Address and Hostname
000.000.000.000 k8s-master      # Set IP Address and Hostname
000.000.000.000 k8s-worker-1    # Set IP Address and Hostname
000.000.000.000 k8s-worker-2    # Set IP Address and Hostname
EOF
      chmod 644 /etc/hosts
    fi
  SHELL

  # =========================================================
  # 1. Ansible Control Plane (ansible-node)
  # =========================================================
  config.vm.define "ansible-node" do |ansible|  # Set IP Address and Hostname
    ansible.vm.hostname = "ansible-node"
    ansible.vm.network "private_network", ip: "000.000.000.000"
    ansible.vm.provider "virtualbox" do |vb|
      vb.name = "ansible-node"
      vb.memory = 1024  # Set Memory and CPU size
      vb.cpus = 1
    end
    ...

    ...
  # =========================================================
  # 2. Kubernetes Master Node (k8s-master)
  # =========================================================
  config.vm.define "k8s-master" do |master|  # Set IP Address and Hostname
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", ip: "000.000.000.000"
    master.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-master"
      vb.memory = 3072  # Set Memory and CPU size
      vb.cpus = 2
    end
    master.vm.provision :shell, privileged: true, inline: $install_default
  end

  # =========================================================
  # 3. Kubernetes Worker Nodes (k8s-worker-1, k8s-worker-2)
  # =========================================================
  (1..2).each do |i|
    config.vm.define "k8s-worker-#{i}" do |worker|  # Set IP Address and Hostname
      worker.vm.hostname = "k8s-worker-#{i}"
      worker.vm.network "private_network", ip: "000.000.000.00#{i}"
      worker.vm.provider "virtualbox" do |vb|
        vb.name = "k8s-worker-#{i}"
        vb.memory = 2560  # Set Memory and CPU size
        vb.cpus = 2
      end
      worker.vm.provision :shell, privileged: true, inline: $install_default
    end
  end
end
```

After Vagrantfile customization, Run these command below to provision VMs:

```bash
vagrant plugin install vagrant-vbguest vagrant-disksize
vagrant up
```

### Setup Ansible
Ansible directory is already cloned in `/vagrant/workspace`, so access Ansible Control Plane and copy it under `/home/admin/`.
```bash
ssh admin@[Ansible_Node_IP_Address]
sudo cp -a /vagrant/workspace/ansible/. /home/admin/ansible/
```
If copy is successful, edit **inventory** file(`hosts`):
```bash
vi ~/ansible/hosts
```
Nodes are grouped into 3 categories.
* `[control]` for Ansible Control Plane,
* `[master]` for Kubernetes Master Node,
* `[workers]` for Kubernetes Worker Nodes

```bash
# Hosts File
[control]
localhost ansible_connection=local

[master]
k8s-master ansible_host=000.000.000.000  # Edit IP Address

[workers]
k8s-worker-1 ansible_host=000.000.000.000  # Edit IP Address
k8s-worker-2 ansible_host=000.000.000.000  # Edit IP Address

[all:vars]
ansible_user=admin
```

### Kubernetes Deployment (kube-adm)
Before kubernetes deployment, Customize global variables settings. Global variables are listed in following path:
```bash
vi ~/ansible/group_vars/all.yml
```
Edit 2 variables below:

_(i) Kubernetes Network Settings(Master Node)_

_(ii) MetalLB Allocatable IP Range_

```yaml
# ========================================================================
# [1] : Kubernetes Global Variables
# ========================================================================
  ...

  ...
# Kubernetes Network Settings
master_node_ip: "000.000.000.000"  # Set K8s Master Node IP Addresses
pod_network_cidr: "172.20.0.0/16"
k8s_service_port: "6443"
  ...

  ...
# ========================================================================
# [6] : MetalLB Global Variables
# ========================================================================
# MetalLB Allocatable IP Range
metallb_ip_pools:
  - "000.000.000.000-000.000.000.000"  # Set IP Address Range
```
Additionally, Container Image Versions can also be modified:
* `kubernetes_version` _(default is v1.30)_
* `crio_version` _(default is v1.30)_
* `helm_version` _(default is v3.20.2)_
* `argocd_version` _(default is v2.10.4)_

After prerequisite global variables are all set up, Run Kubernetes deployment playbook.
```bash
ansible-playbook k8s_deploy.yml
```
Run ArgoCD deployment playbook to deploy ArgoCD.
```bash
ansible-playbook argocd_deploy.yml
```

### Additional Settings (Troubleshoot)

#### 1) Vagrant VM Provisioning Failure

If VM Provisioning fails with couple of error logs below:
```bash
undefined method 'exists?' for class File (NoMethodError)
Did you mean? exist?
```
Edit Vagrant Plugin Source Code (`vagrant-vbguest-0.32.0`) via this path:
```bash
# To specify Home directory, Find where Vagrant is installed.
~\.vagrant.d\gems\3.3.8\gems\vagrant-vbguest-0.32.0\lib\vagrant-vbguest\hosts\virtualbox.rb
```
Change **Line 84**, from `File.exists?(path)` to `File.exist?(path)`.

Here is an example:
```ruby
# Find the first GuestAdditions iso file which exists on the host system
#
# @return [String] Absolute path to the local GuestAdditions iso file, or +nil+ if not found.
def guess_local_iso
  Array(platform_path).find do |path|
    path && File.exist?(path)  # exists? -> exist?
  end
end
```
Run VM Provisioning again.
```bash
vagrant destroy -f
vagrant up
```

#### 2) Outdated Kernel Version For Cilium

This Cluster Uses `Ubuntu 22.04 LTS (Jammy Jellyfish)` as default, and imported kernel is `5.15-generic`.
Cilium will be deployed while running Kubernetes deployment Playbook(`k8s_deploy.yml`), and Kernel version upgrade is required depending on conditions.

In this regard, Kernel Version can be upgraded via playbook in the cluster.
```bash
ansible/.archive/2026-07-24-archived/
```
Simply Run the playbook to upgrade kernel version, from `5.15-generic` to `6.8-generic`.
```bash
ansible-playbook ansible/.archive/2026-07-24-archived/host_kernel_update.yml
```
After Kernel version upgrade, Run Kubernetes deployment playbook again.
```bash
ansible-playbook k8s_deploy.yml
```