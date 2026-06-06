# Lab Observation: MST Interoperability with PVST+ and Rapid-PVST+

## Lab Topology

The following topology was used throughout the experiment.

<p align="center">
  <img width="910" height="560" alt="image" src="https://github.com/user-attachments/assets/cba71424-36f2-4e79-b24b-23199f59123a" />
</p>

Switches SW1, SW2, SW3, SW4 and SW6 belong to MST Region NETBITS.

SW6 was configured as the CIST Root Bridge.

Switch5 was configured under two different scenarios:

1. PVST+ (802.1D)
2. Rapid-PVST+

Packet captures were taken on SW5's interface facing SW2.

---

# Objective

The purpose of this lab was to investigate how Cisco MST interoperates with non-MST switches and to validate several commonly repeated statements regarding BPDU field manipulation.

---

# Observation #1 – MST to PVST+ (802.1D)

<p align="center">
<img width="936" height="585" alt="image" src="https://github.com/user-attachments/assets/31f1b76d-2f72-4746-8a96-9ba2cdd9ad24" />
</p>
SW5 was configured for:

```cisco
spanning-tree mode pvst
```

The MST boundary port on SW2 was identified as:

```text
P2p Bound(STP)
```

indicating that SW2 recognized SW5 as a legacy STP neighbor.

Packet capture on SW5 revealed:

```text
Protocol Version Identifier = 0 (STP)

Root ID        = 5000.0006.0000 (SW6)
Root Path Cost = 0
Bridge ID      = 5000.0002.0000 (SW2)
```

Therefore:

```text
Root ID        = CIST Root Bridge
Bridge ID      = Boundary Switch
Root Path Cost = 0
```

Although the BPDU was transmitted by SW2, the Root Identifier correctly pointed to SW6, which was the CIST Root Bridge.

Interestingly, the Bridge Identifier was not rewritten to the CIST Root Bridge ID, but instead remained the BID of the transmitting boundary switch.

---

# Observation #2 – MST to Rapid-PVST+
<p>
  <img width="959" height="664" alt="image" src="https://github.com/user-attachments/assets/a7b83836-6091-4181-9765-e912287c611a" />
  <img width="851" height="222" alt="image" src="https://github.com/user-attachments/assets/2532e3d5-924e-4c1c-9b30-066a2a4cab2a" />
</p>

SW5 was then configured for:

```cisco
spanning-tree mode rapid-pvst
```

Packet captures revealed:

```text
Protocol Version Identifier = 3 (MSTP)

Root ID        = 5000.0006.0000 (SW6)
Root Path Cost = 0
Bridge ID      = 5000.0006.0000 (SW6)
```

Furthermore, examination of the MST extension revealed:

```text
CIST Bridge Identifier = 5000.0002.0000 (SW2)
```

Thus:

```text
Root ID        = CIST Root Bridge (SW6)
Bridge ID      = CIST Root Bridge (SW6)
CIST Bridge ID = Boundary Switch (SW2)
Root Path Cost = 0
```

This behavior appears to be associated with Cisco's PVST Simulation mechanism.

---

# Validation Against Cisco Documentation

Cisco documentation states:

> "The MST region appears as a single virtual bridge to adjacent CST bridges."

and

> "The MST region transmits BPDUs with the CIST root identifier and the external root path cost."

These statements were validated by packet captures.

Specifically:

* Root ID advertised toward the non-MST switch was always the CIST Root Bridge (SW6).
* Since SW6 itself was the CIST Root inside the region, the external root path cost was zero.

However, Cisco documentation does not explicitly discuss how the Bridge Identifier field is populated in each interoperability scenario.

---

# Summary of Observations

## MST → PVST+ (802.1D)

Observed:

```text
Root ID        = SW6
Bridge ID      = SW2
Root Path Cost = 0
Protocol Version = 0
```

---

## MST → Rapid-PVST+

Observed:

```text
Root ID        = SW6
Bridge ID      = SW6
Root Path Cost = 0
Protocol Version = 3
MST Extension present

CIST Bridge Identifier = SW2
```

---

# Conclusion

The commonly repeated statement:

> "When the CIST Root resides inside an MST region, the sender BID is rewritten to the CIST Root BID."

was found to be an oversimplification.

Lab results indicate that BPDU behavior depends on the STP flavor of the neighboring switch.

When interoperating with PVST+ (802.1D), the transmitting boundary switch retained its own Bridge Identifier.

When interoperating with Rapid-PVST+, the transmitted BPDU used the CIST Root Bridge ID as both the Root Identifier and Bridge Identifier, while the MST extension still carried the CIST Bridge Identifier corresponding to the boundary switch.

These observations highlight the importance of validating protocol behavior through packet captures rather than relying solely on generalized descriptions found in training materials and forum discussions.

Further investigation against IEEE 802.1Q and additional Cisco documentation may be warranted to determine whether this behavior is implementation-specific or mandated by the standard.
