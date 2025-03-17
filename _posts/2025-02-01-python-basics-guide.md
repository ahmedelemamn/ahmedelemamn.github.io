---
layout: post
title: "Python Basics: A One-Stop Reference Guide"
date: 2025-02-02 00:00:00 +0000
categories: [Networking, Automation]
tags: [Networking, Automation, Python]
---

Cisco routers and switches rely on configuration files to set up and manage network behavior. In this guide, we’ll start with a sample Cisco IOS configuration file and build Python examples around it. This approach will help you understand how Python can automate tasks like configuration parsing, interface management, and error handling.

---

## **Cisco IOS Configuration Example**
Below is a simplified configuration file for Router1:
```cisco
!
! Cisco IOS Configuration Example
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
!
hostname Router1
!
interface GigabitEthernet0/0
 description LAN Interface
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description WAN Interface
 ip address 10.0.0.1 255.255.255.252
 shutdown
!
ip route 0.0.0.0 0.0.0.0 10.0.0.2
!
line vty 0 4
 login local
 transport input ssh
!
end
```

---

## **1. Variables & Data Types**
Variables store configuration details for later processing. For example, you can extract the hostname or interface details from the config.

### **Example**
```python
# Extracted values from the Cisco config
hostname = "Router1"       # str
lan_ip = "192.168.1.1"     # str (from GigabitEthernet0/0)
wan_ip = "10.0.0.1"        # str (from GigabitEthernet0/1)
wan_status = False         # bool (False for 'shutdown')

# Display extracted data
print(f"Hostname: {hostname}")
print(f"LAN IP: {lan_ip}")
print(f"WAN IP: {wan_ip}")
print("WAN interface is active" if not wan_status else "WAN interface is shutdown")
```

### **Use Cases**
- **Storing Config Values:** Manage critical data like IP addresses and interface statuses.

---

## **2. Strings & String Methods**
Cisco config lines are plain text. Python’s string methods make it easy to parse these lines.

### **Example**
```python
config_line = " ip address 192.168.1.1 255.255.255.0"
parts = config_line.split()
ip_address = parts[2]
subnet_mask = parts[3]
print("IP Address:", ip_address)
print("Subnet Mask:", subnet_mask)
```

### **Use Cases**
- **Config Parsing:** Extract IP addresses, subnet masks, or descriptions from configuration lines.

---

## **3. Lists (Dynamic Arrays)**
Lists help manage multiple configuration entries, such as interface configurations.

### **Example**
```python
# List of interface configurations extracted from the Cisco config
interfaces = [
    "GigabitEthernet0/0: ip address 192.168.1.1 255.255.255.0, no shutdown",
    "GigabitEthernet0/1: ip address 10.0.0.1 255.255.255.252, shutdown"
]

# Add another interface (e.g., a new VLAN interface)
interfaces.append("Vlan10: ip address 172.16.0.1 255.255.255.0, no shutdown")
for interface in interfaces:
    print(interface)
```

### **Use Cases**
- **Managing Interfaces:** Iterate over interfaces for status checks or bulk updates.

---

## **4. Tuples (Immutable Lists)**
Tuples can store configuration values that shouldn’t change, such as default routing parameters.

### **Example**
```python
# Tuple for default route: (destination, subnet, next-hop)
default_route = ("0.0.0.0", "0.0.0.0", "10.0.0.2")
print("Default Route:", default_route)
```

### **Use Cases**
- **Immutable Settings:** Keep important parameters constant throughout your script.

---

## **5. Dictionaries (Key-Value Pairs)**
Dictionaries are ideal for organizing configuration details into structured data.

### **Example**
```python
# Dictionary representing an interface configuration
interface_config = {
    "name": "GigabitEthernet0/0",
    "description": "LAN Interface",
    "ip_address": "192.168.1.1",
    "subnet_mask": "255.255.255.0",
    "status": "no shutdown"
}
print(interface_config)
```

### **Use Cases**
- **Structured Config Data:** Quickly retrieve and update device settings by key.

---

## **6. Conditional Statements (if-elif-else)**
Conditional statements allow you to take action based on configuration data.

### **Example**
```python
# Check the status of the WAN interface based on config data
wan_interface_status = "shutdown"  # Extracted from the config
if wan_interface_status == "no shutdown":
    print("WAN interface is active.")
elif wan_interface_status == "shutdown":
    print("WAN interface is shutdown.")
else:
    print("Unknown WAN interface status.")
```

### **Use Cases**
- **Status Verification:** Validate if configurations meet the expected criteria.

---

## **7. Loops**
Loops help automate the process of parsing or updating multiple configuration lines.

### **Example**
```python
# Loop through config lines to find all 'ip address' entries
config_lines = [
    "interface GigabitEthernet0/0",
    " description LAN Interface",
    " ip address 192.168.1.1 255.255.255.0",
    " no shutdown",
    "interface GigabitEthernet0/1",
    " description WAN Interface",
    " ip address 10.0.0.1 255.255.255.252",
    " shutdown"
]

for line in config_lines:
    if "ip address" in line:
        print("Found:", line.strip())
```

### **Use Cases**
- **Bulk Parsing:** Process configuration files to extract multiple entries.

---

## **8. Functions**
Encapsulate repetitive tasks like extracting configuration details into reusable functions.

### **Example**
```python
def extract_ip(config_line):
    parts = config_line.split()
    if "ip" in parts and "address" in parts:
        return parts[2]
    return None

# Example usage:
line = " ip address 192.168.1.1 255.255.255.0"
ip = extract_ip(line)
print("Extracted IP:", ip)
```

### **Use Cases**
- **Automation Scripts:** Reuse functions to ensure consistent parsing logic.

---

## **9. Classes & Objects (OOP)**
Use classes to model network devices or interfaces, making your scripts more modular.

### **Example**
```python
class CiscoInterface:
    def __init__(self, name, description, ip_address, subnet_mask, status):
        self.name = name
        self.description = description
        self.ip_address = ip_address
        self.subnet_mask = subnet_mask
        self.status = status

    def display(self):
        return f"{self.name}: {self.description} - {self.ip_address} {self.subnet_mask} ({self.status})"

# Create an instance based on config data
lan_interface = CiscoInterface("GigabitEthernet0/0", "LAN Interface", "192.168.1.1", "255.255.255.0", "no shutdown")
print(lan_interface.display())
```

### **Use Cases**
- **Object Modeling:** Represent device components to facilitate automation and management.
- **Extensibility:** Easily add methods to push updates or retrieve additional data.

---

## **10. Exception Handling**
Error handling is crucial when dealing with real-world configuration data where errors may occur.

### **Example**
```python
try:
    # Simulate an invalid config line extraction
    config_line = " ip address not_valid"
    parts = config_line.split()
    ip_address = parts[2]
    if len(ip_address.split('.')) != 4:
        raise ValueError("Invalid IP address format")
except ValueError as e:
    print("Error:", e)
```

### **Use Cases**
- **Robust Automation:** Ensure your script can handle unexpected config formats without crashing.

---

## **11. File Handling**
Often, you’ll work with configuration files saved on disk. Python makes reading and writing these files straightforward.

### **Example**
```python
# Reading a Cisco configuration file
with open("router_config.txt", "r") as config_file:
    config_data = config_file.read()

# Print the first 200 characters of the configuration
print(config_data[:200])
```

### **Use Cases**
- **Configuration Backups:** Automate reading and storing configuration files for audits or restoration.
- **Bulk Processing:** Load configurations for further analysis or automated modifications.

---

## **12. List Comprehensions**
List comprehensions provide a concise way to process configuration data.

### **Example**
```python
# Extract all IP addresses from the configuration file using a list comprehension
config_lines = config_data.splitlines()
ip_addresses = [line.split()[2] for line in config_lines if line.strip().startswith("ip address")]
print("Extracted IP Addresses:", ip_addresses)
```

### **Use Cases**
- **Data Extraction:** Quickly filter and transform configuration lines for reporting.

---

## **13. Modules & Imports**
Python’s ecosystem includes libraries like **netmiko** that allow direct interaction with Cisco devices. The following example illustrates how you might connect to a device (ensure netmiko is installed and properly configured before using).

### **Example**
```python
# Example using netmiko (uncomment and configure as needed)
# from netmiko import ConnectHandler
#
# device = {
#     'device_type': 'cisco_ios',
#     'host': '192.168.1.1',
#     'username': 'admin',
#     'password': 'password',
# }
#
# connection = ConnectHandler(**device)
# output = connection.send_command("show ip interface brief")
# print(output)
```

### **Use Cases**
- **Direct Device Interaction:** Execute commands on Cisco devices for real-time data.
- **Advanced Automation:** Leverage specialized libraries for configuration changes and monitoring.