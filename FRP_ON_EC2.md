# 🚀 Deploying FRP Server on AWS EC2

This guide walks you through setting up an **FRP (Fast Reverse Proxy) server** on an **Ubuntu-based EC2 instance**. Once configured, you'll be able to securely expose internal services (like a local web app or SSH) to the public internet—even behind firewalls or NAT.

---

## 🔧 Requirements

Before you begin, make sure you have:

* An **AWS EC2 instance** running Ubuntu 20.04/22.04
* An **SSH key pair** to connect to your instance
* A basic understanding of Linux and terminal commands
* FRP client ready (to use from your local or remote device)

---

## ☁️ Step 1: Set Up Your EC2 Server

1. **Launch a new EC2 instance**:

   * **AMI**: Ubuntu 22.04 LTS
   * **Instance Type**: `t2.micro` (or higher)
   * **Storage**: At least 8 GB recommended
   * **Security Group**:

     * Port 22 (SSH) – Your IP
     * Port 7000 – FRP control channel
     * Ports 7500–7600 – For dashboard/proxy services

2. **SSH into your server**:

```bash
ssh -i your-key.pem ubuntu@<ec2-public-ip>
```

---

## 📥 Step 2: Install FRP

1. **Download the latest FRP package**:

```bash
wget https://github.com/fatedier/frp/releases/download/v0.58.0/frp_0.58.0_linux_amd64.tar.gz
tar -xvzf frp_0.58.0_linux_amd64.tar.gz
cd frp_0.58.0_linux_amd64
```

2. **Configure the FRP server**:

Create/edit `frps.ini`:

```ini
[common]
bind_port = 7000
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = changeme123
```

💡 *(Optional)*: Add HTTP/S ports if needed:

```ini
vhost_http_port = 8080
vhost_https_port = 8443
```

---

## ▶️ Step 3: Start the FRP Server

Run it in the foreground for testing:

```bash
./frps -c frps.ini
```

Run it in the background:

```bash
nohup ./frps -c frps.ini &
```

You can also daemonize it with `systemd` (ask me if you want help with that).

---

## 🌐 Step 4: Connect with a Client

On your local device (FRP client machine):

1. Download the matching FRP version.
2. Create a config file called `frpc.ini`:

```ini
[common]
server_addr = <your-ec2-public-ip>
server_port = 7000

[expose-ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

3. Start the client:

```bash
./frpc -c frpc.ini
```

Now you can SSH into your client through the EC2:

```bash
ssh -p 6000 user@<ec2-public-ip>
```

---

## 🔒 Security Notes

* Avoid using weak credentials in your FRP dashboard.
* Only open necessary ports on the EC2 instance.
* Use private IP whitelisting or firewalls when possible.

---

## 📘 References

* 🌍 [FRP on GitHub](https://github.com/fatedier/frp)
* 📖 [FRP Docs](https://github.com/fatedier/frp/blob/dev/README.md)
* ☁️ [AWS EC2 Docs](https://docs.aws.amazon.com/ec2/)

