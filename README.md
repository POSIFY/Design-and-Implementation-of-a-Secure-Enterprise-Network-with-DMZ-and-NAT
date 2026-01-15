# Design-and-Implementation-of-a-Secure-Enterprise-Network-with-DMZ-and-NAT

This project demonstrates the design and implementation of a secure enterprise network using a DMZ and NAT. It showcases traffic segregation between internal and external networks, controlled access to public-facing services, and firewall-enforced security policies to protect sensitive resources.

### Project Objective
The goal of this project is to design and implement a secure enterprise network topology that incorporates a Demilitarized Zone (DMZ) and Network Address Translation (NAT). This topology demonstrates how to:
* Segregate internal and external traffic to protect sensitive internal resources.
* Provide controlled access to publicly accessible services in the DMZ without exposing the internal network.
* Implement NAT to allow internal hosts to access the internet while hiding their private IP addresses.
* Apply firewall policies to enforce security between the LAN, DMZ, and internet.
* Simulate real-world traffic flows for testing and validation of security configurations.

This project provides a hands-on demonstration of perimeter security design, traffic isolation, and access control, reflecting industry-standard practices for enterprise networks.

### Devices in Topology:

<img width="1087" height="592" alt="image" src="https://github.com/user-attachments/assets/b52c0f1f-3d14-43c6-b1c7-70ec212e4fc4" />

* Cisco ASA 5505 – Firewall / Security Gateway
* LAN Switch – Connects internal hosts
* DMZ Switch – Connects servers in the DMZ
* DMZ Server – Public-facing server for external access
* Cisco ISR 4231 – Edge router / Gateway to Internet
* Internet Switch – Simulates external network connectivity

### Step 1: Preparing the ASA Firewall
The first step in configuring the ASA firewall is to review the existing configuration. Using the ASA CLI, the command:

```show running-config```

allows us to view the current running configuration. This is essential for troubleshooting and understanding any pre-existing settings that may affect the new setup.
Once the existing configuration is reviewed, it is cleared using the command:

```write erase```

This resets the firewall to a clean state, ensuring that no previous settings interfere with the new network configuration. Starting from a clean configuration helps prevent conflicts and ensures a smooth implementation of DMZ and NAT rules.
<img width="946" height="825" alt="image" src="https://github.com/user-attachments/assets/5a982dbd-3795-4b73-aff7-ca73186ba9e6" />

### Step 2.1: Configuring the ASA VLAN 1 Interface (Inside Network)

The next step involves configuring the ASA firewall interfaces, starting with the VLAN 1 interface, which will represent the internal LAN. Using the CLI:
```
conf t
interface VLAN 1
nameif inside       ! optional, names the interface
security-level 100  ! sets the trust level of the interface
ip address 192.168.1.1 255.255.255.0
no shutdown
```
<img width="951" height="875" alt="image" src="https://github.com/user-attachments/assets/72a9a81c-09f9-42b3-a4f4-f1a8435d1948" />

### Step 2.2: Configuring End Devices for LAN Connectivity
To ensure proper communication between VLAN 1 on the ASA firewall and the internal end devices, all devices on the inside network must reside in the same subnet.
In this case, the inside network is configured as 192.168.1.0/24. Therefore, the connected PC was assigned the following IP settings:
* IP Address: 192.168.1.10
* Subnet Mask: 255.255.255.0
* Default Gateway: 192.168.1.1 (ASA VLAN 1 interface)
<img width="689" height="693" alt="image" src="https://github.com/user-attachments/assets/f24de5e4-8160-4120-aa95-41ddf8da932d" />

This configuration ensures that the PC can communicate with the firewall and other devices within the same LAN subnet, laying the foundation for NAT, firewall rules, and further DMZ integration.

### Step 2.3: Configuring ASA VLAN 2 Interface (Outside Network)
With VLAN 1 (inside) successfully configured, the next step is to configure VLAN 2, which represents the outside (internet-facing) interface of the ASA firewall.
Although the configuration process is similar to VLAN 1, the security level and IP addressing differ to reflect the untrusted nature of the external network.
The following configurations were applied:
* The interface is assigned a security level of 0, indicating the lowest trust level.
* A public-facing IP address is configured to enable communication with the upstream router.

Configuration commands:
```
conf t
interface VLAN 2
nameif outside
security-level 0
ip address 209.165.200.2 255.255.255.248
no shutdown
```
<img width="921" height="824" alt="image" src="https://github.com/user-attachments/assets/c6a6f774-f6a8-4608-a4cb-2142f5edf3f1" />

This configuration allows the ASA firewall to forward traffic between the internal network and the external network while enforcing security policies. It also serves as the foundation for implementing NAT and controlling internet-bound and inbound traffic.

### Step 2.4: Configuring the DMZ Interface on the ASA Firewall

The next step is to configure a DMZ interface on the ASA firewall to host publicly accessible services while maintaining isolation from the internal LAN.
A new interface, VLAN 3, was created and intended to represent the DMZ. Initially, an error was encountered when attempting to name the interface. This occurs because, on the ASA 5505, a new VLAN interface must be properly associated within the firewall’s internal switching architecture before it can be fully defined.
To resolve this, VLAN 3 was logically associated with the internal switching domain while still maintaining traffic control boundaries. This was achieved using the no forward command, which prevents direct forwarding between the internal LAN and the DMZ without explicit security rules.
Once this was applied, the VLAN interface could be named and configured correctly.

**By design:**
* Traffic from the internal LAN (VLAN 1) to the DMZ (VLAN 3) is permitted.
* Traffic from the DMZ to the internal LAN is denied by default unless explicitly allowed.

Because the DMZ is partially trusted, its security level was adjusted accordingly.

Configuration commands:
```
conf t
interface VLAN 3
nameif dmz
security-level 50
ip address 192.168.2.1 255.255.255.0
no shutdown
```
<img width="1079" height="790" alt="image" src="https://github.com/user-attachments/assets/2a1a063c-f0b1-4fc9-bd2b-fb6d4086a00a" />

This configuration establishes a proper three-zone security model:
1. Inside (VLAN 1) – security level 100
2. DMZ (VLAN 3) – security level 50
3. Outside (VLAN 2) – security level 0
At this stage, the ASA firewall is fully aware of the LAN, DMZ, and upstream router. However, it does not yet know how to reach external networks beyond the router. This will be addressed in the next steps through routing and NAT configuration.

### Step 2.5: Enabling Routes to the Internet

After configuring all ASA firewall interfaces, the next step is to ensure the firewall can forward traffic to external networks. This requires defining a default route that points to the upstream router.
The following commands were used:
```
enable
conf t
route outside 0.0.0.0 0.0.0.0 209.165.200.1
```
<img width="693" height="249" alt="image" src="https://github.com/user-attachments/assets/3a89b2b2-19d8-4ae8-a43f-cab64d37ca9b" />

**Command Breakdown:**
**route outside** – Specifies that this route applies to the outside interface of the ASA firewall. Traffic destined for networks not known to the firewall will use this interface to reach external networks.
**0.0.0.0 (first occurrence)** – Represents the destination network. Using 0.0.0.0 indicates this is a default route, meaning it applies to all traffic for which no specific route exists.
**0.0.0.0 (second occurrence)** – Represents the subnet mask for the destination. A subnet mask of 0.0.0.0 means all IP addresses match, reinforcing that this is a catch-all default route.
**209.165.200.1** – The next-hop IP address, which is the upstream router’s IP on the external network. Traffic will be sent to this device to reach the internet or any external network.

This configuration ensures that any traffic from the inside LAN or DMZ that is not destined for a known network will be forwarded to the router, allowing the ASA firewall to handle NAT and enforce security policies while providing internet access.

### Step 2.6: Assigning Physical Interfaces to VLANs
After configuring the VLAN interfaces on the ASA firewall, the next step is to map the physical Ethernet ports to their respective VLANs. This ensures that traffic from connected devices flows through the correct security zones (Inside, Outside, and DMZ).
The following CLI commands were used:
```
! Assign physical port Ethernet0/1 to the Inside VLAN
interface ethernet0/1
switchport mode access
switchport access vlan 1

! Assign physical port Ethernet0/0 to the Outside VLAN
interface ethernet0/0
switchport mode access
switchport access vlan 2

! Assign physical port Ethernet0/2 to the DMZ VLAN
interface ethernet0/2
switchport mode access
switchport access vlan 3
```
<img width="778" height="280" alt="image" src="https://github.com/user-attachments/assets/7e32ef32-b080-4143-a77c-6ab00c2e3c06" />

**Explanation of commands:**
**interface ethernet0/X** – Selects the physical port on the ASA firewall to configure.
**switchport mode access** – Configures the port as an access port, which carries traffic for a single VLAN.
**switchport access vlan X** – Assigns the port to the corresponding VLAN (1 = Inside, 2 = Outside, 3 = DMZ).
By mapping physical ports to VLANs, the ASA firewall can correctly segregate traffic between the internal LAN, DMZ, and external network, enforcing security policies and enabling proper routing and NAT.


### Step 3: Configuring the Router Interfaces and Connected End Devices
The next step is to configure the internet router to provide connectivity between the ASA firewall and external networks. This ensures proper routing and NAT functionality for the LAN and DMZ.
Upon accessing the router CLI, I was prompted to use default configurations. I selected “no” to disable defaults, ensuring that only explicitly configured settings are active and preventing potential conflicts with the firewall setup.

The router configuration proceeded as follows:
```
enable
conf t

! Configure the interface connecting to the ASA firewall
interface g0/0/0
ip address 209.165.200.1 255.255.255.248
no shutdown

! Configure the interface connecting to the external network
interface g0/0/1
ip address 8.8.8.1 255.255.255.0
no shutdown
```
<img width="809" height="791" alt="image" src="https://github.com/user-attachments/assets/096129dd-3173-4c25-8bcf-976fc4e601c4" />

**Explanation of commands:**
**enable** – Enters privileged mode to allow configuration.
**conf t** – Enters global configuration mode.
**interface g0/0/0** – Selects the router interface connected to the ASA outside interface.
**ip address 209.165.200.1 255.255.255.248** – Assigns the IP address for communication with the ASA firewall.
**no shutdown** – Activates the interface.
**interface g0/0/1** – Selects the router interface connected to the external network.
**ip address 8.8.8.1 255.255.255.0** – Assigns the external IP address for testing or simulating an ISP network.
**no shutdown** – Brings the external interface up.

This configuration ensures the router can forward traffic between the ASA firewall and the external network, allowing internal and DMZ devices to reach the internet while maintaining controlled access via the firewall.

### Step 4: Configuring IP Addresses for Devices Behind the Router
The next step is to configure the end devices (PC and server) connected to the switch on the router’s external interface. Proper IP assignment ensures the devices can communicate with the router and, through the ASA firewall, reach internal or DMZ networks if needed.

| Device | IP Address | Subnet Mask     | Default Gateway |
|--------|-----------|----------------|----------------|
| PC     | 8.8.8.2   | 255.255.255.0  | 8.8.8.1        |
| Server | 8.8.8.3   | 255.255.255.0  | 8.8.8.1        |

<img width="685" height="631" alt="image" src="https://github.com/user-attachments/assets/190424a2-99bc-459e-87e3-9209efb5a9c3" />

**Explanation:**
IP addresses 8.8.8.2 and 8.8.8.3 are in the same subnet as the router interface (8.8.8.1/24), allowing direct communication.
Default gateway 8.8.8.1 ensures that any traffic destined for networks outside this subnet (LAN, DMZ, or internet) is forwarded to the router.
Subnet mask 255.255.255.0 defines the network range and ensures proper communication within the local network segment.

This step completes the IP configuration for all devices in the lab, enabling connectivity through the router and ASA firewall.


