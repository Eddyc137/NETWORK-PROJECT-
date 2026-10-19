# NetCore Systems Ltd. Cisco Packet Tracer Network Project

A small enterprise network built in Cisco Packet Tracer for a fictional company with three departments: IT, Sales and HR. It covers IP addressing, DHCP, HTTP and FTP services, inter-VLAN-style routing between separate subnets, and an Access Control List (ACL) that stops Sales from pinging the IT network while still allowing access to the IT server's services.

## Table of Contents

- [Project Overview](#project-overview)
- [Network Topology](#network-topology)
- [Tools and Technologies](#tools-and-technologies)
- [Configuration Steps](#configuration-steps)
- [Results and Findings](#results-and-findings)
- [Author](#author)

## Project Overview

NetCore Systems Ltd. needs a network that lets its IT, Sales and HR departments communicate while applying a basic security restriction. This project was completed as part of the 10Alytics cybersecurity programme.

The goals were to:

- Build a routed network with a separate subnet for each department
- Assign addresses to all PCs automatically using DHCP, and give the server a static address
- Run HTTP and FTP services on an IT server
- Test connectivity between all departments before applying any security controls
- Apply an ACL that blocks ICMP (ping) traffic from Sales to IT, while keeping HTTP and FTP access to the server working

## Network Topology

The network uses one router connected to three switches, one per department. Each router interface is the default gateway for its department.

| Department | Devices | Network | Default Gateway | Router Interface |
| --- | --- | --- | --- | --- |
| IT | 2 PCs + 1 Server | 192.168.10.0/24 | 192.168.10.1 | GigabitEthernet 0/0 |
| Sales | 2 PCs | 192.168.20.0/24 | 192.168.20.1 | GigabitEthernet 0/1 |
| HR | 2 PCs | 192.168.30.0/24 | 192.168.30.1 | GigabitEthernet 0/2 |

**Devices used**

- 1 Cisco 2911 router
- 3 Cisco 2960 switches (one each for IT, Sales and HR)
- 6 PCs
- 1 server (IT-SERVER)

**Addressing**

- IT-SERVER uses a static address of 192.168.10.2 with subnet mask 255.255.255.0 and gateway 192.168.10.1
- All six PCs receive their addresses from the router's DHCP pools

**Cabling**

All links use copper straight-through cables:

- Router G0/0 to the IT switch
- Router G0/1 to the Sales switch
- Router G0/2 to the HR switch
- Each switch to the PCs (and the server, for IT) in its department

## Tools and Technologies

- Cisco Packet Tracer
- Cisco 2911 router (IOS command line)
- Cisco 2960 switches
- DHCP (router-based address pools)
- Static IP addressing and subnetting (/24 networks)
- HTTP and FTP services on the server
- Extended Access Control Lists (ACLs)
- ICMP (ping) for connectivity testing
- Windows (host operating system)

## Configuration Steps

1. Build the topology in Packet Tracer: add one 2911 router, three 2960 switches, six PCs and one server, then connect them with copper straight-through cables as described above.

2. Configure the IT server with a static IP address. Under IT-SERVER, Desktop, IP Configuration, enter:

   ```
   IP Address:      192.168.10.2
   Subnet Mask:     255.255.255.0
   Default Gateway: 192.168.10.1
   ```

3. Configure the router interfaces in the router CLI:

   ```
   enable
   configure terminal

   interface gigabitEthernet 0/0
   ip address 192.168.10.1 255.255.255.0
   no shutdown
   exit

   interface gigabitEthernet 0/1
   ip address 192.168.20.1 255.255.255.0
   no shutdown
   exit

   interface gigabitEthernet 0/2
   ip address 192.168.30.1 255.255.255.0
   no shutdown
   exit
   ```

4. Verify the interfaces with `show ip interface brief`. All three should show Status = up and Protocol = up.

5. Configure a DHCP pool on the router for each department, excluding the first addresses so they stay free for the gateway and server:

   ```
   ip dhcp excluded-address 192.168.10.1 192.168.10.4
   ip dhcp excluded-address 192.168.20.1 192.168.20.4
   ip dhcp excluded-address 192.168.30.1 192.168.30.4

   ip dhcp pool IT
   network 192.168.10.0 255.255.255.0
   default-router 192.168.10.1
   exit

   ip dhcp pool SALES
   network 192.168.20.0 255.255.255.0
   default-router 192.168.20.1
   exit

   ip dhcp pool HR
   network 192.168.30.0 255.255.255.0
   default-router 192.168.30.1
   exit
   ```

6. Set each PC to DHCP under Desktop, IP Configuration, then check the assigned address with `ipconfig` in the Command Prompt.

7. Enable HTTP on IT-SERVER under Services, HTTP, and edit the default web page if desired.

8. Enable FTP on IT-SERVER under Services, FTP, and create an FTP user account.

9. Test connectivity before applying the ACL. Pings between Sales, HR and IT should all succeed at this stage.

10. Test the HTTP server from a Sales PC by browsing to `http://192.168.10.2`.

11. Test FTP from a Sales PC by running `ftp 192.168.10.2` in the Command Prompt and logging in with the FTP account.

12. Create the ACL on the router. It blocks ICMP from the Sales network to the IT network and permits everything else:

    ```
    ip access-list extended Sales-ACL
    deny icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
    permit ip any any
    exit
    ```

13. Apply the ACL inbound on the Sales-facing interface:

    ```
    interface gigabitEthernet 0/1
    ip access-group Sales-ACL in
    exit
    end
    ```

14. Verify the ACL with `show access-lists` and `show ip interface gigabitEthernet 0/1`, then save the configuration with `write memory`.

15. Re-run the tests: Sales to IT ping, Sales to HTTP and FTP, and the HR connectivity tests.

## Results and Findings

- The router interfaces came up correctly, with all three networks reachable through their default gateways.
- DHCP assigned addresses to all six PCs, for example 192.168.10.5 and .6 (IT), 192.168.20.5 and .6 (Sales) and 192.168.30.5 and .6 (HR), each with the correct subnet mask and gateway.
- The IT server's web page loaded from a Sales PC, and an FTP session to 192.168.10.2 connected and listed the server's directory.
- The `show access-lists` output confirmed the extended ACL with its ICMP deny rule and the permit-all rule, applied inbound on GigabitEthernet 0/1.
- Because the ACL denies only ICMP and then permits all other traffic, Sales is blocked from pinging the IT network but still reaches the server over HTTP and FTP. HR and the other department-to-department traffic is unaffected.

| Test | Expected Result |
| --- | --- |
| IT to Sales ping | Success |
| IT to HR ping | Success |
| HR to IT ping | Success |
| HR to Sales ping | Success |
| Sales to HR ping | Success |
| Sales to IT ping | Blocked |
| Sales to IT HTTP | Allowed |
| Sales to IT FTP | Allowed |
| HR to IT HTTP | Allowed |
| HR to IT FTP | Allowed |

**Key takeaway:** an ACL can restrict one specific type of traffic (ICMP) between two networks without cutting off the services (HTTP and FTP) that users still need.

## Author

**Edward Esin**
Aspiring penetration tester and cybersecurity student, United Kingdom

- GitHub: [your-username](https://github.com/your-username)
- Email: your-email@example.com
