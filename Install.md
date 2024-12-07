
# Vagrantfile to Start Two VMs and Set Up Kubernetes 1.26.6 with CRI-O 1.26.3

This guide walks you through setting up two virtual machines using Vagrant, which will host a Kubernetes cluster running version 1.26.6 with CRI-O 1.26.3 as the container runtime.

---

## **Vagrantfile**

The `Vagrantfile` contains all the necessary configuration for provisioning two VMs and setting up the Kubernetes environment. 

### **Check the File**

Ensure you have saved the file as `Vagrantfile` in your working directory. It should include the required configuration for setting up both VMs and installing Kubernetes components with CRI-O.

---

## **Steps to Run the Script**

### **Prerequisites**

Before running the script, make sure you have:
1. [Vagrant](https://www.vagrantup.com/) installed on your system.
2. [VirtualBox](https://www.virtualbox.org/) as the provider for Vagrant.

### **Provision the Virtual Machines**

1. Save the `Vagrantfile` in your working directory.
2. Open a terminal, navigate to the directory, and run:
   ```bash
   vagrant up
   ```
   This command provisions two VMs (`k8s-node-1` and `k8s-node-2`), installs Kubernetes 1.26.6, and sets up CRI-O 1.26.3.

---

## **Post-Setup**

Once the VMs are running, follow the steps below to initialize and configure the Kubernetes cluster.

### **Step 1: Initialize the Cluster**

1. SSH into the first VM:
   ```bash
   vagrant ssh k8s-node-1
   ```
2. Initialize the Kubernetes cluster:
   ```bash
   sudo kubeadm init \
       --apiserver-advertise-address=192.168.56.11 \
       --apiserver-cert-extra-sans=192.168.56.11 \
       --pod-network-cidr=10.244.0.0/16 --upload-certs
   ```

3. Follow the output instructions to configure `kubectl` for the current user:
   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   export KUBECONFIG=/etc/kubernetes/admin.conf
   ```

4. Save the Kubernetes configuration file for use on the host machine if needed:
   ```bash
   sudo cp /etc/kubernetes/admin.conf /vagrant/config
   ```

---

### **Step 2: Join the Second Node**

1. SSH into the second VM:
   ```bash
   vagrant ssh k8s-node-2
   ```
2. Use the command provided by `kubeadm init` to join the second node to the cluster. For example:
   ```bash
   sudo kubeadm join 192.168.56.11:6443 --token <TOKEN_FROM_PREVIOUS_STEP> \
       --discovery-token-ca-cert-hash sha256:<SHA256_FROM_PREVIOUS_STEP>
   ```

Replace `<TOKEN_FROM_PREVIOUS_STEP>` with the token value provided during the cluster initialization.
Replace `<SHA256_FROM_PREVIOUS_STEP>` with the SHA256 value provided during the cluster initialization.

---

## **Verifying the Setup**

1. SSH back into the first VM (`k8s-node-1`):
   ```bash
   vagrant ssh k8s-node-1
   ```
2. Verify that both nodes are successfully added to the cluster:
   ```bash
   kubectl get nodes
   ```
   The output should show both `k8s-node-1` and `k8s-node-2` with a `Ready` status.

---

## **Additional Commands**

- Restart services if needed:
  ```bash
  sudo systemctl daemon-reload && sudo systemctl restart kubelet
  ```
- Apply a pod network plugin, such as Flannel, for networking:
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  ```

---

## **Summary**

This setup allows you to run a Kubernetes cluster locally on two VMs, with CRI-O as the container runtime. It's ideal for testing and development purposes. Customize the `Vagrantfile` or the Kubernetes configuration to suit your requirements.
