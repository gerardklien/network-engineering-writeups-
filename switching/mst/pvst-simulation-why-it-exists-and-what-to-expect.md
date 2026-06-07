# PVST Simulation: Why It Exists and What to Expect in Modern Networks

## Overview

PVST Simulation is a Cisco interoperability mechanism that allows a **Cisco PVST+/Rapid-PVST+ domain** to coexist with a **Cisco MST region**.

It exists primarily to provide a migration path between Cisco's proprietary per-VLAN spanning-tree architecture and the IEEE-standard MST architecture.

---

## Historical Background

Before MST, Cisco networks commonly used:

* PVST
* Rapid-PVST+

These protocols provide:

* One spanning-tree instance per VLAN
* Per-VLAN load balancing
* Fast convergence (Rapid-PVST+)

Example:

```text
VLAN 10 Root = SW1
VLAN 20 Root = SW2
VLAN 30 Root = SW1
```

Many enterprise networks were built using this model.

---

## The Introduction of MST

IEEE introduced Multiple Spanning Tree (802.1s) to solve the scaling limitations of maintaining one spanning-tree instance per VLAN.

Instead of:

```text
1000 VLANs
=
1000 STP instances
```

MST allows:

```text
1000 VLANs
=
3 MST instances
```

Example:

```text
MSTI1 → VLANs 1-200
MSTI2 → VLANs 201-400
MSTI3 → VLANs 401-1000
```

Benefits include:

* Better scalability
* Lower CPU and memory utilization
* Fast convergence (RSTP-based)
* Efficient load balancing

---

## The Migration Problem

Consider an enterprise with:

```text
500 Cisco switches
running Rapid-PVST+
```

Replacing the entire campus with MST during one maintenance window is unrealistic.

Nobody wants to do:

```text
Friday 10 PM

Convert 500 switches to MST.

Pray.
```

---

## Cisco-to-Cisco Migration

### Phase 1

Entire campus:

```text
Rapid-PVST+
```

---

### Phase 2

New building or expansion:

```text
Rapid-PVST+
        |
        |
PVST Simulation
        |
        |
      MST
```

Both domains temporarily coexist.

---

### Phase 3

More switches are migrated:

```text
Rapid-PVST+
        |
PVST Simulation
        |
Growing MST Region
```

---

### Phase 4

Eventually:

```text
MST
|
|
MST
```

PVST simulation disappears.

---

## Why PVST Simulation Exists

PVST simulation was never intended to be the destination architecture.

Network engineers do not intentionally design networks because they want PVST simulation.

Instead, they say:

> "I have an existing Rapid-PVST network and need to connect a new MST region."

PVST simulation solves this temporary interoperability problem.

---

## Important Clarification

The migration story above only applies when the new infrastructure is also Cisco.

Example:

```text
Old Campus (Cisco Rapid-PVST+)
            ↓
New Building (Cisco MST)
```

PVST simulation enables a gradual migration.

---

## Multi-Vendor Networks Are Different

Suppose the new infrastructure is Juniper:

```text
Cisco Rapid-PVST+  ←→  Juniper MST
```

This is undesirable because:

* PVST+ is Cisco proprietary.
* Juniper does not speak PVST+.
* Juniper does not implement PVST simulation.

Instead, the migration should look like:

### Step 1

Old Cisco network:

```text
Rapid-PVST+
```

### Step 2

Convert Cisco side to MST:

```text
Cisco MST
```

### Step 3

Connect to the new vendor:

```text
Cisco MST ←→ Juniper MST
```

No PVST simulation is involved.

---

## Therefore...

A more accurate statement is:

> PVST simulation is primarily a Cisco-to-Cisco migration and interoperability feature that allows legacy PVST+/Rapid-PVST+ domains to coexist temporarily with MST regions.

Or more precisely:

> PVST simulation exists because Cisco had to provide a migration path from its proprietary PVST+/Rapid-PVST+ architecture to the IEEE-standard MST architecture without requiring customers to replace their entire Cisco campus at once.

---

## What Should a Network Engineer Expect Today?

### Small Cisco Networks

Common:

```text
Rapid-PVST+
```

or

```text
MST
```

---

### Large Cisco Networks

Most common:

```text
MST
```

---

### Multi-Vendor Networks

Most common:

```text
MST ←→ MST
```

Examples:

* Cisco ↔ Juniper
* Cisco ↔ Aruba
* Cisco ↔ HPE
* Juniper ↔ Extreme

Since MST is IEEE 802.1s, virtually every enterprise vendor supports it.

---

### Modern Data Centers

Most common:

```text
Layer 3 Fabric
VXLAN EVPN
```

Spanning Tree plays little or no role.

---

## When Will You Encounter PVST Simulation?

Usually when:

* A legacy Cisco Rapid-PVST network is being upgraded.
* Some parts of a Cisco campus have already migrated to MST while others have not.
* An acquired company still runs Rapid-PVST.
* There is temporary coexistence between old and new Cisco infrastructure.

Not because it is considered the ideal long-term design.

---

## Key Takeaway

> PVST Simulation is primarily a Cisco-to-Cisco migration and interoperability feature. Its purpose is to allow legacy Cisco PVST+/Rapid-PVST+ networks to coexist with newer MST regions during a phased transition. In modern enterprise environments, engineers should expect MST for large Layer-2 domains and RSTP/Rapid-PVST+ for smaller deployments, with PVST simulation being relatively uncommon and rarely part of the intended final architecture.
