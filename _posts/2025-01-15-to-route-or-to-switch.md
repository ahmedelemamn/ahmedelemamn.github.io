---
layout: post
title: "To Route or to Switch?"
date: 2025-01-15 08:25:00 +0000
categories: [Networking Basics]
tags: [Networking, Routing, Switching]
---

# **To Route or to Switch?**

At the core of networking lie two fundamental operations: **switching** and **routing**. Both are responsible for directing data across networks, yet they operate at different layers of the OSI model and have distinct purposes. This post dives into the deep meaning of these concepts, starting with switching, exploring VLANs, and then moving into routing.

## **Switching: The Foundation of Local Communication**

Switching operates at the **Data Link Layer (Layer 2)** of the OSI model. A switch's primary job is to forward frames (data packets encapsulated with MAC addresses) within a local area network (LAN). It does so by learning which devices (MAC addresses) are connected to its various ports and then creating a **MAC address table** to intelligently forward traffic.

### **The Role of VLANs in Switching**

In a typical LAN, devices might all be part of the same broadcast domain. This means that any broadcast sent by one device is received by all others—potentially overwhelming the network. **Virtual Local Area Networks (VLANs)** solve this problem by logically segmenting the network. 
Each VLAN acts as an independent broadcast domain. Devices in VLAN 10, for example, cannot communicate directly with devices in VLAN 20 unless a router or Layer 3 switch is used to route between them. Using VLANs is like assigning separate lanes on a highway for different types of vehicles. Even though all vehicles travel on the same highway (the physical switch), they remain separated logically to improve safety and performance.

**Why VLANs Matter**:
- **Improved Security**:  by isolating certain users or devices from others.
- **Increase efficiency**: by reducing broadcast traffic.
- **Enhance scalability**: by allowing administrators to create separate virtual broadcast domains without adding more hardware.

Switches perform their operations based on **Layer 2 headers (MAC addresses)**, and VLANs enhance the flexibility and scalability of this local communication.


## **Routing: Finding the Best Path**

Routing takes communication beyond the boundaries of a single network. Operating at the **Network Layer (Layer 3)**, a router determines the best path for forwarding packets to their destination. This can involve traversing multiple networks, subnets, and hops.

Think of routing as receiving directions to reach a specific destination. The parameters affecting routing are analogous to factors influencing how you plan your journey:
- **Administrative Distance (Trustworthiness)**: Trust levels for different types of "advice" (e.g., static routes, OSPF, BGP).
- **Metrics (Cost of Path)**: Metrics like bandwidth, hop count, and delay help determine the most efficient path.
- **Policies**: Certain paths may be preferred or avoided based on policies (e.g., avoid toll roads).


### **Switching vs. Routing: The Key Differences**

| **Feature**         | **Switching**               | **Routing**                   |
|----------------------|-----------------------------|--------------------------------|
| **Layer**            | Data Link Layer (Layer 2)   | Network Layer (Layer 3)       |
| **Primary Unit**     | Frame (MAC address-based)   | Packet (IP address-based)     |
| **Domain**           | Local broadcast domain      | Inter-network communication   |
| **Primary Device**   | Switch                      | Router                        |

---

### **Packet Processing in Routing Devices**

What happens when a router receives a packet? How does it decide its path?

To answer that, we need to discuss a few core concepts to understand the packet flow within a routing device. Notice I didn’t say "router," but instead intentionally mentioned "routing device" because routers aren’t the only devices that perform routing. Layer 3 switches, firewalls, and even your Linux server can perform advanced routing if configured to do so. The naming of concepts and definitions may vary across different platforms and vendors, but the core idea remains the same.

### Forwarding Information Base (FIB), Cisco Express Forwarding (CEF), and Routing Table (RIB)

The **Forwarding Information Base (FIB)**, **Cisco Express Forwarding (CEF)**, and the **Routing Table (RIB)** are all crucial components of packet forwarding in a router. Here’s an explanation of each and how they differ, followed by an overview of how packets flow between them.

---

#### 1. Routing Table (RIB)
- **What it is**: The Routing Information Base (RIB) is a table that stores the best routes learned from routing protocols (e.g., OSPF, EIGRP, BGP), directly connected networks, and static routes.
- **Purpose**: It determines the best path for a destination and installs it into the FIB.
- **Key Attributes**:
  - Contains all routes learned, including backup paths (if supported by the protocol).
  - Created and managed by routing protocols and is not directly used for packet forwarding.

---

#### 2. Forwarding Information Base (FIB)
- **What it is**: The FIB is a hardware-optimized table derived from the routing table. It holds the best paths for forwarding packets.
- **Purpose**: Provides the router with fast lookup capabilities to forward packets.
- **Key Attributes**:
  - Contains only the best path to each destination.
  - Is optimized for speed and uses a structure suitable for hardware lookup (like a trie or hash table).

---

#### 3. Cisco Express Forwarding (CEF)
- **What it is**: CEF is a high-performance, hardware-accelerated packet forwarding mechanism used by Cisco devices.
- **Purpose**: Enables fast, scalable forwarding of packets using precomputed data structures (the FIB and adjacency table).
- **Key Attributes**:
  - The FIB and adjacency table are the two main data structures used in CEF:
    - **FIB**: Contains the best path information (IP prefixes).
    - **Adjacency Table**: Contains Layer 2 (MAC address) and next-hop information for resolved IP addresses.
  - Operates in the data plane, bypassing the CPU for most traffic.

---

#### Differences Between RIB, FIB, and CEF

| **Feature**         | **RIB (Routing Table)**  | **FIB (Forwarding Table)**  | **CEF**                     |
|----------------------|--------------------------|-----------------------------|-----------------------------|
| **Purpose**          | Stores routes           | Forwards packets            | Mechanism for fast forwarding |
| **Scope**            | Control plane           | Data plane                  | Data plane                  |
| **Content**          | All available routes    | Best routes for forwarding  | Uses FIB + adjacency table  |
| **Computation**      | Includes all routes     | Populated from RIB          | Uses precomputed FIB        |
| **Role in Forwarding** | Indirect (for FIB updates) | Directly used for forwarding | Underpins FIB and adjacency |

---

#### How Packets Flow Between RIB, FIB, and CEF

1. **RIB Creation**:
   - Routing protocols populate the RIB with routes.
   - Best routes are selected based on protocol preferences (Administrative Distance and Metric).

2. **FIB Population**:
   - The FIB is updated with the best routes from the RIB. It contains only the information needed for packet forwarding (destination prefixes and next-hops).

3. **Adjacency Table (CEF)**:
   - The CEF adjacency table resolves the next-hop IP addresses to Layer 2 information (MAC addresses for Ethernet).
   - It contains precomputed, ARP-resolved information for fast packet forwarding.

4. **Packet Flow**:
   - **Step 1**: A packet arrives at the router interface.
   - **Step 2**: The router examines the packet's destination IP address and looks it up in the FIB (not the RIB).
   - **Step 3**: The FIB identifies the best next-hop for the destination.
   - **Step 4**: The adjacency table provides the Layer 2 information to forward the packet to the next hop.

---

#### Visual Flow

```plaintext
Routing Protocols (e.g., OSPF, BGP) 
            ↓
      Routing Table (RIB)
            ↓
       FIB (Best Routes)
            ↓
   CEF (FIB + Adjacency Table)
            ↓
       Packet Forwarding
```

This flow ensures that packets are forwarded quickly and efficiently, with minimal involvement from the CPU after the initial route calculations.

#### Note

- CEF is a Cisco technology and is not available in other vendors' devices. Most likely, the two main components you'll encounter are the RIB and FIB.

