# Enterprise-Network-Build

# Overview #
In this lab, I built a greenfield enterprise network using the network emulation software, EVE-NG. I created this lab to demonstrate several networking concepts that could be used in the deployment of a company's enterprise network.

# Objective #
Business is booming at FixIT!, a local technology firm. The company's owners have decided to expand their operations by opening two new company locations. The Chicago, IL and New York City, NY offices will house the Sales and Marketing departments. The company will also rent space in a datacenter colocation for the purposes of interconnecting the Chicago and New York offices as well as providing a connection to the Internet. The datacenter will host a web server, database server, file server, and a log server to support day-to-day operations. Management's top priority is providing their customers the best support possible, as such, the network design should incorproate resiliency and security best practices as appropriate.

# Resources used #
The following resources were used to build this lab.

1. My Dell T620 server to support the virtualization.
2. VMware Workstation 17 Player
3. A copy of the EVE-NG community edition OVF file (can be found [here](https://www.eve-ng.net/index.php/download/#DL-COMM)).
4. Device image files below (run within EVE-NG). (My images came from the Cisco software site using a [CML subscription](https://www.cisco.com/c/en/us/products/cloud-systems-management/modeling-labs/index.html)).

# Device Images #

| Device Type | Platform | Image |
| ----------- | ----- | ----- |
| Router      | Cisco IOS | vios-adventerprisek9-m.vmdk.SPA.157-3.M3      |
| Switch (L2) |  Cisco IOS| vios_l2-adventerprisek9-m.SSA.high_iron_20180619.qcow2     |
| Switch (L3) | Cisco Nexus| nexus9500v.9.3.7.qcow2      |
| Firewall    | Cisco ASA | asav992.qcow2     |
| Workstation | Linux Ubuntu Desktop 22.04.1 | ubuntu-22.04.1-desktop-amd64.iso     |
| Server | Linux Ubuntu Server 22.04.3 | ubuntu-22.04.3-live-server-amd64.iso      |

# Resource Consumption #
My EVE-NG VM had the following resource allocations:

![vm-settings](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/0233db44-94cd-41a5-837c-fb006c8cf9bc)

With every node turned on and configured, here was the resource utilization on the VM.

![top](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/d733642d-c84d-4c3d-9b53-103dbad27076)

# Topology #

![final topology](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/d83b1e69-933a-4de9-a623-33692f4e3bbc)

# Design #

## Workstations ##

Company workstations will be deployed to the office locations for use by the employees. A Sales employee will use Workstation-01 while a Marketing employee will use Workstation-02.

## Webserver ##

Located in the datacenter, this webserver hosts the FixIT! external facing website. Customers can access the website from the Internet to browse the company's product offerings and order a variety of computer parts. The server is running Nginx.

## Database Server ##

Located in the datacenter, this database stores inventory records, sales records, and customer records needed to support the Sales and Marketing departments. The server is running PostgreSQL.

## Marketing Server ##

Located in the datacenter, this file server stores product and service advertisements, images, and other files needed to support the marketing department. The server is running Samba.

## Log Server ##

Located in the datacenter, this log server is responsible for aggregating logs and event data from the firewalls, routers and switches. The server is running Splunk Enterprise.

## Routers ##

Two network routers will be deployed to each location which will be responsible for routing network traffic in and out of the location. Each router will contain two uplinks. The primary uplink will connect directly to the remote router while the secondary uplink will connect to the peer router. If the primary uplink fails, the router will route traffic through the peer router, utilizing its primary uplink. Routers will exchange network reachability information using the BGP protocol. Each router will connect to each downstream switch. The office routers will use the "Router-on-a-stick" technique using 802.1Q VLAN encapsulation and will house the gateways for the downstream workstations. The network assignments are in the table below.

| Network |  HSRP IP | Router-01 IP | Router-02 IP | VLAN-ID | Assigned to |
| ----------- | ----- | ----- | ----- | ----- | ----- |
| 10.20.0.0/23 | 10.20.0.1 | 10.20.0.2 | 10.20.0.3 | 100 | CHI-WORKSTATION-01 |
| 10.20.2.0/23 | 10.20.2.1 | 10.20.2.2 | 10.20.2.3 | 200 | CHI-WORKSTATION-02 |
| 10.30.0.0/23 | 10.30.0.1 | 10.30.0.2 | 10.30.0.3 | 100 | NYC-WORKSTATION-01 |
| 10.30.2.0/23 | 10.30.2.1 | 10.30.2.2 | 10.30.2.3 | 200 | NYC-WORKSTATION-02 |

## Switches ##

One network switch will be deployed to each office location and will be responsible for switching network traffic between the workstations/security appliance and the routers. Each switch will contain two uplinks to the upstream routers. These switches will segment Layer-2 traffic using two VLANs. Each workstation will reside in a separate VLAN. The VLAN assignments are in the table below. The same IDs are used in both sites for consistency only; the layer-2 domains will not be extended between sites.

| VLAN-ID | VLAN Name | Assigned to |
| ----------- | ----- | ----- |
| 100 | SALES-VLAN | CHI-WORKSTATION-01, NYC-WORKSTATION-01 |
| 200 | MARKETING-VLAN | CHI-WORKSTATION-02, NYC-WORKSTATION-02 |

Two Nexus 9500 series network switches will be deployed to the datacenter facility. These layer-3 switches will be responsible for the switching and routing of network traffic between the servers and the upstream routers. Both switches will be peers in a Virtual Port Channel (VPC) domain and maintain an iBGP peering with eachother and eBGP peerings with the data center routers. Each datacenter server will reside in a separate VLAN. These switches will contain the following network assignments.

| Network | VLAN-ID | VLAN Name | HSRP IP | Switch-01 IP | Switch-02 IP | Assigned to |
| ----------- | ----- | ----- | ----- | ----- | ----- | ----- |
| 50.50.50.5/27 | 300 | WEBSERVER-VLAN | 50.50.50.1 | 50.50.50.2 | 50.50.50.3 | DC-WEBSERVER-01 |
| 10.10.2.5/23 | 400 | DATABASE-VLAN | 10.10.2.1 | 10.10.2.2 | 10.10.2.3 | DC-DATABASE-01 |
| 10.10.4.5/23 | 500 | MARKETING-VLAN | 10.10.4.1 | 10.10.4.2 | 10.10.4.3 | DC-MARKETING-01 |
| 10.10.6.5/23 | 600 | LOGGING-VLAN | 10.10.6.1 | 10.10.6.2 | 10.10.6.3 | DC-LOGSERVER-01 |

## Firewalls ##

Two Cisco ASA firewalls will be deployed to the datacenter facility. These devices will be responsible for inspecting incoming and outgoing network traffic to the organization. Each firewall maintains a BGP peering to one of the two ISP and data center routers.

## ISP Routers ##

There are two ISP routers in the topology. Each router will maintain a BGP peering to one of the two firewalls through which it advertises a default (0.0.0.0/0) route that is leaked into the data center and office locations. If a route can't be found on an internal device, the traffic is sent to the ISP router via the firewall.

# Practical #

Both Sales workstations are able to connect to the Sales Database on DC-DATABASE-01. Each workstation is able to execute a SQL query against the DB that returns inventory data.

**CHI-WORKSTATION-01**

![chi-sales-db](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/36ee5390-34d0-44ea-aedf-85ecdcfe13dd)

**NYC-WORKSTATION-01**

![nyc-sales-db](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/a388e598-5fb7-4608-8e9d-7d9bea4210b7)

Both Marketing workstations are able to connect to the Marketing file share on DC-MARKETING-01. Each workstation is able to browse to the remote directory.

**CHI-WORKSTATION-02**

![chi-workstation-02](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/863c6d48-70e9-4e02-b6b7-38f4b2eb82b7)

**NYC-WORKSTATION-02**

![nyc-workstation-02](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/84a0216e-b031-49eb-af06-7bb183c414db)

**DC-FIREWALL-01**

There is an ACL on the firewall called INTERNET-INBOUND that is permitting HTTPS/443 connections to the webserver. All other connection types will be denied by the firewall. The firewall is sending its logs to the log server.

![firewall-acl-inbound](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/5f52b0e4-a209-4da6-8a4d-af32582693ca)

**EXTERNAL-WORKSTATION**

The External workstation, located somewhere on the public Internet, is able to connect to the FixIT! website on DC-WEBSERVER-01, using HTTPS (443).

![fixit-validation](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/dcfb822a-6c57-4363-821b-8c2ee1836230)

The telnet (21), ssh (22), and http (80) connections are unsuccessful as they are blocked by the firewall.

![fixitfails](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/16addd30-2581-481c-9224-0a63841c8416)

**DC-LOGSERVER-01**

This log server is running Splunk Enterprise and is listening for logs at UDP port 514 (syslog).

**Allow Logs**

The server is displaying logs from the firewall which indicate that the previous connection attempts between the external workstation and the webserver at port 443 have been permitted.

![splunk-443](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/4af7406e-19a6-477b-8d3e-ecc60b07280c)

**Deny Logs**

Additional logs indicate that connection attempts using telnet (port 23), ssh (port 22), and http (port 80) were denied by the firewall.

![23](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/4a5ead46-682e-4d14-b8d3-f8c8f0190dc9)

![22](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/90774366-4b8a-4738-a5e9-426cc55493e9)

![80](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/feb7a2e8-cc5e-4c80-8752-d7a03aad7008)

# Resiliency Testing #

In networking, the phrase 'two is one' is used to describe the redundant deployment of network infrastructure devices. The current topology employs fault-tolerant configuration at the network device level to ensure
that the failure of any one network device does not cause downstream impact to application traffic. This section will simulate the failure of half of the network devices in the path between *NYC-WORKSTATION-02* and *DC-MARKETING-01*, in order to test the resiliency of the current deployment.

# Baseline Connectivity #

*NYC-WORKSTATION-02*

A traceroute displays the layer-3 hops between *NYC-WORKSTATION-02* and *DC-MARKETING-01* on a viable path. I added the interface IPs and hostnames to the local hosts file on the workstation to clearly discern each hop. Take note of the current path. (Disregard the high RTT as it is a consequence of the virtual lab).

![nyc-workstation-02-traceroute](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/2ef99361-d0b0-4d7d-9c18-e40cfe89e441)

*NYC-ROUTER-01*

The output below indicates that the workstation's gateway, 10.30.2.1, is currently active on this router (HSRP Primary) while the peer router is in a standby role (HSRP Standby). The traceroute above supports this fact.

![nyc-router-01-hsrp-act](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/38e47290-2ac4-4743-a161-70558b32481d)

![nyc-router-02-hsrp-sb](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/56c3bb5e-e792-411d-93f3-95fb75db07d3)

*DC-SWITCH-01*

The output below indicates that the server's gateway, 10.10.4.1, is currently active on this switch (HSRP Primary) while the peer switch is in a standby role (HSRP Standby). The traceroute above supports this fact.

![dc-switch-01-hsrp-active](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/d8a2fec2-3bed-4c6b-9014-412575caadc4)

![dc-switch-02-hsrp-sb](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/f32e98aa-0bdf-4fb1-80c8-f7aea0771cd1)

A continuous ping to the destination server is initiated from the workstation.

![nyc-workstation-02-ping](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/2b63fbc5-a377-41e8-aa56-6bdaff24ba80)

# Failure Scenario #

Multiple network devices will now be shutdown to simulate a failure. I'm going to shutdown each device that was displayed in the traceroute, forcing traffic over to the peer device.

![failed-topology](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/99d30377-177c-42f0-8afd-192804d5f101)

As the devices are shutdown, **the ping test begins to stall** because the ICMP requests and responses are failing due to a lack of network connectivity while the network reconverges.

![stalled-icmp](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/c35c1c5b-751f-44ee-a010-93836c675e52)

*NYC-ROUTER-02* assumes the HSRP active role for the workstation's gateway, 10.30.2.1 while *DC-SWITCH-02* assumes the HSRP active role for the server's gateway, 10.10.4.1.

![nyc-router-02-hsrp-active](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/cd1873cb-1f51-4ed7-86ab-b645c34e4d76)

![dc-switch-02-hsrp-active](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/f9db8a33-c48f-4f88-b282-5f94b2c333d7)

**The previously stalled ping test starts seeing success** after 13 seconds without the need for manual intervention.

![nyc-workstation-02-ping-continuous](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/5c078db1-bbba-483c-b384-b60e5d1bcf96)

A traceroute from the workstation indicates that traffic has successfully routed around the failed '01' devices automatically.

![nyc-workstation-02-post-reconvergence](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/6586faf2-4f03-474a-8d63-0609802581f5)

# Restoring the environment #

Each device is turned back on to restore the environment to its normal and resilient state.

![all-good](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/a13e8958-b13f-4de4-a298-30e23ce5a824)

Another disruption to traffic occurs while the network re-converges.

*NYC-ROUTER-01* and *DC-SWITCH-01* regain their HSRP active roles as *preemption* is configured. *NYC-ROUTER-02* and *DC-SWITCH-02* assume HSRP standby.

![nyc-router-01-hsrp-act-normalized](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/11e2b638-26a7-4d03-adcb-f92a8b660568)

![dc-switch-01-hsrp-active-normalized](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/84ecf9b0-4c07-4f10-a620-14df08cd9eed)

![nyc-router-02-hsrp-sb-normalized](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/947fe053-f9d7-41e0-8ca9-8e72a0906e40)

![dc-switch-02-hsrp-sb-normalized](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/7c057322-784f-4e3b-9a3b-131c88593269)

A ping test to the server is successful.

![nyc-workstation-02-ping-2](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/bf4ebb00-5f03-404d-ac34-e8317b20a034)

A traceroute indicates that traffic is now being routed to some of the '01' devices again.

![nyc-workstation-02-traceroute-3](https://github.com/BrandonVolker/Enterprise-Network-Build/assets/150546583/cffe259d-0bc8-4c31-b7fe-39193edc8dd4)

## Conclusion ##

This lab began by demonstrating network connectivity between the workstations and the servers in the data center. Nginx (Web server), PostgreSQL (database), Samba (file server), and Splunk (log server) are the applications that were deployed--many of which are commonly found in an enterprise environment. Power failures were then introduced which tested the resiliency of the network deployment. It was observed that connectivity between the workstation and the server restablished after a period of packet loss caused by network reconvergence. The network devices were able to automatically recover from the failures due to a few redundancy technologies.

**Redundant Hardware** in the form of multiple routers, switches and firewalls allows us to configure each device using the redundant technologies described below.

**HSRP** is configured on the routers to provide a fault-tolerant deployment of gateways for the workstations. Both routers participate in the HSRP process and only one is active for that gateway IP at a time. When an issue occurs with that router, in our case a power failure, the standby router becomes active and forwards traffic.

**Multiple uplinks** on the switches and routers provide alternate paths for traffic when a preferred link fails. Combined with **multiple BGP peerings**, the uplinks provide multiple interfaces in which to learn network routing information when a path goes down. Though not tested in this lab, if the uplink on an office routers goes down, the router will still learn prefixes through the peer router, thanks to iBGP. Additionally, each data center server uses interface bonding which specifies an active and backup interface. When the active interface fails, the backup will automatically sense the failure and begin forwarding traffic.

Our network recovered within 13 seconds which is fairly quick in this non-critical environment. For mission-critical environments however, a multi-second outage can be very costly. It's important to mention that there are technologies that can be deployed that will minimize the network downtime even further. BFD (Bi-Directional Forwarding), for example, can be used on our BGP peers to detect link failures within milliseconds. BFD allows BGP to failover before waiting for the multi-second hold timer after which it would normally fail over.

This marks the end of this lab.

## Helpful Resources ##

Here is some helpful reference material in the event you'd like to re-create this lab.

[How to Configure Router on a Stick](https://networklessons.com/cisco/ccna-routing-switching-icnd1-100-105/how-to-configure-router-on-a-stick)

[HSRP Overview and Basic Configuration (IOS)](https://community.cisco.com/t5/networking-knowledge-base/hsrp-overview-and-basic-configuration/ta-p/3131590)

[Cisco Nexus: HSRP Configuration](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/93x/unicast/configuration/guide/b-cisco-nexus-9000-series-nx-os-unicast-routing-configuration-guide-93x/b-cisco-nexus-9000-series-nx-os-unicast-routing-configuration-guide-93x_chapter_010010.html)

[Cisco Nexus: Configure Basic BGP](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/9-x/unicast/configuration/guide/l3_cli_nxos/l3_bgp.html)

[IP Routing: BGP Configuration Guide (IOS)](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_bgp/configuration/xe-16/irg-xe-16-book/configuring-a-basic-bgp-network.html)

[Cisco ASA Access List](https://networklessons.com/cisco/asa-firewall/cisco-asa-access-list)

[How to Install and Configure Nginx on Ubuntu 20.04](https://phoenixnap.com/kb/how-to-install-nginx-on-ubuntu-20-04)

[How to Install and Connect to PostgreSQL on Ubuntu](https://www.liquidweb.com/kb/install-postgresql-ubuntu/)

[How to Install Samba in Ubuntu](https://phoenixnap.com/kb/ubuntu-samba)

[Installing Splunk Enterprise on Linux](https://www.youtube.com/watch?v=RaccexMm_Ts)
