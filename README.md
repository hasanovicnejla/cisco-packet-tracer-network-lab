# Cisco Packet Tracer Network Topology Lab


## Overview

This repo documents two tasks from my computer networks lab exam. The exam itself covered more ground, but I've chosen to focus here on the parts that involved hands-on implementation: subnetting an IP block by hand and building a fully functional multi-subnet topology in Cisco Packet Tracer covering everything from router configuration to DHCP, DNS, and HTTP.

The exam covered two main tasks: subnetting calculations (Task 3) and building + verifying the actual topology in Packet Tracer (Task 4). This README explains the what and why behind each part. If you want to see the step-by-step implementation, configuration details, and screenshots, check out `notes.docx`, that's where all the nitty-gritty lives. :)

## Technologies Used

The lab was built using **Cisco Packet Tracer** as the simulation environment. On the networking side, I worked with **IPv4 subnetting** using VLSM to carve up the address space, **DHCP** to automate host IP assignment, **DNS** to resolve the domain name `lab-group-a.ba`, and an **HTTP server** to host a web page accessible across both subnets. Connectivity was verified through **ICMP ping**, and **ARP** plays a role behind the scenes whenever a device needs to resolve a MAC address before sending its first packet.

## Repository Structure

```
/
├── README.md          ← you are here
├── notes.pdf         ← full working notes with screenshots and config 
├── topology.pkt       ← Cisco Packet Tracer project file

```

## Task 3 - IP Subnetting 

The starting point was the IP block `122.87.96.0/21`, which I had to divide into three subnets large enough to support 168, 75, and 13 hosts respectively. The technique used is called **VLSM (Variable Length Subnet Masking)**, instead of giving every subnet the same size, each one gets a mask just big enough for its needs, which keeps address waste to a minimum. Subnets are allocated from largest to smallest so the address space stays contiguous.

Before getting into results, here are the key concepts that come up throughout:

A **subnet mask** is a 32-bit value that tells you which part of an IP address is the network and which part identifies individual hosts. The more network bits you have, the fewer hosts fit in that subnet.

The **Network ID** is the very first address in a subnet's range. It identifies the subnet itself and can't be assigned to any device.

The **broadcast address** is the last address in the range. It's used to send data to every device in the subnet at once, so it also can't be assigned to a host.

The **first and last usable host** addresses are what's left between those two. In this lab, the first usable address in each subnet goes to the router interface, and the last usable address is reserved for a server.

### Results

| Subnet            | Hosts Needed | Mask                    | Network ID    | Broadcast      | First Host     | Last Host      |
|-------------------|--------------|-------------------------|---------------|----------------|----------------|----------------|
| Subnet 1          | 168          | /24 (255.255.255.0)     | 122.87.96.0   | 122.87.96.255  | 122.87.96.1    | 122.87.96.254  |
| Subnet 2          | 75           | /25 (255.255.255.128)   | 122.87.97.0   | 122.87.97.127  | 122.87.97.1    | 122.87.97.126  |
| Subnet 3 (link)   | 13           | /28 (255.255.255.240)   | 122.87.97.128 | 122.87.97.143  | 122.87.97.129  | 122.87.97.142  |


Subnet 3 is a point-to-point link that connects the two routers together. Router 1 gets the first usable address and Router 2 gets the last. All calculation steps are in `notes.docx`.

## Task 4 - Packet Tracer Topology 

This task was about taking those numbers and actually making them work. The two things I needed to verify were: every PC in Subnet 1 can ping every PC in Subnet 2 (and back), and the HTTP server is reachable from all PCs using the domain name `lab-group-a.ba` instead of a raw IP.

The topology was built in a sensible order: routers first, then DHCP, then the application-layer services.

**Router and interface setup** came first. Each router interface got the IP address corresponding to the first usable host of its subnet and was brought up. This is what makes inter-subnet routing possible. Without the routers knowing about both subnets, traffic from Subnet 1 would have no way to reach Subnet 2.

**DHCP** was configured next so that PCs don't need manual IP configuration. The server hands out addresses from the correct pool for each subnet and also tells each client what its default gateway and DNS server are. That last part matters a lot for the next step.

**DNS and HTTP** were the final piece. The HTTP service was enabled on the server, and a DNS record was created linking `lab-group-a.ba` to the server's IP. Because DHCP already told every client where the DNS server is, any PC in either subnet can type in the domain name and get the right IP back automatically.

> ![Full Topology](screenshots/topology-overview.png)

### Subtask 1 – Ping Between Subnets

To confirm connectivity, I used PC1 in Subnet 1 to ping PC5 in Subnet 2.

Worth noting: the first one or two ping packets will time out, and that's completely normal. Before sending any ICMP echo request, the device has no idea what MAC address belongs to the next hop (the router). So it first sends out an **ARP broadcast** essentially asking the network "who has this IP address?" Once the router responds and the MAC address is cached, the ICMP packets go through without any issues. Seeing this timeout-then-success pattern actually confirms that ARP, routing, and ICMP are all doing their jobs correctly.

> ![Ping from PC1 to PC5](screenshots/ping-pc1-pc5.png)

### Subtask 2 - Web Server Access by Domain Name

The goal here was to prove that the HTTP server works from both subnets using `lab-group-a.ba` as the address, not the IP directly. For this to work, three things have to be true at the same time: DNS has a record pointing the domain to the server's IP, DHCP has told every client where the DNS server is, and routing is functional so that DNS queries and HTTP responses can cross subnet boundaries.

I verified this by opening the web browser on PCs in both Subnet 1 and Subnet 2 and navigating to `lab-group-a.ba`. A successful page load means the entire chain worked - name resolution, routing, and the HTTP service itself.

> ![Browser Subnet 1](screenshots/browser-subnet1.png)

> ![Browser Subnet 2](screenshots/browser-subnet2.png)

## Full Documentation

Everything that isn't in this README like calculation steps, configuration screenshots, and implementation details are in **`notes.docx`**. That file is the main reference for how things were actually set up.
