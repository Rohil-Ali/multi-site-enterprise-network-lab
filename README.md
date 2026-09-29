# Multi-Site Enterprise Network Lab

A hands-on networking lab built in **Cisco Packet Tracer** to practise designing, configuring and troubleshooting an enterprise-style network.

The lab consists of a Headquarters site and a Branch site connected over a WAN. I’m building the network in stages, starting with the core infrastructure and gradually adding security, network services and automation.

---

## End Goal

The end goal is to build a multi-site enterprise network that includes:

- Multiple VLANs for different departments and services
- Inter-VLAN routing
- Dynamic routing between sites using OSPF
- A DMZ/server network
- ACLs to control traffic between network segments
- Basic switch security and hardening
- Network services such as DHCP and DNS
- Python/Netmiko automation for tasks such as configuration backups

The aim is to use the lab to practise the full process of building a network — configuring it, testing it, troubleshooting problems and eventually automating parts of it.

---

# Progress

## 1. Initial Topology

I created the initial multi-site topology in Cisco Packet Tracer with a Headquarters site and a Branch site.

The two sites are connected using a point-to-point serial WAN connection between `HQ-RTR` and `B1-RTR`.

![Network Topology](img/topology.png)

The current devices are:

### Headquarters

- `HQ-RTR` — Cisco ISR 4331
- `HQ-DIST-SW` — Cisco Catalyst 2960
- `HQ-DMZ-SW` — DMZ/server switch
- `HQ-STAFF-SW` — Staff access switch
- `HQ-MAN-SW` — Management access switch
- `HQ-WEB-SVR01` — Web/server
- `HQ-STAFF-PC1` — Staff PC
- `HQ-MAN-PC0` — Management PC

### Branch 1

- `B1-RTR` — Cisco ISR 4331
- `BR1-DIST-SW` — Distribution switch
- `BR1-STAFF-SW` — Staff access switch
- `BR1-STAFF-PC2` — Staff PC

---

## 2. VLANs

I separated the different network segments into their own VLANs:

| VLAN | Purpose | Network |
|---|---|---|
| 10 | Management | `192.168.10.0/24` |
| 20 | HQ Staff | `192.168.20.0/24` |
| 30 | DMZ / Servers | `192.168.30.0/24` |
| 40 | Branch Staff | `192.168.40.0/24` |

This gives each segment its own broadcast domain and provides a foundation for applying security controls between them later.

![VLAN Configuration](img/vlan-config.png)

---

## 3. Trunking

I configured the relevant links as **802.1Q trunks** so that traffic from multiple VLANs can travel across the same physical link while keeping the VLANs logically separated.

Example configuration:

```text
interface FastEthernet0/1
 switchport mode trunk
````

![Trunk Configuration](img/trunk-config.png)

I verified the trunk configuration using:

```text
show interfaces trunk
```

---

## 4. Inter-VLAN Routing

I used **Router-on-a-Stick** to allow the different VLANs to communicate through the router.

The physical router interface is divided into subinterfaces, with each one acting as the default gateway for a VLAN.

Example configuration:

```text
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

The Branch router uses the same approach for VLAN 40.

![Router Interfaces](img/router-interfaces.png)

I verified the interfaces using:

```text
show ip interface brief
```

---

## 5. OSPF

I configured **OSPF Area 0** between `HQ-RTR` and `B1-RTR`.

The routers are connected using the `10.0.0.0/30` WAN network.

Using OSPF means the routers can dynamically learn about the networks at the other site instead of relying on manually configured static routes.

I verified the OSPF neighbour relationship with:

```text
show ip ospf neighbor
```

![OSPF Neighbour](img/ospf-neighbor.png)

I also checked the routes being learned through OSPF using:

```text
show ip route ospf
```

![OSPF Routes](img/ospf-routes.png)

---

## 6. Connectivity Testing

Once the routing was configured, I tested connectivity across the network.

One of the end-to-end tests was from the Branch staff PC to the HQ web server:

```text
BR1-STAFF-PC2 -> 192.168.30.2
```

The traffic has to travel from the Branch network, through `B1-RTR`, across the WAN, through `HQ-RTR` and finally into the HQ server network.

![Connectivity Test](img/connectivity-test.png)

This confirmed that the VLANs, routing and WAN connection were working together correctly.

---

# Problems & Troubleshooting

## Native VLAN Mismatch

While connecting the downstream switches to `HQ-DIST-SW`, I started receiving CDP errors about a native VLAN mismatch.

The error looked similar to:

```text
%CDP-4-NATIVE_VLAN_MISMATCH:
Native VLAN mismatch discovered on FastEthernet0/1 (1),
with HQ-DIST-SW FastEthernet0/2 (20).
```

### What I found

The links between the switches were not configured consistently.

The distribution switch was using VLAN-specific configuration while the downstream switches were still using their default VLAN configuration.

### Fix

I configured the relevant switch-to-switch links as trunks:

```text
switchport mode trunk
```

I also made sure the required VLANs existed on the connected switches.

### What I learned

I initially assumed the downstream switches could be connected like simple access switches.

The issue showed me that I need to consider what traffic actually needs to cross each link and make sure the Layer 2 configuration is correct on both ends.

I also got some practical experience using CDP messages to help identify a configuration problem.

![Troubleshooting](img/troubleshooting.png)

> The troubleshooting screenshot shows the **current corrected configuration**, as I did not capture the original error while I was building the lab.

---

# What's Next

* [ ] Configure Extended ACLs
* [ ] Restrict Staff → Management traffic
* [ ] Restrict access to the DMZ/server network
* [ ] Test allowed and blocked traffic
* [ ] Configure basic switch port security
* [ ] Harden unused switch ports
* [ ] Add DHCP
* [ ] Add DNS
* [ ] Begin Python/Netmiko automation
* [ ] Automate configuration backups
* [ ] Automate device information collection

---
