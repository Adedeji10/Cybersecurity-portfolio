Network Design, ACLs & Firewalls
*Project Overview*
Built a scalable, redundant enterprise network infrastructure with 4 VLANs, dual routers, dual switches, implementing network segmentation, access control policies, automated IP management, and high availability features with automatic failover.

Complexity: Intermediate to Advanced
Status: ✅ Completed

🛠️ Tools & Technologies
Simulation Tool: Cisco Packet Tracer 8.x
Devices: Cisco 2911 Routers (x2), Cisco 2960 Switches (x2)
Protocols: HSRP, LACP, NAT/PAT, DHCP, 802.1Q
Security: Extended ACLs, Reflexive ACLs, Stateful Filtering
🎯 Learning Objectives
By completing this project, I learned:

✅ VLAN segmentation and inter-VLAN routing (router-on-a-stick)
✅ Access Control Lists (ACLs) for policy enforcement
✅ NAT/PAT for Internet connectivity
✅ DHCP server configuration for automated IP assignment
✅ HSRP for gateway redundancy and automatic failover
✅ EtherChannel (LACP) for link aggregation
✅ Reflexive ACLs for stateful firewall protection
✅ Complex network troubleshooting (asymmetric routing)
🏗️ Network Architecture
Topology Diagram
                        [ISP-Router] (8.8.8.1)
                        /          \
                  (G0/0)            (G0/2)
                      /                \
                200.1.1.1          200.2.2.1
                    /                    \
            [Router-1]----------------[Router-2]
          (Active VLAN 10,20,30)   (Active VLAN 40)
                /   |                  |   \
          (G0/0) (G0/2)            (G0/0) (G0/2)
              /       \              /       \
        [Switch-1] ═══════════ [Switch-2]
         (Core)    EtherChannel   (Core)
            |      (F0/23-24)        |
            |                        |
      +-----+-----+            +-----+-----+
      |     |     |            |           |
   Admin   IT  Finance       Sales       
   VLAN10 VLAN20 VLAN30      VLAN40
   (2 PCs)(2 PCs)(2 PCs)     (2 PCs)

Physical Connectivity
Source-Device 	Interface   Destination-Device  	Interface  	Cable-Type
Router-1	      G0/1	      ISP-Router	          G0/0	      Copper Straight-Through
Router-2	      G0/1	      ISP-Router	          G0/2	      Copper Straight-Through
Router-1	      G0/2	      Router-2	            G0/2	      Copper Straight-Through
Router-1	      G0/0	      Switch-1	            G0/1	      Copper Straight-Through
Router-2	      G0/0	      Switch-1	            G0/2	      Copper Straight-Through
Router-2	      G0/0	      Switch-2	            G0/1	      Copper Straight-Through
Switch-1	      F0/23	      Switch-2	            F0/23	      Copper Cross-over
Switch-1	      F0/24	      Switch-2	            F0/24	      Copper Cross-over
ISP-Router	    G0/1	      Web Server	          Fa0	        Copper Straight-Through

🌐 IP Addressing Scheme
Internal VLANs (Private Networks)
VLAN-ID  	Department  	Network  	        Subnet-Mask  	 HSRP-VIP(Gateway)  	Router-1-IP  	  Router-2-IP  	 HSRP-Active
10	      Admin	        192.168.10.0/24	  255.255.255.0	 192.168.10.1	        192.168.10.2	  192.168.10.3	 Router-1
20	      IT	          192.168.20.0/24	  255.255.255.0	 192.168.20.1	        192.168.20.2	  192.168.20.3	 Router-1
30	      Finance	      192.168.30.0/24	  255.255.255.0	 192.168.30.1	        192.168.30.2	  192.168.30.3	 Router-1
40	      Sales	        192.168.40.0/24	  255.255.255.0	 192.168.40.1	        192.168.40.2	  192.168.40.3	 Router-2
Note: HSRP Virtual IP (.1) is what all PCs use as their default gateway.

WAN Links (Point-to-Point)
Link-Description	  Network	      Router-1	  Router-2	  ISP-Router
ISP to Router-1	    200.1.1.0/30	200.1.1.1	     -	      200.1.1.2
ISP to Router-2	    200.2.2.0/30	   -	      200.2.2.1	  200.2.2.2
Inter-Router Link	  10.0.0.0/30	  10.0.0.1	  10.0.0.2	     -

External Server
Device	    IP-Address  	 Gateway	    Purpose
Web Server	8.8.8.8	       8.8.8.1	    Simulates Internet resource
ISP-Router 	8.8.8.1	          -	        Gateway for external resources

🔧 Project Components
Component 1: VLAN Segmentation & Inter-VLAN Routing
Goal: Separate departments into isolated broadcast domains

Implementation:
Created 4 VLANs on both switches (10, 20, 30, 40)
Configured trunk links carrying all VLANs between switches and routers
Implemented router-on-a-stick (sub-interfaces) for inter-VLAN routing
Each VLAN has dedicated sub-interface on both routers

Why This Matters:
Network segmentation improves security
Reduces broadcast traffic
Simplifies network management
Allows granular access control

Component 2: Access Control Lists (ACLs)
Goal: Enforce security policies restricting access between departments

Security Policy:
Admin can access: IT, Finance, Sales, Internet
IT cannot access: Admin, Finance (but can respond to their requests)
Sales cannot access: Admin, Finance (but can respond to their requests)
Finance can access: Admin, IT, Internet
All departments can access Internet

Implementation:
Extended named ACLs on Router-1 and Router-2
Stateful ACLs using established keyword and ICMP echo-reply
Applied inbound on IT and Sales sub-interfaces
Uses permit/deny logic with implicit deny at end

ACL Logic Example (IT-SECURITY):
1. Permit TCP from IT to Admin if connection is established (response traffic)
2. Permit ICMP echo-reply from IT to Admin (ping responses)
3. Permit TCP from IT to Finance if established
4. Permit ICMP echo-reply from IT to Finance
5. Deny all other IT → Admin traffic
6. Deny all other IT → Finance traffic
7. Permit everything else (IT can access Sales and Internet)

Component 3: NAT/PAT & Internet Connectivity
Goal: Enable internal private networks to access the Internet using Network Address Translation

Implementation:
Configured NAT overload (PAT) on both routers
All internal IPs (192.168.x.x) translated to router's public IP
Router-1: Translates to 200.1.1.1
Router-2: Translates to 200.2.2.1
Access-list 1 defines which networks can be translated
Inside interfaces: All VLAN sub-interfaces
Outside interfaces: WAN interfaces (G0/1)

How It Works:
Internal PC (192.168.10.10) → Router → NAT Translation → 200.1.1.1:random_port → Internet
Internet Response → 200.1.1.1:random_port → Router → NAT Reverse Translation → 192.168.10.10

Component 4: DHCP (Dynamic Host Configuration Protocol)
Goal: Automatically assign IP addresses to all devices

Implementation:
DHCP pools created on both routers for all VLANs
Each pool provides: IP address, subnet mask, default gateway, DNS server
Excluded addresses: .1 (HSRP VIP), .2 (Router-1), .3 (Router-2)
Router-1 serves: VLANs 10, 20, 30
Router-2 serves: VLAN 40 (and backups for others)

DHCP Pool Configuration Example:
Pool Name: ADMIN-POOL
Network: 192.168.10.0/24
Default Gateway: 192.168.10.1 (HSRP Virtual IP)
DNS Server: 8.8.8.8
Excluded: 192.168.10.1 - 192.168.10.3

Component 5: HSRP (Hot Standby Router Protocol)
Goal: Provide gateway redundancy with automatic failover

Implementation:
HSRP groups configured on all 4 VLANs on both routers
Virtual IP (.1) shared between routers
Priority determines Active/Standby roles:
Router-1: Priority 110 for VLANs 10, 20, 30 (Active)
Router-2: Priority 110 for VLAN 40 (Active)
Standby router: Priority 100
Preempt enabled (Active router reclaims role after recovery)

How HSRP Works:
Both routers configured with same virtual IP
Active router "owns" the virtual MAC address
Standby router monitors Active via hello packets (every 3 seconds)
If Active fails (no hello for 10 seconds), Standby becomes Active
PCs never change their gateway - they always use the virtual IP
Failover time: ~3 seconds

Key Configuration:
VLAN 10 (Admin):
- Virtual IP: 192.168.10.1
- Router-1: Active (Priority 110)
- Router-2: Standby (Priority 100)

VLAN 40 (Sales):
- Virtual IP: 192.168.40.1
- Router-1: Standby (Priority 100)
- Router-2: Active (Priority 110)

Why Different Active Routers:
Load balancing: Distributes traffic across both routers
Efficient routing: Devices use physically closest router
Prevents asymmetric routing issues

Component 6: EtherChannel (Link Aggregation)
Goal: Increase bandwidth and provide link redundancy between switches

Implementation:
Bundled ports F0/23 and F0/24 on both switches
Protocol: LACP (Link Aggregation Control Protocol) - IEEE 802.3ad
Mode: Active (both sides actively negotiate)
Creates logical Port-Channel 1
Trunk mode carrying all VLANs (10, 20, 30, 40)

Benefits:
Bandwidth: 2 x 100 Mbps = 200 Mbps total
Redundancy: If one link fails, other continues
Load balancing: Traffic distributed across both links
No loops: STP sees EtherChannel as single link

Component 7: Reflexive ACLs (Stateful Firewall)
Goal: Protect internal network from unsolicited external connections

Implementation:
Outbound ACL: Permits and "reflects" all outgoing traffic
Inbound ACL: Only allows traffic matching reflected entries
Applied on WAN interfaces (Router G0/1)
Blocks external devices from initiating connections to internal network
Automatically permits return traffic for connections initiated internally

How It Works:
1. Internal PC initiates connection to Internet
2. Outbound ACL permits and creates temporary "reflection" entry
3. Internet responds
4. Inbound ACL checks reflection list
5. Match found → traffic permitted
6. No match → traffic denied (unsolicited)
7. Reflection entry times out after connection ends

Security Result:
Internal users can browse Internet
Responses automatically allowed
External attackers cannot probe internal network
Unsolicited inbound traffic blocked

🔒 Security Policies
Department Access Matrix
Source-Department	            Can Access	        Cannot Access	              Notes
Admin, IT, Finance, Sales     Internet	              -	                      Full access (most privileged)
IT	Sales                     Internet	          Admin, Finance (initiate)	  Can respond to Admin/Finance requests
Finance	Admin,                IT, Internet, Sales (initiate)	                Highly sensitive data
Sales	IT                      Internet	          Admin, Finance (initiate)	  External-facing, least privileged

Security Implementation Details
1. IT Department Restrictions (VLAN 20):
Applied: Router-1 and Router-2, G0/0.20 (inbound)
ACL: IT-SECURITY
Rules:
- Permit TCP IT→Admin if established (stateful)
- Permit ICMP echo-reply IT→Admin
- Permit TCP IT→Finance if established
- Permit ICMP echo-reply IT→Finance
- Deny IP IT→Admin (blocks new connections)
- Deny IP IT→Finance (blocks new connections)
- Permit IP any any (allows IT→Sales and Internet)

2. Sales Department Restrictions (VLAN 40):
Applied: Router-1 and Router-2, G0/0.40 (inbound)
ACL: SALES-SECURITY
Rules:
- Permit TCP Sales→Admin if established
- Permit ICMP echo-reply Sales→Admin
- Permit TCP Sales→Finance if established
- Permit ICMP echo-reply Sales→Finance
- Deny IP Sales→Admin
- Deny IP Sales→Finance
- Permit IP any any (allows Sales→IT and Internet)

3. External Firewall (Reflexive ACLs):
Applied: Router-1 and Router-2, G0/1 (WAN interface)
Outbound (G0/1 out):
- Permit ICMP from all VLANs, reflect to REFLECT-LIST
- Permit TCP from all VLANs, reflect to REFLECT-LIST
- Permit UDP from all VLANs, reflect to REFLECT-LIST
Inbound (G0/1 in):
- Evaluate REFLECT-LIST (check for matching outbound connection)
- Deny IP any 192.168.10.0/24
- Deny IP any 192.168.20.0/24
- Deny IP any 192.168.30.0/24
- Deny IP any 192.168.40.0/24
- Permit IP any any

⚙️ Configuration Summary
Router-1 Configuration Highlights:
Interfaces:
G0/1: WAN to ISP (200.1.1.1)
G0/2: Inter-router link (10.0.0.1)
G0/0: Trunk to Switch-1
Sub-interfaces: .10, .20, .30, .40 (all VLANs)
HSRP:
VLANs 10, 20, 30: Active (Priority 110)
VLAN 40: Standby (Priority 100)
Services:
DHCP: Pools for VLANs 10, 20, 30
NAT: Overload on G0/1 (200.1.1.1)
See full configuration: configs/router1-config.txt

Router-2 Configuration Highlights
Interfaces:
G0/1: WAN to ISP (200.2.2.1)
G0/2: Inter-router link (10.0.0.2)
G0/0: Trunk to Switch-1 and Switch-2
Sub-interfaces: .10, .20, .30, .40 (all VLANs)
HSRP:
VLANs 10, 20, 30: Standby (Priority 100)
VLAN 40: Active (Priority 110)
Services:
DHCP: Pools for all VLANs (backup and primary for VLAN 40)
NAT: Overload on G0/1 (200.2.2.1)
See full configuration: configs/router2-config.txt

Switch-1 Configuration Highlights:
VLANs:
VLAN 10 (Admin): Ports F0/1-2
VLAN 20 (IT): Ports F0/3-4
VLAN 30 (Finance): Ports F0/5-6
Trunks:
G0/1: To Router-1 (VLANs 10,20,30,40)
G0/2: To Router-2 (VLANs 10,20,30,40)
Port-Channel 1: To Switch-2 (F0/23-24, EtherChannel)
See full configuration: configs/switch1-config.txt

Switch-2 Configuration Highlights
VLANs:
VLAN 40 (Sales): Ports F0/1-2
Trunks:
G0/1: To Router-2 (VLANs 10,20,30,40)
G0/2: To Router-1 (VLANs 10,20,30,40) - Optional redundancy
Port-Channel 1: To Switch-1 (F0/23-24, EtherChannel)
See full configuration: configs/switch2-config.txt

Testing & Verification

Test 1: VLAN Connectivity
Objective: Verify devices within same VLAN can communicate
Test Cases:
Admin-PC1 → Admin-PC2 (same VLAN)
IT-PC1 → IT-PC2 (same VLAN)
Finance-PC1 → Finance-PC2 (same VLAN)
Sales-PC1 → Sales-PC2 (same VLAN)
Expected Result: All pings successful. Actual Result: ✅ Pass

Test 2: Inter-VLAN Routing
Objective: Verify router enables communication between VLANs
Test Cases:
Admin → IT (allowed by policy)
Admin → Finance (allowed)
Finance → Admin (allowed)
Finance → IT (allowed)
Expected Result: All pings successful. Actual Result: ✅ Pass

Test 3: ACL Policy Enforcement
Objective: Verify access control policies are enforced
Test Cases:
IT-PC1 → Admin-PC1 (blocked by ACL)
Admin-PC1 → IT-PC1 (allowed)
IT-PC1 receives reply from Admin (stateful ACL)
Sales-PC1 → Finance-PC1 (blocked)
Finance-PC1 → Sales-PC1 (allowed)
Sales-PC1 receives reply from Finance (stateful ACL)
Expected Result: Denied connections timeout, allowed connections succeed. Actual Result: ✅ Pass

Test 4: Internet Connectivity (NAT)
Objective: Verify all VLANs can reach Internet through NAT

Test Cases:
Admin-PC1 → 8.8.8.8 (web server)
IT-PC1 → 8.8.8.8
Finance-PC1 → 8.8.8.8
Sales-PC1 → 8.8.8.8
Command: ping 8.8.8.8 from each PC

Verification on Router:
Router# show ip nat translations
(Shows internal IPs translated to 200.1.1.1 or 200.2.2.1)
Expected Result: All departments reach Internet. Actual Result: ✅ Pass

Test 5: External Firewall (Reflexive ACLs)
Objective: Verify external devices cannot initiate connections to internal network
Test Cases:
Web Server → Admin-PC1 (blocked by reflexive ACL)
Web Server → IT-PC1 (blocked)
Web Server → Finance-PC1 (blocked)
Web Server → Sales-PC1 (blocked)
Expected Result: All connection attempts timeout. Actual Result: ✅ Pass

Test 6: DHCP Functionality
Objective: Verify PCs receive IP configuration automatically

Test Procedure:
Set PC to DHCP mode
Verify IP address received in correct range
Verify gateway is HSRP Virtual IP (.1)
Verify DNS server configured

Results:
Admin-PC1:
  IP: 192.168.10.10
  Gateway: 192.168.10.1 ✅
  DHCP Server: 192.168.10.2 (Router-1) ✅
Sales-PC1:
  IP: 192.168.40.10
  Gateway: 192.168.40.1 ✅
  DHCP Server: 192.168.40.3 (Router-2) ✅
Actual Result: ✅ Pass

Test 7: HSRP Failover
Objective: Verify automatic gateway failover when Active router fails

Test Procedure:
Verify Router-1 is Active for VLAN 10
   Router-1# show standby brief
   (Shows State: Active for VLAN 10)
Start continuous ping from Admin-PC1 to Internet
   Admin-PC1> ping 8.8.8.8 -t
Power off Router-1 (simulate failure)
Observe ping results
Verify Router-2 becomes Active
   Router-2# show standby brief
   (Shows State: Active for VLAN 10)
Power Router-1 back on
Verify Router-1 reclaims Active role (preempt)

Expected Results:
~3 seconds of ping timeout during failover
Connectivity restored automatically
No manual intervention needed
Actual Results: ✅ Pass

2-3 ping timeouts observed
Router-2 became Active in ~3 seconds
Traffic seamlessly failed over
Router-1 reclaimed Active role when recovered
Failover Time: ~3 seconds ✅

Test 8: EtherChannel Redundancy
Objective: Verify EtherChannel continues functioning if one link fails

Test Procedure:
Verify EtherChannel status
   Switch-1# show etherchannel summary
   (Shows Port-channel1: SU with Fa0/23(P) and Fa0/24(P))
Start ping from Switch-1 PC to Switch-2 PC
Physically disconnect one cable (F0/23)
Observe ping continues without interruption
Reconnect cable
Expected Result: No packet loss, seamless operation Actual Result: ✅ Pass

Ping continued without interruption
EtherChannel automatically adjusted
Load redistributed to remaining link

Test 9: Routing Table Verification
Objective: Verify proper routes exist on all routers

Commands:
Router# show ip route
Expected Routes on Router-1:
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
C    192.168.30.0/24 is directly connected, GigabitEthernet0/0.30
C    192.168.40.0/24 is directly connected, GigabitEthernet0/0.40
C    10.0.0.0/30 is directly connected, GigabitEthernet0/2
S*   0.0.0.0/0 [1/0] via 200.1.1.2 (default route to ISP)
Actual Result: ✅ Pass - All routes present

🐛 Troubleshooting
Major Issue Encountered: Sales VLAN Internet Connectivity Failure

Symptoms:
Sales PCs could ping their gateway (192.168.40.1) ✅
Sales PCs could ping Router-2 (192.168.40.3) ✅
Sales PCs could NOT ping ISP Router (200.2.2.2) ❌
Sales PCs could NOT ping Internet (8.8.8.8) ❌
Other VLANs worked perfectly ✅
NAT debugging on Router-2 showed no output (traffic not reaching NAT)

Troubleshooting Process
Step 1: Initial Hypothesis - NAT Configuration
Checked:
✅ VLAN 40 in access-list 1 (NAT permit list)
✅ G0/0.40 marked as ip nat inside
✅ G0/1 marked as ip nat outside
✅ NAT overload configured
Result: NAT configuration was correct. Not the issue.

Step 2: ACL Verification
Checked:
✅ OUTBOUND-FILTER included 192.168.40.0 permits
✅ SALES-SECURITY ACL ended with permit ip any any
✅ Temporarily removed ACLs - problem persisted
Result: ACLs were not blocking traffic. Not the issue.

Step 3: Layer 2 Verification
Checked:
✅ VLAN 40 created on Switch-2
✅ Trunk between Switch-2 and Router-2 carrying VLAN 40
✅ Sales PCs receiving DHCP from Router-2
Result: Layer 2 connectivity was working. Not the issue.

Step 4: DHCP Analysis - First Clue
Discovery:
Sales-PC1> ipconfig /all
DHCP Server: 192.168.40.2 (Router-1)
Initially confused - Sales on Switch-2, but getting DHCP from Router-1?
This hinted that traffic might be routing through Router-1

Step 5: Traceroute - The Smoking Gun 🎯
Sales-PC1> tracert 8.8.8.8
1  192.168.40.2 (Router-1)
   * * * timeout
Revelation: Traffic was going to Router-1, not Router-2!

Why this was happening:
Sales PCs physically on Switch-2 (connected to Router-2)
Gateway configured as 192.168.40.1 (HSRP Virtual IP)
Router-1 was Active for VLAN 40 (Priority 110)
Router-2 was Standby (Priority 100)
HSRP made Router-1 respond to ARP for 192.168.40.1
Traffic path: Sales PC → Switch-2 → EtherChannel → Switch-1 → Router-1

Root Cause Analysis
Problem: Asymmetric Routing
Traffic Flow (Broken):
Outbound:
Sales-PC (Switch-2) → EtherChannel → Switch-1 → Router-1 → ISP
Inbound (Expected):
ISP → Router-1 → Switch-1 → EtherChannel → Switch-2 → Sales-PC
Inbound (Actual):
ISP → Router-1 → [routing confusion] → ❌ DROPPED

Why it failed:
Router-1 performed NAT translation
Return traffic came back to Router-1
Router-1 tried to route back to 192.168.40.x
But Sales PC was on Switch-2, physically connected to Router-2
Return path through EtherChannel failed due to complex routing

The Solution
Changed HSRP priority for VLAN 40:

On Router-2:
interface gigabitEthernet 0/0.40
 standby 40 priority 110  ← Changed from 100 to 110

On Router-1:
interface gigabitEthernet 0/0.40
 standby 40 priority 100  ← Changed from 110 to 100
Result: Router-2 became Active for VLAN 40

Traffic Flow (Fixed)
Outbound:
Sales-PC (Switch-2) → Router-2 (directly connected) → ISP
Inbound:
ISP → Router-2 → Switch-2 → Sales-PC
✅ Symmetric routing!
✅ Direct path!
✅ No EtherChannel traversal needed!

Verification:
Sales-PC1> tracert 8.8.8.8
1  192.168.40.3 (Router-2)  ← Correct!
2  200.2.2.2 (ISP)
3  8.8.8.8 (Web Server)

Sales-PC1> ping 8.8.8.8
Reply from 8.8.8.8 ✅

Lessons Learned from This Issue
HSRP Active/Standby Role Selection Matters
Active router should match physical topology
Devices should use the geographically closest router. Prevents asymmetric routing issues

Traceroute is Essential for Troubleshooting. Shows actual path packets take
Reveals unexpected routing. More informative than ping alone

DHCP Server Identity is a Diagnostic Clue. If device gets DHCP from unexpected router
Indicates traffic is flowing through that router. Helps identify routing path

Load Balancing HSRP Design
Router-1 Active for Switch-1 VLANs
Router-2 Active for Switch-2 VLANs
Distributes load and prevents asymmetric routing
