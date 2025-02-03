---
layout: post
title: "Handling Cisco Configuration Files with Python"
date: 2025-02-03 08:25:00 +0000
categories: [Networking, Automation]
tags: [Networking, Automation, Python]
---

Cisco devices store configurations as plain text files, making them easy to read, modify, and analyze with Python. Whether you’re managing routers, switches, or firewalls, automating configuration parsing can **save time, reduce errors, and enforce consistency**.  

In this post, we’ll cover:  
- **How to read, write, and search text-based Cisco configurations**  
- **How to extract and modify interface settings**  
- **How to analyze configurations for security compliance (e.g., SSH, shutdown interfaces, and Telnet settings)**  

Let’s get started! 🚀  

---

## **1. Handling Text Files in Python (Basics)**  

Before working with Cisco configs, let’s cover the basics of handling text files in Python.  

### **Reading a Cisco Config File**  
```python
def read_config(file_path):
    """Reads a Cisco configuration file and returns a list of lines."""
    with open(file_path, 'r') as file:
        lines = file.readlines()
    return lines

config_lines = read_config("cisco_config.txt")
print(config_lines[:10])
```
This function reads a configuration file line by line and stores it in a list.  

---

### **Writing to a Cisco Config File**  
If you need to save modified configurations, use this function:  

```python
def write_config(file_path, lines):
    """Writes modified configuration back to a file."""
    with open(file_path, 'w') as file:
        file.writelines("\n".join(lines))

write_config("modified_cisco_config.txt", config_lines)
```

---

### **Searching for Specific Commands**  
Sometimes, you may need to check if a **certain command is present** in a configuration file.  

```python
def search_config(lines, keyword):
    """Searches for a keyword"""
    return [line.strip() for line in lines if keyword in line]

ssh_config = search_config(config_lines, "ip ssh version")
print(ssh_config)
```
This helps verify whether a feature like **SSH or SNMP** is configured.  

---

## **2. Extracting Cisco Interface Configurations**  

Cisco configurations often contain multiple interface settings. Let’s extract all interfaces from a config file.  

```python
def extract_interfaces(config_lines):
    """Extracts interface configurations from a Cisco config."""
    interfaces = {}
    current_interface = None

    for line in config_lines:
        line = line.strip()
        if line.startswith("interface"):
            current_interface = line.split()[1]
            interfaces[current_interface] = []
        elif current_interface and not line.startswith("!"):
            interfaces[current_interface].append(line)
    
    return interfaces

interfaces = extract_interfaces(config_lines)
print(interfaces)
```
This function organizes **all interface settings** into a dictionary.  

---

## **3. Modifying an Interface Configuration**  

Let’s say we want to **add a description to an interface** without changing other settings.  

```python
def modify_interface_description(config_lines, interface_name, description):
    """Adds or modifies the description of a given interface."""
    modified_config = []
    in_interface = False

    for line in config_lines:
        if line.startswith(f"interface {interface_name}"):
            in_interface = True
            modified_config.append(line)
        elif in_interface and line.strip().startswith("description"):
            modified_config.append(f" description {description}")
            in_interface = False
        elif in_interface and line.strip() == "!":
            modified_config.append(f" description {description}")
            modified_config.append(line)
            in_interface = False
        else:
            modified_config.append(line)

    return modified_config

updated_config = modify_interface_description(config_lines, "GigabitEthernet1", "New uplink")
write_config("updated_cisco_config.txt", updated_config)
```
This script finds the **interface block**, adds a description if missing, or updates it if already present.  

---

## **4. Filtering Configurations for Compliance & Security**  

Now, let’s **analyze** Cisco configurations to check for potential issues.  

### **Checking How Many Interfaces Are Shut Down**  
```python
def count_shutdown_interfaces(config_lines):
    """Counts the number of shutdown interfaces in a Cisco config."""
    shutdown_count = 0
    for line in config_lines:
        if "shutdown" in line:
            shutdown_count += 1
    return shutdown_count

shutdown_count = count_shutdown_interfaces(config_lines)
print(f"Number of shutdown interfaces: {shutdown_count}")
```
This helps identify devices with too many **disabled interfaces**.  

---

### **Checking If SSH Is Configured**  
Secure Shell (SSH) is essential for secure device management.  

```python
def is_ssh_configured(config_lines):
    """Checks if SSH is enabled in the Cisco config."""
    ssh_commands = ["ip ssh version", "line vty", "transport input ssh"]
    for line in config_lines:
        if any(cmd in line for cmd in ssh_commands):
            return True
    return False

ssh_enabled = is_ssh_configured(config_lines)
print(f"SSH Configured: {ssh_enabled}")
```
This script **verifies SSH settings** to prevent insecure Telnet access.  

---

### **Checking for Telnet Access**  
Telnet is insecure and should be disabled in modern networks.  

```python
def is_telnet_enabled(config_lines):
    """Checks if Telnet is enabled on VTY lines."""
    for line in config_lines:
        if "transport input telnet" in line:
            return True
    return False

telnet_enabled = is_telnet_enabled(config_lines)
print(f"Telnet Enabled: {telnet_enabled}")
```
If Telnet is enabled, you might need to **remove or modify the config** to enforce security best practices.  

---

### **Checking for Incomplete Security Settings**  
We can also **combine checks** for SSH and Telnet to generate a compliance report.  

```python
def check_security_compliance(config_lines):
    """Checks if Telnet is disabled and SSH is enabled."""
    telnet_found = is_telnet_enabled(config_lines)
    ssh_found = is_ssh_configured(config_lines)

    if telnet_found and not ssh_found:
        return "Warning: Telnet is enabled but SSH is missing!"
    elif not ssh_found:
        return "Warning: SSH is not configured!"
    else:
        return "Security settings are compliant."

security_status = check_security_compliance(config_lines)
print(security_status)
```
This script helps **enforce best security practices** across multiple devices.  

---

## **5. Automating Cisco Configuration Checks**  
We can **combine** these functions into a single automation script.  

```python
def analyze_cisco_config(file_path):
    """Analyzes a Cisco config file for key parameters."""
    config_lines = read_config(file_path)
    shutdown_count = count_shutdown_interfaces(config_lines)
    ssh_configured = is_ssh_configured(config_lines)
    telnet_enabled = is_telnet_enabled(config_lines)
    security_status = check_security_compliance(config_lines)

    print(f"Shutdown Interfaces: {shutdown_count}")
    print(f"SSH Configured: {ssh_configured}")
    print(f"Telnet Enabled: {telnet_enabled}")
    print(security_status)

analyze_cisco_config("cisco_config.txt")
```
This script **automates multiple configuration checks** in a single function.  


This is just the beginning! In future posts, we’ll cover **Juniper** and **Arista** configurations.  
