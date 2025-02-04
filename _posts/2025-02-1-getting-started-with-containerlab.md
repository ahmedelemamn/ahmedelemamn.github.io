---
layout: post
title: "Getting Started with Containerlab"
date: 2025-02-01 00:00:00 +0000
categories: [Networking, Lab]
tags: [Networking, containerlab]
---

## What is Containerlab?  
if you landed on this page then you probably know what containerlab is but just in case, [Containerlab](https://containerlab.dev/) is an open-source tool that simplifies the deployment of container-based networking labs. It uses **Docker** and **containerized network operating systems (NOS)** like **FRR, Nokia SR Linux, Arista cEOS, Cisco NX-OS, and SONiC** to create realistic network environments.

### Why Use Containerlab?  
✅ **Lightweight** – Uses containers instead of full VMs  
✅ **Faster Deployment** – Start up a lab in seconds  
✅ **Multi-Vendor Support** – Cisco, Arista, Nokia, Juniper, and more  
✅ **Reproducibility** – YAML-based configuration for easy sharing  
✅ **Integration with Automation** – Works well with Ansible, Netconf, and gRPC  

---

## Installing Containerlab  
Containerlab requires **Docker** and **sudo/root privileges** to run.  

### 📌 Step 1: Install Docker  
Containerlab runs on top of Docker, so ensure you have it installed:

```bash
sudo apt update && sudo apt install -y docker.io
```
Verify Docker is running:  
```bash
docker --version
```

### 📌 Step 2: Install Containerlab  
The easiest way to install Containerlab is via the official script:

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```
Alternatively, install manually using `deb` or `rpm` packages:

```bash
curl -sL https://github.com/srl-labs/containerlab/releases/latest/download/containerlab-linux-amd64 -o /usr/local/bin/containerlab
chmod +x /usr/local/bin/containerlab
```

### 📌 Step 3: Verify Installation  
After installation, check if Containerlab is working:

```bash
containerlab version
```

---

## Understanding Containerlab Topology Files  
Containerlab uses a **YAML-based topology definition** file. This file defines **nodes (devices), links, and management settings**.

### 🌍 Basic Structure of a Containerlab Topology File  
```yaml
name: my-first-lab

topology:
  nodes:
    router1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:latest

    router2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:latest

  links:
    - endpoints: ["router1:eth1", "router2:eth1"]
```

### 📌 Key Sections:
1. **name** – Name of the lab  
2. **topology** – Defines all devices and links  
3. **nodes** – Lists the devices (e.g., routers, switches)  
4. **kind** – Specifies the device type (e.g., `vr-ros` for RouterOS, `ceos` for Arista EOS)  
5. **image** – The Docker image used  
6. **links** – Defines connections between devices  

---

## Deploying Your First Lab  
Now, let's **deploy and manage** a simple lab with two routers.

### 📌 Step 1: Define the Topology  
Create a new directory for your lab and navigate into it:

```bash
mkdir ~/containerlab-labs && cd ~/containerlab-labs
```

Create a new YAML file (`mylab.clab.yml`) and add the following:

```yaml
name: simple-lab

topology:
  nodes:
    r1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:latest

    r2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:latest

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
```

### 📌 Step 2: Deploy the Lab  
Run the following command to deploy the lab:

```bash
sudo containerlab deploy -t mylab.clab.yml
```

Once deployed, list running labs:

```bash
containerlab inspect
```

To see the running Docker containers:

```bash
docker ps
```

---

## Logging into Nodes  
Once the lab is deployed, you can access the network devices using different methods.

### 🛠 Method 1: Using `docker exec`  
Each node runs as a Docker container. To access a node’s shell:

```bash
docker exec -it r1 vtysh
```
or for a Bash shell:
```bash
docker exec -it r1 /bin/bash
```

For routers running Cisco’s IOS-XE or Arista EOS:
```bash
docker exec -it r1 Cli
```

### 🛠 Method 2: Using `containerlab inspect`  
Get the **management IP** of a node:
```bash
containerlab inspect -t mylab.clab.yml
```
Find the IP under `mgmt_ipv4` and SSH into it:

```bash
ssh admin@<MGMT-IP>
```

Example:
```bash
ssh admin@172.20.20.2
```

### 🛠 Method 3: Directly Connecting via Terminal  
For NOS images like **Arista cEOS**, **Cisco NX-OS**, or **Nokia SR Linux**, they expose a **default SSH port**. You can SSH using:

```bash
ssh -p 22 admin@$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' r1)
```

For containers exposing **console ports** via **telnet**, check with:
```bash
docker ps | grep r1
```
Then connect using:
```bash
telnet 127.0.0.1 <PORT>
```

---

## Stopping and Deleting Labs  
Once you're done, **clean up the environment** with:

```bash
sudo containerlab destroy -t mylab.clab.yml
```

This will **stop and remove** the containers while keeping the YAML file intact.
