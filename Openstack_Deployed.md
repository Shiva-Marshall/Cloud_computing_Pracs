# 🏗️ OpenStack on EC2 — Setup Guide

---

## 📌 What You'll Set Up

You'll install OpenStack using **DevStack** on a Ubuntu-based EC2 instance. After installation, you’ll access the OpenStack Dashboard (Horizon) from your browser to create virtual machines and manage cloud resources.

---

## 🧰 Prerequisites

### 📦 EC2 Instance Configuration

* **AMI**: Ubuntu 22.04 LTS
* **Instance Type**: `t2.large` (2 vCPUs, 8GB RAM minimum)
* **Storage**: At least 30 GB
* **Security Group**:

  * **Port 22 (SSH)**: Your IP
  * **Port 80, 443**: Open (for dashboard access)
* **Key Pair**: Use an existing or create a new one for SSH

---

## 🔐 Step 1: Connect to EC2 via SSH

```bash
ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>
```

---

## 🔄 Step 2: Update System & Install Essentials

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3-pip python3-dev net-tools
```

---

## 📥 Step 3: Clone DevStack

```bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

---

## ⚙️ Step 4: Configure DevStack (`local.conf`)

Create a config file:

```bash
nano local.conf
```

Paste the following:

```ini
[[local|localrc]]
ADMIN_PASSWORD=SuperSecret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

HOST_IP=$(hostname -I | awk '{print $1}')

disable_service tempest
disable_service cinder
disable_service swift
disable_service heat

USE_PYTHON3=True
```

---

## ▶️ Step 5: Run the Installer

```bash
./stack.sh
```

⚠️ **Takes \~30–45 mins.** Don't interrupt.
✅ You’ll see a success message when it finishes.

---

## 🌐 Step 6: Access OpenStack Dashboard (Horizon)

* Open your browser and go to:
  `http://<EC2-PUBLIC-IP>/dashboard`

* Login:

  * Username: `admin`
  * Password: `SuperSecret` (as defined in `local.conf`)

---

## ✅ Step 7: Verify Services

Run this after sourcing environment:

```bash
source openrc admin admin
openstack service list
```

You should see a list of running OpenStack services (Nova, Glance, Keystone, etc.).

---

## 🛠️ Optional: Restart Services

If a service is down:

```bash
sudo systemctl restart devstack@<service-name>
```

Example:

```bash
sudo systemctl restart devstack@keystone
```

---

## 📌 Notes

* **This setup is for educational/demo purposes only.**
* EC2 virtualization (inside AWS) may limit some OpenStack features (like nested virtualization).
* Avoid using this for production workloads.

---

## 📚 References

* [DevStack GitHub](https://opendev.org/openstack/devstack)
* [OpenStack Docs](https://docs.openstack.org/)
* [AWS EC2 Docs](https://docs.aws.amazon.com/ec2/)