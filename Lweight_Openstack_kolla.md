Deploying OpenStack on your laptop with 8 GB RAM using Kolla-Ansible in all-in-one (AIO) mode. though performance will be limited due to RAM constraints.

---

## ✅ Overview

* Tool: Kolla-Ansible (runs OpenStack services in Docker containers)
* System: Ubuntu 22.04 LTS, 8 GB RAM, 200 GB disk
* Deployment mode: All-in-One (single node)
* Target OpenStack version: 2023.1 or latest supported by Kolla

---

## 🛠 Step-by-Step Instructions

---

### 🔹 Step 1: Install Prerequisites

Open terminal and run:

```
sudo apt update && sudo apt upgrade -y
sudo apt install python3-dev libffi-dev gcc libssl-dev python3-pip git -y
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker  # Reload docker group
```

---

### 🔹 Step 2: Install Ansible & Kolla-Ansible

```
pip3 install -U pip
pip3 install 'ansible>=6,<8'
pip3 install kolla-ansible
```

---

### 🔹 Step 3: Setup Kolla-Ansible Configuration

```
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp /usr/local/share/kolla-ansible/ansible/inventory/all-in-one .
```

---

### 🔹 Step 4: Generate Passwords

kolla-genpwd

Optional: Edit /etc/kolla/passwords.yml to set your own keystone_admin_password.

---

### 🔹 Step 5: Configure Global Settings

Open globals.yml:

```
nano /etc/kolla/globals.yml
```

Add or modify:

```
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "2023.1"
kolla_internal_vip_address: "127.0.0.1"
network_interface: "eth0"  # Run `ip a` to confirm your main interface
neutron_plugin_agent: "openvswitch"
enable_horizon: "yes"
enable_cinder: "no"
enable_heat: "no"
enable_sahara: "no"
```

Tip: Turn off unused services to save RAM.

---

### 🔹 Step 6: Bootstrap Your Local Node

```
kolla-ansible -i all-in-one bootstrap-servers
```

---

### 🔹 Step 7: Run Prechecks and Pull Images

```
kolla-ansible -i all-in-one prechecks
kolla-ansible -i all-in-one pull
```

This step pulls Docker images (about 5–10 GB).

---

### 🔹 Step 8: Deploy OpenStack

```
kolla-ansible -i all-in-one deploy
```

If successful, you’ll see TASK [Deployment completed].

---

### 🔹 Step 9: Setup OpenStack CLI Access

```
source /etc/kolla/admin-openrc.sh
```

Check if it's working:

```
openstack service list
```

---

### 🔹 Step 10: Access Horizon Dashboard


Get your IP:

```
echo "http://$(hostname -I | awk '{print $1}')/"
```
Login with:

* Username: admin
* Password: Found in /etc/kolla/passwords.yml → keystone_admin_password


---

## ✅ Optional: Clean Up

To remove all containers and data:

```
kolla-ansible -i all-in-one destroy --yes-i-really-really-mean-it
```

---

## 🧠 Tips to Save RAM

* Disable services like Cinder, Heat, Sahara.
* Use minimal services (Keystone, Glance, Nova, Neutron, Horizon).
* Consider running the deployment overnight due to image pulls.