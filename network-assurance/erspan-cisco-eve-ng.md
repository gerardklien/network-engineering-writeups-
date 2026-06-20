# ERSPAN Configuration on Cisco CSR1000v in EVE-NG

## Overview

This lab demonstrates a basic ERSPAN configuration using Cisco CSR1000v routers in EVE-NG.

The goal is to mirror traffic from a source interface on one router and send the mirrored traffic across a routed Layer 3 network to another router where packet capture is performed.

In this lab:

- CSR4 is the ERSPAN source
- CSR5 is the ERSPAN destination
- OSPF provides basic Layer 3 reachability
- VPC7 generates ICMP traffic
- Packet capture on the destination side confirms that ERSPAN is working

---

## Lab Topology

<p align="center">
  <img width="1452" height="503" alt="erspan-topology" src="https://github.com/user-attachments/assets/75814e9a-6a1e-43e3-a46d-94d434ef6d6e" />
</p>

The lab topology is:

```text
VPC7 ---- CSR4 ---- CSR ---- CSR5 ---- VPC8
```

The addressing used in the lab is:

```text
VPC7
  IP Address: 10.10.20.10/24

CSR4
  Gi2: 10.10.20.1/24
  Gi1: 10.10.30.4/24

Middle CSR
  Gi2: 10.10.30.1/24
  Gi4: 10.10.40.1/24

CSR5
  Gi3: 10.10.40.5/24
  Gi1: 10.10.10.1/24

VPC8
  IP Address: 10.10.10.10/24
```

The traffic being monitored is between VPC7 and CSR4:

```text
VPC7 10.10.20.10  <-->  CSR4 Gi2 10.10.20.1
```

---

## What ERSPAN Solves

SPAN is used when the analyzer is local to the same device.

RSPAN extends SPAN across a Layer 2 network using a dedicated RSPAN VLAN.

ERSPAN extends SPAN across a Layer 3 routed network.

```text
SPAN   = Local mirroring
RSPAN  = Remote mirroring across Layer 2
ERSPAN = Remote mirroring across Layer 3
```

In this lab, the analyzer is not directly connected to the source traffic segment.

The source traffic exists on the CSR4 side, while the packet capture is performed on the CSR5 side. Because the mirrored traffic must cross a routed network, ERSPAN is the correct feature to use.

---

## Lab Objective

The objective is to mirror traffic from CSR4's interface facing VPC7.

```text
Source interface:
CSR4 GigabitEthernet2
```

CSR4 sends the mirrored traffic to CSR5 using ERSPAN.

```text
ERSPAN Source:       CSR4
ERSPAN Destination:  CSR5
Destination IP:      10.10.40.5
Origin IP:           10.10.30.4
ERSPAN ID:           100
```

The logical flow is:

```text
VPC7 ICMP traffic
      ↓
CSR4 Gi2
      ↓
ERSPAN source session
      ↓
Routed OSPF network
      ↓
CSR5 ERSPAN destination session
      ↓
Packet capture
```

---

## Routing Requirement

ERSPAN requires IP reachability between the source and destination devices.

In this lab, all routers run OSPF single area 0. OSPF is only used to advertise the connected routes and provide basic reachability between the routers.

CSR4 must be able to reach CSR5's ERSPAN destination IP:

```text
10.10.40.5
```

CSR5 must also have reachability back to CSR4's ERSPAN origin IP:

```text
10.10.30.4
```

Without Layer 3 reachability, ERSPAN will not work.

---

## CSR4 ERSPAN Source Configuration

CSR4 is configured as the ERSPAN source.

It monitors both inbound and outbound traffic on `GigabitEthernet2`.

```text
conf t
!
monitor session 1 type erspan-source
source interface GigabitEthernet 2 both
no shutdown
!
destination
ip address 10.10.40.5
origin ip address 10.10.30.4
!
erspan-id 100
end
!
```

Important parts:

```text
source interface GigabitEthernet2 both
```

This tells CSR4 to mirror both received and transmitted traffic on Gi2.

```text
ip address 10.10.40.5
```

This is the ERSPAN destination IP address on CSR5.

```text
origin ip address 10.10.30.4
```

This is the source/origin IP address used by CSR4 for the ERSPAN session.

```text
erspan-id 100
```

This identifies the ERSPAN session. The same ERSPAN ID must be configured on the destination side.

---

## CSR5 ERSPAN Destination Configuration

CSR5 is configured as the ERSPAN destination.

It receives ERSPAN traffic sent to `10.10.40.5` and forwards the mirrored packets out `GigabitEthernet1`.

```text
conf t
!
monitor session 1 type erspan-destination
destination interface GigabitEthernet 1
no shutdown
!
source
ip address 10.10.40.5
erspan-id 100
end
!
```

Important parts:

```text
destination interface GigabitEthernet1
```

This is the interface where CSR5 sends the mirrored packets after receiving the ERSPAN traffic.

```text
source
 ip address 10.10.40.5
```

This tells CSR5 to accept ERSPAN traffic sent to its local IP address.

```text
erspan-id 100
```

This must match the ERSPAN ID configured on CSR4.

---

## Traffic Generation

To test the ERSPAN session, VPC7 continuously pings CSR4.

The command used on VPC7 is:

```text
VPCS> ping 10.10.20.1 -t
```
Successful replies were received:

<p align="left">
 <img width="620" height="261" alt="image" src="https://github.com/user-attachments/assets/e9d84d92-7e45-4876-b218-e01da7b6c11a" />
</p>

This confirms that live ICMP traffic exists on the monitored segment.

```text
10.10.20.10 → 10.10.20.1
10.10.20.1  → 10.10.20.10
```

---

## Packet Capture Verification

Packet capture was performed from the ERSPAN destination side.

<p align="center">
  <img width="1629" height="476" alt="image" src="https://github.com/user-attachments/assets/2cfd07a0-efe8-4b74-ac95-a64ef96ba619" />
</p>

**Note:** Wireshark may display the inner packet as ICMP in the packet list, but expanding the packet details confirms it is encapsulated inside GRE/ERSPAN from `10.10.30.4` to `10.10.40.5`.

<p align="center">
 <img width="1013" height="328" alt="image" src="https://github.com/user-attachments/assets/772d051d-a82f-4ac2-b243-81b8440b4a8a" />
</p>

The capture shows ICMP echo requests and replies between VPC7 and CSR4:

```text
10.10.20.10 → 10.10.20.1   ICMP Echo Request
10.10.20.1  → 10.10.20.10  ICMP Echo Reply
```

This confirms that ERSPAN is working.

The packet capture was not taken directly from the `10.10.20.0/24` segment. The analyzer was on the CSR5 side of the topology.

Without ERSPAN, the analyzer on the CSR5 side should not see the local ICMP exchange between VPC7 and CSR4.

Since the analyzer can see traffic between `10.10.20.10` and `10.10.20.1`, CSR4 successfully mirrored the traffic and transported it across the routed network using ERSPAN.

---

## Why This Confirms ERSPAN Works

The original traffic exists on the left side of the topology.

```text
VPC7 <--> CSR4
```

The packet capture is taken on the right side of the topology.

```text
CSR5 side
```

The proof chain is:

```text
VPC7 generates ICMP traffic
      ↓
CSR4 sees the traffic on Gi2
      ↓
CSR4 mirrors Gi2 using ERSPAN
      ↓
Mirrored traffic crosses the routed network
      ↓
CSR5 receives the ERSPAN traffic
      ↓
Packet capture shows the original ICMP packets
```

That is the main purpose of ERSPAN.

It allows traffic from one part of the network to be mirrored and analyzed from another part of the network, even when the source and analyzer are separated by Layer 3 routing.

---

## Useful Verification Commands

Check the ERSPAN session:

```text
show monitor session 1
```

Check routing:

```text
show ip route
```

Check OSPF neighbors:

```text
show ip ospf neighbor
```

Test reachability from CSR4 to CSR5:

```text
ping 10.10.40.5 source 10.10.30.4
```

---

## Troubleshooting Notes

If ERSPAN does not work, check the following:

- Is the monitored source interface correct?
- Is the ERSPAN destination IP reachable?
- Is the origin IP address correct?
- Are both sides using the same ERSPAN ID?
- Is the monitor session enabled with `no shutdown`?
- Is OSPF advertising the required connected routes?
- Is the analyzer connected to the correct destination interface?
- Is packet capture running on the correct interface?
- Is any ACL or firewall blocking ERSPAN/GRE traffic?

---

## Key Takeaway

ERSPAN allows mirrored traffic to be transported across a routed Layer 3 network.

Unlike RSPAN, ERSPAN does not require a dedicated RSPAN VLAN.

In this lab, CSR4 mirrored ICMP traffic from the `10.10.20.0/24` segment and sent it to CSR5 across an OSPF-routed network.

```text
CSR4 Gi2 traffic
      ↓
ERSPAN source
      ↓
Routed OSPF network
      ↓
CSR5 ERSPAN destination
      ↓
Packet capture
```

The packet capture showing ICMP traffic between `10.10.20.10` and `10.10.20.1` confirms that the ERSPAN session is working.
