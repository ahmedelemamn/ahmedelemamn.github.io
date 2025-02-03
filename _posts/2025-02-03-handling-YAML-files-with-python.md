---
layout: post
title: "Handling YAML Files with Python"
date: 2025-02-03 09:00:00 +0000
categories: [Networking, Automation]
tags: [Networking, Automation, Python, yaml]
---

YAML (YAML Ain’t Markup Language) is a human-readable data format widely used for configuration files and data serialization. In Python, handling YAML files is straightforward using the **PyYAML** library. This guide will cover reading, writing, updating, and converting YAML data in Python.

---

## **1. Installing PyYAML**
Before working with YAML in Python, install the PyYAML package:

```bash
pip install pyyaml
```

Once installed, you can start using it in your scripts.

---

## **2. Reading YAML Files**
To read and parse a YAML file, use the `yaml.safe_load()` function.

### **Example YAML File (`config.yaml`):**
```yaml
app:
  name: MyApplication
  version: 1.0
  debug: true

database:
  host: localhost
  port: 5432
  user: admin
  password: secret
```

### **Python Code to Read YAML:**
```python
import yaml

# Load YAML file
with open("config.yaml", "r") as file:
    config = yaml.safe_load(file)

# Print the parsed YAML data
print(config)
print(f"App Name: {config['app']['name']}")
print(f"Database Host: {config['database']['host']}")
```

### **Output:**
```python
{
  'app': {'name': 'MyApplication', 'version': 1.0, 'debug': True},
  'database': {'host': 'localhost', 'port': 5432, 'user': 'admin', 'password': 'secret'}
}
App Name: MyApplication
Database Host: localhost
```
> 🔹 **Tip:** `yaml.safe_load()` prevents arbitrary code execution, making it safer than `yaml.load()`.

---

## **3. Writing YAML Files**
To write a Python dictionary into a YAML file, use `yaml.dump()`.

### **Python Code to Write YAML:**
```python
import yaml

# Data to be written in YAML format
data = {
    "server": {
        "host": "192.168.1.100",
        "port": 8080
    },
    "users": [
        {"name": "Alice", "role": "admin"},
        {"name": "Bob", "role": "user"}
    ]
}

# Writing data to YAML file
with open("output.yaml", "w") as file:
    yaml.dump(data, file, default_flow_style=False)

print("YAML file created successfully!")
```

### **Generated `output.yaml` File:**
```yaml
server:
  host: 192.168.1.100
  port: 8080
users:
  - name: Alice
    role: admin
  - name: Bob
    role: user
```

> 🔹 **Note:** `default_flow_style=False` ensures that lists and dictionaries are formatted in a readable structure.

---

## **4. Updating YAML Files**
To update a YAML file, you must first read it, modify the data, and write it back.

### **Python Code to Update a YAML File:**
```python
import yaml

# Read the existing YAML file
with open("config.yaml", "r") as file:
    config = yaml.safe_load(file)

# Modify the YAML data
config["app"]["version"] = 2.0
config["database"]["password"] = "new_secret"

# Write the updated data back to the file
with open("config.yaml", "w") as file:
    yaml.dump(config, file, default_flow_style=False)

print("YAML file updated successfully!")
```

> 🔹 **Note:** Be careful while overwriting YAML files. Always keep a backup if necessary.

---

## **5. Handling Complex YAML Structures**
YAML supports advanced features like **nested lists, dictionaries, and references**.

### **Example Complex YAML (`complex.yaml`):**
```yaml
defaults: &defaults
  timeout: 30
  retries: 3

server:
  <<: *defaults
  host: "10.0.0.1"
  port: 8080

clients:
  - name: Alice
    age: 30
  - name: Bob
    age: 25
```

### **Python Code to Handle Complex YAML:**
```python
import yaml

with open("complex.yaml", "r") as file:
    data = yaml.safe_load(file)

print(data)
```

### **Output:**
```python
{
  'defaults': {'timeout': 30, 'retries': 3},
  'server': {'timeout': 30, 'retries': 3, 'host': '10.0.0.1', 'port': 8080},
  'clients': [{'name': 'Alice', 'age': 30}, {'name': 'Bob', 'age': 25}]
}
```

> 🔹 **Note:** YAML references (`&defaults` and `*defaults`) are resolved automatically.

---

## **6. Converting YAML to JSON**
Sometimes, you may need to convert YAML data to JSON format.

### **Python Code for YAML to JSON Conversion:**
```python
import yaml
import json

# Load YAML file
with open("config.yaml", "r") as file:
    yaml_data = yaml.safe_load(file)

# Convert YAML to JSON
json_data = json.dumps(yaml_data, indent=4)

print(json_data)
```

### **Example Output (JSON Format):**
```json
{
    "app": {
        "name": "MyApplication",
        "version": 1.0,
        "debug": true
    },
    "database": {
        "host": "localhost",
        "port": 5432,
        "user": "admin",
        "password": "secret"
    }
}
```

---

## **7. Error Handling in YAML Processing**
When working with YAML, always handle exceptions to prevent crashes.

### **Example Code with Exception Handling:**
```python
import yaml

try:
    with open("config.yaml", "r") as file:
        data = yaml.safe_load(file)
    print("YAML file loaded successfully!")
except FileNotFoundError:
    print("Error: YAML file not found.")
except yaml.YAMLError as e:
    print(f"YAML Parsing Error: {e}")
```

> 🔹 **Tip:** Always handle `yaml.YAMLError` to catch parsing issues.

---

## **8. Using OrderedDict for Ordered YAML Data**
By default, dictionaries in Python don’t maintain order (before Python 3.7). To preserve order, use `yaml.SafeLoader` with `collections.OrderedDict`.

### **Example Code for OrderedDict:**
```python
import yaml
from collections import OrderedDict

def ordered_yaml_load(stream, Loader=yaml.SafeLoader):
    class OrderedLoader(Loader):
        pass

    def construct_mapping(loader, node):
        return OrderedDict(loader.construct_pairs(node))

    OrderedLoader.add_constructor(yaml.resolver.BaseResolver.DEFAULT_MAPPING_TAG, construct_mapping)
    return yaml.load(stream, OrderedLoader)

with open("config.yaml", "r") as file:
    data = ordered_yaml_load(file)

print(data)
```
