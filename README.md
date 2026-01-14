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

This configuration ensures that the PC can communicate with the firewall and other devices within the same LAN subnet, laying the foundation for NAT, firewall rules, and further DMZ integration.
