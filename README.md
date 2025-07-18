# bdcom_switch_chip

Topics:

2.5 spanning tree
1.Describe the principle and function of stp;
2.Describe several STP states of the port;
3.Describe the difference between MSTP and SSTP;
4.Describe the table items involved in sstp, rstp, and mstp chips;
5.Through which hardware table items does the stp protocol message receive the CPU?

2.6 VLAN Translation, Dot1q-Tunnel
1.Describe the principle of vlan translation;
2.Describe the principle of QinQ
3.Describe the behavior of QinQ mode and flat mode of VLAN translation on VLAN;
4.Describe how vlan translation achieves two-way translation, and their corresponding table items;
5.Describe the principle and table items of enabling global dot1q-tunnel, configuring the uplink port, and the corresponding chip of the translation port;

2.7 Port Link Aggregation
1.Describe the role and principle of aggregation;
2.Describe the aggregation group member table entries of the corresponding chip of the aggregation group, how the flow is hashed and how to decide which physical port to forward from;
3.In L2_entry, which bit indicates that the port is an aggregation port?
4.If the aggregation group has 3 physical port members, play 1000 evenly distributed hash flows, can the flow of each physical port be 1:1:1;
5.If the aggregation group has 4 physical port members, play 1000 evenly distributed hash flows, can the flow of each physical port be 1:1:1:1;
6.What is the difference between static aggregation and dynamic aggregation?
7.Configure a static aggregation group, describe how to view the aggregation group and HASH method under the shell?

2.8 Layer 2 Multicast
1.Describe the chip physical table entries corresponding to layer 2 multicast;
2.Configure a static multicast MAC address, mac address-table static 0100.0001.0001 vlan 1 interface g0/1 g0/2; describe how to view the multicast table and forwarding port in the shell;


2.10 Port Mirroring
1.Describe the chip table entries corresponding to ingress mirroring, and the principle;
2.Describe the chip table entries corresponding to egress mirroring, and the principle;
3.Describe the general principle of flow-based mirroring;

2.11 Port Isolation
1.Describe the port isolation function and the corresponding chip table entries;
2.The difference between global port isolation and group-based port isolation; describe the requirements for group-based port isolation on the table items?

2.12 Storm Control
1.Describe the chip register table entry corresponding to storm control storm-control;

2.13 Port Speed Limit
1.Describe the chip register table items corresponding to the port speed limit function, or describe the leaky bucket principle of the chip to implement the speed limit function, choose one of the two;



#####################################################################################
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

How QinQ Works:
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

Principle:
Enabling global Dot1Q tunneling allows the switch to globally recognize and handle double-tagged frames using QinQ logic.

Uplink Port Configuration:
The uplink port (provider-facing port) must be configured as a QinQ tunnel port.

The port adds or recognizes an outer S-VLAN tag to encapsulate or decapsulate frames.

Tunnel Ethertype (0x88a8) is set globally to distinguish S-VLAN tags.

Hardware Table Entries and Resources:
Component	Function
Global QinQ Enable Flag	Enables QinQ logic globally (affects Ethertype parsing and tagging logic).
Port Type Table	Marks uplink ports as tunnel ports for QinQ processing.
S-VLAN Table / VLAN Edit Table	Inserts/removes outer tag based on port role.
TPID Table	Stores Tunnel Protocol Identifier (0x88a8) for S-VLAN handling.
Ingress Parser + Egress Modifier	Responsible for tag parsing and insertion/removal in datapath.




2.7.1
Describe the role and principle of aggregation
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
Describe the aggregation group member table entries of the corresponding chip of the aggregation group, how the flow is hashed and how to decide which physical port to forward from
Chip Table Structure:
In switch ASICs (like RTL93xx), there are dedicated hardware tables:

Trunk group table or aggregation group member table: defines which physical ports are part of which trunk group (e.g., trunk ID 3 → ports 5, 6, 7).

Hash function control register: configures which fields are used for hashing.

Flow Hashing Mechanism:

ASIC computes a hash key using fields like:

L2: Src MAC, Dst MAC

L3: Src IP, Dst IP

L4: Src Port, Dst Port (for TCP/UDP)

The hash result (e.g., 8-bit value) is modulo-ed by the number of active members in the LAG.

The resulting index selects a physical port from the group.

Example:
If hash result is 0x23 and 3 ports are in the group:

plaintext
Copy
Edit
selected_index = hash_result % 3 = 0x23 % 3 = 2
→ Choose 3rd port from group members
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



2.8 Layer 2 Multicast
2.8.1. Describe the chip physical table entries corresponding to layer 2 multicast;
In most Ethernet switch ASICs (such as RTL931x), Layer 2 multicast forwarding is handled via dedicated multicast hardware tables, separate from unicast L2 entries.

Field   	        Description
MAC          	    Multicast MAC (e.g., 01:00:5e:xx:xx:xx)
VLAN ID   	        VLAN context of the multicast group
Type            	Whether it's a static/dynamic entry
Port mask       	Bitmap of egress ports or group ID for hardware replication


2.8.2. Configure a static multicast MAC address, mac address-table static 0100.0001.0001 vlan 1 interface g0/1 g0/2; describe how to view the multicast table and forwarding port in the shell;

configuration:
mac address-table static 0100.0001.0001 vlan 1 interface g0/1 g0/2

What happens inside the ASIC:
    1.The multicast MAC + VLAN is inserted into the L2 table.
    2. A multicast index is allocated.
    3. That index maps to a port bitmap with ports g0/1 and g0/2 set.

showing multicast table:
show mac address-table multicast




2.10
Port mirroring allows to copy traffic from one or more source ports or VLANs and send it to a monitor port for analysis (e.g., using Wireshark)

2.10.1 
Ingress Mirroring – Chip Table & Principle
Chip Table Entry Used:
Port Configuration Table (Port-Based Control Table)

Each ingress port has a mirror enable flag and a mirror-to port ID field.

Principle:
When a packet enters the switch (ingress):

The switch looks up the ingress port configuration.

If ingress mirroring is enabled:

The packet is cloned.

The original packet continues normal processing (e.g., forwarding via L2/L3 logic).

The mirrored copy is sent to the designated mirror port.

Mirror port must be configured in egress-only mode (not used for normal switching).

Key Chip Fields:
INGRESS_MIRROR_ENABLE (per port)

MIRROR_DEST_PORT (global or per port depending on platform)

CLONE_CTRL for packet copy operation

2.10.2
Egress Mirroring – Chip Table & Principle
Chip Table Entry Used:
Egress Port Control Table

Contains mirror enable flags per port and possibly mirror session IDs.

Principle:
When a packet is about to be transmitted on an egress port:

The switch checks if egress mirroring is enabled for that port.

If yes:

A copy of the packet is sent to the mirror port.

The original packet is still transmitted normally.

Key Chip Fields:
EGRESS_MIRROR_ENABLE (per port)

MIRROR_DEST_PORT

Possibly mirror flags in Egress Scheduler/Queue Management module

2.10.3 
Flow-Based Mirroring – General Principle
Flow-Based = Match certain packets by rules, not just port
Used Table(s):
ACL Table / Policy Engine Table / Flow Table

Matches packets based on:

Source/destination IP

L4 ports

Protocol

VLAN ID

Ingress port

MAC address, etc.

Principle:
The switch uses a flow classifier (ACL or TCAM match):

If a packet matches a configured flow-based mirror rule:

It is cloned.

Copy is sent to the mirror port.

Original packet proceeds as usual.

This is often more fine-grained than port mirroring.

Key Fields in ACL Table:
MATCH_FIELDS: fields to match (IP, port, protocol, etc.)

ACTION: mirror, drop, forward, count, etc.

MIRROR_PORT_ID: port to which mirrored copy is sent



2.11 Port Isolation

Port Isolation Function and Chip Table Entries
Function:
Blocks traffic between two ports (usually within the same VLAN).

Even if MAC address learning and forwarding logic allows a packet from Port A to Port B, isolation logic overrides forwarding to prevent delivery

Global vs Group-Based Port Isolation
Global Port Isolation
One global rule applies across all isolated ports.
All isolated ports cannot communicate with each other, but can talk to uplink or gateway.

Group-Based Port Isolation
Ports are divided into isolation groups.

Ports in the same group cannot talk to each other.

Ports in different groups may or may not be allowed depending on configuration.

More flexible than global isolation.

Table Requirements for Group-Based Isolation:
Field	                Description
PORT_GROUP_ID	        An identifier for each isolation group (e.g., 0–7)
GROUP_ISOLATION_MASK[n]	Defines allowed egress ports per group
GROUP_EN	            Enable group-based logic in isolation module

2.12 
Chip Register/Table Entry Corresponding to Storm Control (storm-control)
Storm Control is used to prevent excessive broadcast, multicast, or unknown unicast traffic from overwhelming the switch by limiting their rate per port.

Key Registers/Tables Typically Involved:
These can vary by chip vendor, but generally include:

Table/Register Name	            Field / Description
STORM_CTRL_CFG (per-port)	    Enables/disables storm control for broadcast/multicast/unknown-unicast
STORM_CTRL_RATE_LIMIT	        Specifies rate limit threshold in kbps or packets per second
STORM_CTRL_BUCKET_SIZE	        Specifies burst size (how many packets/bytes can be received in burst)
STORM_CTRL_ACTION	            Action on exceed: drop or trap to CPU


2.13

How Port Speed Limiting Works in the Chip
There are two methods:

Method 1: Chip Registers (Rate Limit Configuration)

What the chip does:
For each port, it has registers like:
PORT_RATE_LIMIT_ENABLE
PORT_RATE_LIMIT_MAX_RATE (e.g., 5 Mbps)
PORT_RATE_LIMIT_ACTION (drop or delay)

For each packet:
It calculates how many bytes passed in a given time (e.g., last 1 second).
Compares with threshold (say, 625 KB for 5 Mbps).
If under limit → forward packet
If over limit → drop packetor delay
This is usually based on counters + timer inside the chip.

Method 2: Leaky Bucket Algorithm Description
Leaky Bucket is a commonly used algorithm for rate limiting. Here's how it works:

Leaky Bucket Components:
Bucket size (B) – Maximum burst allowed
Leak rate (r) – Constant rate at which packets are allowed to pass
Incoming packets – Treated as water poured into the bucket

Working Principle:
Packets arrive and are stored in a buffer (bucket).
The bucket leaks at a fixed rate (configured speed limit).
If the arrival rate exceeds leak rate and the bucket overflows → packets are dropped.

So this ensures that average bandwidth doesn’t exceed the limit but allows short bursts within the bucket size.





Got it! Here's the corrected version:

---

Topics Covered This Week:

2.5 Spanning Tree
2.6 VLAN Translation, Dot1q-Tunnel
2.7 Port Link Aggregation
2.8 Layer 2 Multicast
2.10 Port Mirroring
2.11 Port Isolation
2.12 Storm Control
2.13 Port Speed Limit

---

Tasks for Next Week:

2.9 Port Security
2.14 Queue Scheduling Policy
2.15 Flow Control
2.16 Access List Principle
2.17 QoS Policy-Map
2.18 DHCP Snooping
2.19 Filter Security Function
2.20 CPU Receives Protocol Messages
2.21 CPU Receives Priority Processing of Protocol Messages
3.1 L3 Layer Message Forwarding Process

---

Let me know if you want this in document format or added to a presentation.



3.3.2
Port-based Isolation
When hosts are connected to router through switch ports, they can utilize Port Isolation to force the
communication between hosts through the router. Port isolation is a mechanism to provide a CPU
configurable destination port mask for an ingress port. For each ingress port, the device provides a
port isolation port mask configuration defined in Figure 3-3. Bit value 1 of the port mask means the
port can communicate with each other. Bit value 0 is that the port can't forward any packet. No matter
the received packets are known/unknown Unicast, known/unknown Multicast,Broadcast packets. All
of the packets are limited by the configuration of Port Isolation port mask in forwarding.
Table 3-54 1
PORT_ISO_CTRL Register
Bits Field Description
63:7 P_ISO_MBR_0 Port isolation configuration that each port can specify a port list to
communicate with.

For example as Figure 3-3, hosts A and B are connected to ports 0 and 3 separately while router is
connected to port 7. The traffic between A and B must be forwarded by router. Therefore, user can
configure the 7th bit of Port isolation port mask of ports 0 and 3 to be 1 while other bits should be
configured to 0, and Port Isolation port mask of port 7 still remains all ports. Then, direct
communication between downlink ports, 0 and 3, is forbidden and only traffic between uplink port
and downlink port can be forwarded.






3.3.3
VLAN-based Isolation
Sometimes, uplink ports communicate with uplink port and downlink ports, but downlink ports don't
be allowed to communicate with each other in VLAN domain. Customer can use VLAN-based Isolation
to achieve the application. The device provides 32 sets VLAN-based Isolation configurations in Table
3-56.

Table 3-56
PORT_ISO_VB_ISO_PMSK_CTRL Register
Bits    Field       Description
88:32   VB_ISO_MBR  VLAN-based isolated port mask (port 56 to 0). Ports which have corresponding port mask clear(i.e. 0) in this mask could not communicate with each other, while ports which have corresponding port mask set could communicate with any other ports.
24:13   VID_UPPER   Upper bound VLAN ID
12:1    VID_LOWER   Upper bound VLAN ID
0:0     VALID       Indicates whether this entry is valid, 0: entry invalid, 1: entry valid.

For every packet, system will check all the 32 sets if matched or not. The matched condition is: the VALID field is valid and forwardingVID in the range [VID_UPPER,VID_LOWER]. If matched (multiple hit, select the lowest index set), the field VB_ISO_MBR will be referred, The ports with bit value 0 in VB_ISO_MBR can't communicate with other ports having same bit value 0.
However, they can communicate with the other port with bit value 1. If bit value of ports is set to 1, the port can communicate to all ports in theVLAN. Sometimes, bit value of uplink port is set to 1 and

bit value of downlink port is set to 0 in network environment. For example as Figure 3-4, ports 0 and 1
are uplink ports and other ports are downlink ports. Only configure (VB_ISO_MBR) of VLAN-based
lsolation entry to be (0x3), then uplink ports can communicate to all ports, but downlink ports only
communicate to uplink port.

Unlike the port-based isolation, the VLAN based isolation always has no effect on the routing packet,
including unicast routing and multicast routing packet. But both the unicast and multicast bridged
packet are still restricted.


3.4 Trunk
3.4.1 0verview
Trunk also named link aggregation is used to utilize bandwidth effectively and improve network
resilience by aggregating several physical ports into a logical port.
The system supports max 26 trunk groups in stand-alone mode and max 128 trunk groups in stacking
mode for normal ports. Each of the group can aggregate at most 8 physical ports across stacking
devices.
Besides that, System also supports link aggregation of stacking ports to at most 8 groups with
maximum 8 member ports for each group.

3.4.2
Stand-Alone And Stacking Trunk Mode
System supports 2 mode of trunk operation: stand-alone and stacking mode.
TRK_STAND_ALONE_MODE register field globally decides if trunks work in stand alone or stacking mode.

Table 3-57 TRK_CTRL Register
Bits    Field                   Description
2:2     TRK_STAND_ALONE_MODE    Configure the enable status of stand-alone mode for normal port link aggregation. In stand alone mode, hardware would skip the link down trunk member ports. Bit value: 0b0: disable stand-alone mode, 0b1: enable stand-alone mode

The characteristics of two modes are:

Table 3 58 Differences of Stand-Alone and Stacking Mode
Trunk mode      Number of trunks    Port Link Change Fail-Over      Other Difference
Stand-Alone     26                  Hardware Supported
Stacking        128                 Software Handle                 Supported local first function

In stand-alone mode, the link down ports of trunk egress ports would be excluded from traffic TX ports
by hardware. However, in stacking mode, since hardware could not detect link status of ports on
remote switch, hardware would disable trunk member link change fail-over mechanism and software
should handle it.



3.4.3
Normal Port Trunk
Packet received from one physical port will be processed by the ingress process phase, to decide
whether the RX port belongs to a single trunk or not which would affect L2 learnt port/trunk ID and
source port filtering. While packet is going out through one trunk, the egress process phase will do the
load sharing operation to decide the finally outgoing physical port.
While setting a normal port trunk, both the ingress and egress member ports should be set right. The
ingress member ports settings locate in the Source Port Mapping Table (also known as Ingress Trunk
Table) and local trunk portmask. The egress member set is fulfilled by the Egress Trunk Table.
In general, for the same trunk group, member ports in the Egress Trunk Table entry should be identical
to the Ingress Trunk Table entry, but for some protocol application, our system supports that egress
member set is less than ingress member set.



2:2 
TRK_STAND_ALONE_
 MODE 
Configure the enable status of stand-alone mode for normal port link 
aggregation. In stand-alone mode, hardware would skip the link down 
trunk member ports. 
Bit value: 
0b0: disable stand-alone mode 
0b1: enable stand-alone mode



3.4.3.1 Ingress Process
System provides 1024 Source Port Mapping Table (Ingress Trunk Table) entries, which is used to 
configure different device and port belongs to a trunk or not. The format of the entry is as follows. 

Table 3-59 Source Port Mapping Table Entry 
Fields                  Description  
TRK_ID_VALID            Specify if TRK_ID is valid or not. If TRK_ID is valid, the port is a  trunk member port of the trunk. 
TRK_ID                  ID of link aggregation group of source port.


While a packet is received from normal or stacking port, system will use KEY <DEV+PORT> to decide 
whether if it is from trunk and the corresponding TRK_ID. The KEY <DEV> represents the stacking 
device ID (more detailed information, please refer to stacking develop guide). The entry index would 
be DEV*64 + PORT. The result TRK_ID can be used to learn into L2 table as the source port and so on. 
For example, if a packet is received from local port 3, and the DEV configuration of the switch is 0, it 
would lookup Source Port Mapping Table of index (0*64 + 3) = 3 to decide if it is a trunk member port 
and the trunk ID



Table 3-60 TRK_ID_CTRL Register 
Bits        Field           Description 
7:7         TRK_ID_VALID    Specify if TRK_ID is valid or not. If TRK_ID is valid, the TRK_ID decides the trunk ID and TRK_PPM decides the local member ports. 
6:0         TRK_ID          ID of link aggregation group of source port. 


Table 3-61 TRK_MBR_CTRL Register 
Bits        Field       Description 
55:0        TRK_PPM     Local member portmask of link aggregation group. Bit value: 0b0: individual port, 0b1: individual port


3.4.3.2 Egress Process 
When packets are going out through one trunk, packets will do load balancing within member ports. 
The final outgoing port is decided by the trunk member port set, load sharing algorithm selected, 
traffic separation and local first if set. 
 
Egress Member Port 
System provides Egress Trunk Table to configure egress member port. Per Egress Trunk Table entry has 
8 (TRK_DEV, TRK_PORT) configurations that denotes trunk TX candidate ports. At most 8 ports can be 
configured as member port.  
The Egress Trunk Table has 128 entries in total. Per Entry format is as follows.


Table 3-62 Egress Trunk Table Entry
Fields          Description 
NUM_TX_CANDI    Number of valid TX candidate ports of this trunk group. 
TRK_DEV7        Device ID 7 
TRK_PORT7       Trunk port 7 
TRK_DEV6        Device ID 6 
TRK_PORT6       Trunk port 6 
TRK_DEV5        Device ID 5 
TRK_PORT5       Trunk port 5 
TRK_DEV4        Device ID 4 
TRK_PORT4       Trunk port 4 
TRK_ DEV3       Device ID 3 
TRK_PORT3       Trunk port 3 
TRK_ DEV2       Device ID 2 
TRK_PORT2       Trunk port 2 
TRK_DEV1        Device ID 1 
TRK_PORT1       Trunk port 1 
TRK_DEV0        Device ID 0 
TRK_PORT0       Trunk port 0 
L2_HASH_MSK_IDX Index of hash parameters mask of a trunk of L2 packets. (index to L2_HASH_MASK) 
IP4_HASH_MSK_IDX Index of hash parameters mask of a trunk of IPv4 packets. (index to L3_HASH_MASK) 
IP6_HASH_MSK_IDX Index of hash parameters mask of a trunk of IPv6 packets. (index to L3_HASH_MASK)


able 3-71 RMA_PORT_BPDU_CTRL 
Bits, Field, Description 
2:0, ACT, BPDU (01-80-C2-00-00-00) Configuration. 0x0: Forward 0x1: Drop 0x2: Trap to local CPU 0x3: Trap to master CPU 0x4: Flood(limited by RMA_BPDU_FLD_PMSK) 0x5 0x7: Reserved Note: When it is set to Trap, BPDU packet should be able to bypass VLAN.





3.5.3 User Defined RMA 
The device supports 4 user defined RMA settings. Using them, you can configure specific MAC address 
as RMA. The fields of each user defined RMA setting are shown as Table 3-78. The whole 48 bits can be 
configured in user defined RMA. 

Table 3-78 RMA_USR_DEF_CTRL Register 
Bits Field Description 
143:128, ADDR_MAX_HI, The address [47:32] of maximum boundary of user-defined RMA. 
127:96, ADDR_MAX_LO, The address [31:0] of maximum boundary of user-defined RMA. 
79:64, ADDR_MIN_HI, The address [47:32] of minimum boundary of user-defined RMA. 
63:32, ADDR_MIN_LO, The address [31:0] of minimum boundary of user-defined RMA. 
24:9, ETHER_TYPE, Ether type value 
8 BYPASS_VLAN User-defined RMA bypass VLAN drop ("VLAN ingress filter", "VLAN error", "VLAN accept frame type" and "CFI = 1"). This works only if EN is set as enabled. 0b0: Follow VLAN decision 0b1: Bypass (works only if the ACT field is trapping to CPU or forward) 
7, LRN, 0b0: Not learn the SAMC 0b1: Learn the SMAC 
6, BYPASS_STP, User-defined RMA bypass "Blocking" and "Learning" port state of spanning tree. This works only if EN is set as enabled. 0b0: Drop by STP 0b1: Bypass (works only if the ACT field is trapping to CPU)
5:3, ACT, User-defined RMA action. 0x0: Forward 0x1: Drop 0x2: Trap to local CPU 0x3: Trap to master CPU 0x4: Flood (limited by RMA_USR_DEF_FLD_PMSK) 0x5 0x7: Reserved 
2:1, TYPE, Compare field. 0b00: Compare ADDR only 0b01: Compare EtherType only 0b10: Both ADDR and EtherType 0b11: Reserved 
0, EN, Enable this User-defined RMA rule. 0b0: Disable 0b1: Enable 


Vlan MacAddress Type Ports
1, 0100.0001.0001, STATIC, g0/1 g0/2
10, 0100.0001.0001, STATIC, g0/1 g0/2


Table 3-2 L2 Unicast Table Entry 
Fiels, Description 
VALID, This entry is valid or not: 0:invalid 1:valid 

FMT, Indicates entry format 0b00: L2 or port extension CB entry 0b01: OpenFlow 

ENTRY_TYPE, Entry type 0b0: L2 unicast/multicast entry 0b1: PE forwarding entry 

FID_RVID, FID/RVID(known as forwarding VID) 
MAC, MAC address 
L2_TNL, L2 tunnel indicator 
NEXT_HOP, Next Hop (flag indicating can be used for routing) 
IS_TRK, trunk or physical port 0b0: physical port 0b1: trunk 
SPA if IS_TRK=0 SPA<9:6> is unit, SPA<5:0> is physical port if IS_TRK=1, SPA<5:0> is trunk id(0-63) 
AGE, Aging Time Range from 0 to 7 
SA_BLK, Source MAC address blocking 
DA_BLK, Destination MAC address blocking 
STATIC, Static entry 
SUSPEND, Suspend (this entry waits to be removed/modified) 
TAGSTS, VLAN tag status (used by VLAN aggregation) 
AGG_PRI, Aggregating priority (used by VLAN aggregation) 
AGG_VID, Aggregating VLAN ID (used by VLAN aggregation)


Should we complete RTL9310 developer guide in the next week? But if we study deeply then more time spended for each topic.

Should we aim to complete the RTL9310 Developer Guide by next week? Based on our previous experience, if we studying each topic in depth maybe we could not complete in time.


###################################################################################

4 L3 Feature
4.1.1 Routing overview
4.1 Routing  
4.1.1
Routing Overview 
L3 is the Network Layer in OSI (Open Systems Interconnection) model. The network switching device 
routes and forwards packets between different networks (VLANs) as a router. It’s a common approach 
to make hosts in different networks able to communicate with each other. 
Routing is the process of forwarding packets hop by hop on L3. It primarily includes finding an outgoing 
interface and a next hop, modifying the packets’ SMAC, DMAC, VID (if provided), TTL and L3 header 
checksum (for IPv4) and forwarding. 
The device supports IPv4/IPv6 unicast and multicast routing. The system provides global options for 
each type of packets to determine whether to route or discard those L3 packets. 
Figure 4-1 Classic Forwarding Model of L2 Bridging and L3 Routing 


.1.2
 Routing Overview 
L3 is the Network Layer in OSI (Open Systems Interconnection) model. The network switching device 
routes and forwards packets between different networks (VLANs) as a router. It’s a common approach 
to make hosts in different networks able to communicate with each other. 
Routing is the process of forwarding packets hop by hop on L3. It primarily includes finding an outgoing 
interface and a next hop, modifying the packets’ SMAC, DMAC, VID (if provided), TTL and L3 header 
checksum (for IPv4) and forwarding. 
The device supports IPv4/IPv6 unicast and multicast routing. The system provides global options for 
each type of packets to determine whether to route or discard those L3 packets. 
Figure 4-1 Classic Forwarding Model of L2 Bridging and L3 Routing 
VLAN X
 (L2 Bridging)
 HOST 1
 (MAC A)
 HOST 2
 (MAC B)
 L3 Interface 
L3
 Interface
 (MAC E)
 VRF N
 (L3 Routing)
 VLAN Y
 (L2 Bridging)
 
 (MAC F)
 HOST 3
 (MAC C)
 HOST 4
 (MAC D)



4.1.2 L3 Interface
An L3 interface is conceptually where packets enter and leave a network (VLAN). It’s used to route 
packets between VLANs, so it is also called VLAN interface. 
Normally, L3 routing is receiving a packet from an L3 interface and transmitting the packet to another 
L3 interface. 

4.1.2.1 Overview
The device supports 1024 L3 interfaces and up to 16 different MTU values used by all L3 interfaces.

4.1.2.2 Attribute 
There are bunch of attributes belong to an interface, the details are described as below.
(Physically, these attributes are stored into L3_IGR_INTF and L3_EGR_INTF separately.)






3.3 Trafic Isolation.

3.3.1
Traffic Isolation Overview 
Traffic isolation separates all the traffic (unicast, multicast, and broadcast) in layer 2 to provide traffic 
interference and bandwidth utilization. The system provides two type isolation modules, Port-based 
Isolation and VLAN-based Isolation. The details are as below.



3.3.2
Port-based Isolation 
When hosts are connected to router through switch ports, they can utilize Port Isolation to force the 
communication between hosts through the router. Port isolation is a mechanism to provide a CPU 
configurable destination port mask for an ingress port. For each ingress port, the device provides a 
port isolation port mask configuration defined in Figure 3-3. Bit value 1 of the port mask means the 
port can communicate with each other. Bit value 0 is that the port can’t forward any packet. No matter 
the received packets are known/unknown Unicast, known/unknown Multicast, Broadcast packets. All 
of the packets are limited by the configuration of Port Isolation port mask in forwarding. 
Table 3-54 PORT_ISO_CTRL Register 
Bits Field Description 
63:7 P_ISO_MBR_0 Port isolation configuration that each port can specify a port list to communicate with. 
For example as Figure 3-3, hosts A and B are connected to ports 0 and 3 separately while router is 
connected to port 7. The traffic between A and B must be forwarded by router. Therefore, user can 
configure the 7th bit of Port Isolation port mask of ports 0 and 3 to be 1 while other bits should be 
configured to 0, and Port Isolation port mask of port 7 still remains all ports. Then, direct 
communication between downlink ports, 0 and 3, is forbidden and only traffic between uplink port 
and downlink port can be forwarded. 
The device provides a control setting RESTRICT_ROUTE, which indicates whether the routed packet is 
restricted by Port-based port isolation. 

Table 3-55 PORT_ISO_RESTRICT_ROUTE_CTRL Register 
Bits, Field, Description,
0:0, RESTRICT_ROUTE, Indicates whether the routed packet is restricted by Port-based port isolation. 0b0: not restricted 0b0: restricted


3.3.3
VLAN-based Isolation 
Sometimes, uplink ports communicate with uplink port and downlink ports, but downlink ports don’t 
be allowed to communicate with each other in VLAN domain. Customer can use VLAN-based Isolation 
to achieve the application. The device provides 32 sets VLAN-based Isolation configurations in Table 

3-56.  Table 3-56 PORT_ISO_VB_ISO_PMSK_CTRL Register 
Bits,Field, Description, 
88:32, VB_ISO_MBR, VLAN-based isolated port mask (port 56 to 0). Ports which have corresponding port mask clear(i.e. 0) in this mask could not communicate with each other, while ports which have corresponding port mask set could communicate with any other ports. 
24:13, VID_UPPER, Upper bound VLAN ID,
12:1, VID_LOWER, Upper bound VLAN ID,
0:0, VALID, Indicates whether this entry is valid 0: entry invalid; 1: entry valid. 

For every packet, system will check all the 32 sets if matched or not. The matched condition is: the 
VALID field is valid and forwarding VID in the range [VID_UPPER, VID_LOWER]. If matched (multiple hit, 
select the lowest index set), the field VB_ISO_MBR will be referred.  
The ports with bit value 0 in VB_ISO_MBR can’t communicate with other ports having same bit value 0. 
However, they can communicate with the other port with bit value 1. If bit value of ports is set to 1, 
the port can communicate to all ports in the VLAN. Sometimes, bit value of uplink port is set to 1 and 
bit value of downlink port is set to 0 in network environment. For example as Figure 3-4, ports 0 and 1 
are uplink ports and other ports are downlink ports. Only configure (VB_ISO_MBR) of VLAN-based 
Isolation entry to be (0x3), then uplink ports can communicate to all ports, but downlink ports only 
communicate to uplink port.
Unlike the port-based isolation, the VLAN based isolation always has no effect on the routing packet, 
including unicast routing and multicast routing packet. But both the unicast and multicast bridged 
packet are still restricted.