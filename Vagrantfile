Vagrant.configure("2") do |config|
    (1..2).each do |i|
      config.vm.define "k8s-node-#{i}" do |node|
        node.vm.box = "ubuntu/jammy64" # Ubuntu 22.04 LTS
        node.vm.hostname = "k8s-node-#{i}"
        # Configure private network
        node.vm.network "private_network", ip: "192.168.56.1#{i}"
        
        node.vm.provider "virtualbox" do |vb|
          vb.memory = "2048"
          vb.cpus = 2
        end
        # Forward port 6443 only for k8s-node-1
        if i == 1
          node.vm.network "forwarded_port", guest: 6443, host: 6443
        end
        node.vm.provision "shell", inline: <<-SHELL
          # Update and install dependencies
          sudo apt-get update -y
          sudo apt-get install -y curl gnupg2 software-properties-common
  
          # Add CRI-O repository
          OS="xUbuntu_22.04"
          VERSION="1.26"
          echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"| sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
          echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.26:/1.26.3/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list
          curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:/1.26:/1.26.3/$OS/Release.key | sudo apt-key add -
          curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -
  
          # Install CRI-O
          sudo apt-get update 
          sudo apt install cri-o cri-o-runc -y
          sudo systemctl enable crio --now

          sudo apt install containernetworking-plugins -y
          sudo apt install -y cri-tools
   
          # Install Kubernetes components
          sudo apt-get update -y
          sudo apt-get install -y snapd curl software-properties-common

          # Install kubeadm, kubelet, and kubectl via Snap
          curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
          # sudo echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.26/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
          cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.26/deb/ /
EOF
          sudo apt update
          sudo apt install kubeadm=1.26.6-1.1 kubelet=1.26.6-1.1 kubectl=1.26.6-1.1 -y
          sudo apt-mark hold kubeadm kubelet kubectl

          # or using snap in case the above repo are gone x_x 
          # sudo snap install kubeadm --classic --channel=1.26/stable
          # sudo snap install kubectl --classic --channel=1.26/stable
          # sudo snap install kubelet --classic --channel=1.26/stable

          # Enable and start kubelet
          # sudo systemctl enable snap.kubelet.daemon
          # sudo systemctl start snap.kubelet.daemon

          # Disable swap
          sudo swapoff -a
          sudo sed -i '/ swap / s/^/#/' /etc/fstab
  
          # Load necessary kernel modules
          sudo modprobe overlay
          sudo modprobe br_netfilter

          cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
          # Configure sysctl
          cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
          sudo sysctl --system
          cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=cgroupfs"
EOF


  
          echo "Setup complete on k8s-node-#{i}"
        SHELL
      end
    end
  end
  
