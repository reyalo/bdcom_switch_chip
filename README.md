# bdcom_switch_chip
/##################################################################
2.5.1
Principle and Function of STP
The Spanning Tree Protocol (STP) is designed to prevent loops in Layer 2 networks (Ethernet). In redundant switch topologies, loops can cause broadcast storms, multiple frame copies, and MAC table instability. STP dynamically builds a loop-free tree by selectively blocking certain ports.

Mechanism:

Elects a Root Bridge.

Calculates the shortest path to the Root.

Blocks redundant paths by putting ports into non-forwarding states.



2.5.2
In Spanning Tree Protocol, ports transition through a defined set of states to safely build a loop-free network topology. Based on the guidance provided, the following are the official port states and their roles:

Port State	Description
Disable	The port is administratively down; no traffic or BPDU is processed.
Block	The port only receives BPDUs to monitor network topology; data traffic is blocked.
Listen	The port participates in active topology building, listening to BPDUs but not learning or forwarding.
Learn	The port starts learning MAC addresses and building the bridge table, but still does not forward frames.
Forward	The port is fully active; it forwards data traffic, learns MAC addresses, and processes BPDUs.

These state transitions ensure that only verified loop-free paths are activated in the network. Ports gradually move from "Block" to "Forward" as the topology stabilizes.

2.5.3
Difference Between MSTP and SSTP
Feature	MSTP (Multiple STP, IEEE 802.1s)	SSTP (Shared Spanning Tree Protocol, per IEEE 802.1Q)
Scope	Supports multiple VLANs in instances.	Uses a single STP instance for all VLANs.
Efficiency	Better resource utilization (groups VLANs).	Less efficient (one tree for all VLANs).
Convergence	Faster (per-instance computation).	Slower (single instance affects all VLANs).
Complexity	More complex configuration.	Simpler but less flexible.
2.5.4

2.5.5
Realtek RTL9310 STP Packet Handling:
1. Reserved Multicast MAC Table
Realtek hardware reserves certain destination MAC addresses for special handling.

STP packets use the well-known multicast MAC address:

01:80:C2:00:00:00

This MAC is part of the IEEE Reserved Multicast Range.
On RTL9310, this MAC address is matched in a reserved multicast trap table, and it is automatically trapped to the CPU unless configured otherwise.





2.6.1 
Principle of VLAN Translation and How It Works

VLAN translation refers to the process of modifying the VLAN tag of incoming or outgoing Ethernet frames to match different network segments. It enables interconnection between different VLAN domains.

How It Works:
When a frame arrives on a translation port, the switch checks the ingress VLAN ID.

Based on configured translation rules, the VLAN tag is rewritten to a different VLAN ID before further processing.

Similarly, for egress translation, the tag is again modified before the frame exits the switch.

This enables interoperability between VLAN schemes of different departments, ISPs, or customers.

2.6.2

Principle of QinQ (802.1Q-in-802.1Q) and VLAN Stacking
QinQ (IEEE 802.1ad) is an extension of the IEEE 802.1Q standard that supports VLAN stacking by inserting two VLAN tags into Ethernet frames.

✅ How QinQ Works:
A customer frame already tagged with a Customer VLAN (C-VLAN) enters a QinQ-enabled port.

The switch adds an outer tag (S-VLAN / Service VLAN), forming a double-tagged frame.

This allows traffic from multiple customers (with overlapping VLAN IDs) to be uniquely identified and forwarded by the provider network.


2.6.3

QinQ Mode vs Flat Mode in VLAN Translation
Mode                	Description	            Behavior
QinQ Mode           	Enables VLAN stacking (C-VLAN inside S-VLAN). Adds outer tag.	Frame becomes double-tagged;                        used in service provider core networks.
Flat Mode           	Translates one VLAN tag to another (no stacking).	Frame remains single-tagged;                                    original tag is replaced.

2.6.4
Bidirectional VLAN Translation and Hardware Table Entries
Principle:
Bidirectional VLAN translation allows a port to:

Translate Ingress VLAN A → VLAN B

Translate Egress VLAN B → VLAN A

This ensures consistent mapping in both directions across the same link.

Hardware Table Entries Involved:
Table	                            Purpose
VLAN Translation Table	            Stores mapping between original and translated VLAN IDs (both directions).
Port VLAN Config Table	            Defines whether the port is in translation/QinQ/flat mode.
Ingress VLAN Lookup Table	        Matches incoming VLANs to perform translation.
Egress VLAN Edit Table	            Rewrites VLAN ID before frame is transmitted.
2.6.5



2.7.1
1. Describe the role and principle of aggregation
Role:

Link Aggregation (LAG) groups multiple physical ports into one logical port.

Provides:

Bandwidth increase

Redundancy

Load balancing

Principle:

Each ingress or egress flow is hashed.

The hash result selects one member port in the group.

2.7.2
2.7.3
2.7.4
No, not exactly 1:1:1.

Reason:

Hash modulo 3 distributes flows as evenly as possible, but 1000 ÷ 3 = 333.33

So the distribution will be something like:

Port 1: 334 flows

Port 2: 333 flows

Port 3: 333 flows

Hash collisions and modulo remainder cause a small imbalance.


2.7.5
Yes, exactly.

Reason:

1000 ÷ 4 = 250

With a good hash algorithm, the flows can be evenly divided:

Port 1: 250

Port 2: 250

Port 3: 250

Port 4: 250

Assumes that:

Hash function is uniform and has enough entropy.

All ports are up and active.


2.7.6
Feature	Static Aggregation	Dynamic Aggregation
Protocol	No protocol involved	Uses LACP (Link Aggregation Control Protocol - IEEE 802.3ad)
Configuration	Manually configured	Negotiated dynamically between devices
Link negotiation	Not aware of peer configuration	Ensures consistency between both ends
Fault detection	Limited	Better failure detection and auto-recovery
Use case	Simpler setups, fixed links	More robust, for dynamic environments



2.7 Port Link Aggregation
1. Describe the role and principle of aggregation;
2. Describe the aggregation group member table entries of the corresponding chip of the aggregation group, how the flow is hashed and how to decide which physical port to forward from;
3. In L2_entry, which bit indicates that the port is an aggregation port?
4. If the aggregation group has 3 physical port members, play 1000 evenly distributed hash flows, can the flow of each physical port be 1:1:1;
5. If the aggregation group has 4 physical port members, play 1000 evenly distributed hash flows, can the flow of each physical port be 1:1:1:1;
6. What is the difference between static aggregation and dynamic aggregation?
7. Configure a static aggregation group, describe how to view the aggregation group and HASH method under the shell?
