# blush-response
![blush-response](https://github.com/user-attachments/assets/3cc2ae50-9032-4008-b160-0cf264ab19e3)


## Collection of inference performance tests for Telco LLMs

These tests are designed to evaluate the inference performance (accuracy, relevance, completeness, and potentially speed) of Large Language Models (LLMs) in the context of analyzing and troubleshooting ISP provider networks. The focus is on core provider protocols like BGP, MPLS (LDP/RSVP-TE), SRv6, and underlying IGP (OSPF/ISIS). Examples utilize Nokia SR OS command syntax and log formats where appropriate.

**Test ID: ISP_LLM_001**
*   **Objective:** Diagnose BGP neighbor state issues from command output.
*   **Category:** Routing Problems
*   **Protocols:** BGP
*   **Input Data:**
    ```
    show router bgp summary
    ===============================================================================
    BGP Summary
    ===============================================================================
    Legend: P-# Prepend Summary
    AS            : 65001      Router ID      : 192.168.1.1
    VRF ID        : N/A        VRF Name       : N/A
    Admin Status  : Up         Oper Status    : Up
    Hold Time     : 90         Min Route Adver: 30         Route Reflector: Disabled
    Confed AS     : 0
    Cluster ID    : 0.0.0.0
    Local AS Loops: 1          Default Orig   : Disabled   Send Def Route : Disabled

    Restart Time  : 120        Stale Route Tim: 360        Peer Tracking  : Disabled
    Graceful Rstrt: Enabled    Helper Support : Enabled
    Rapid Update  : Disabled   Rapid Fallover : Disabled   Log Neighbor Ch: Enabled
    ECMP          : 2          Default MED    : N/A        Compare MED All: Disabled
    Compare RtrID : Disabled   Ignore Nexthop : Disabled   Ignore AS Path : Disabled
    Private AS Cnt: Disabled   Remove Prvt All: Disabled   Remove Prvt Rep: Disabled
    Aggregator ID : 192.168.1.1 Aggr AS        : 65001      Use Aggr ID AS : Disabled
    Best Path Sel : ---
      Always Comp MED: Disabled   Path Selection : Enabled    Add Path       : Disabled
      Ignore NH MED  : Disabled   AS Path MP Multipath: Disabled Add Path Max Path: 0
      BGP Adv Passive: Disabled
      Local Pref     : 100        Dampening      : Disabled   Route Policies : Disabled

    -------------------------------------------------------------------------------
    Neighbor Summary (all)
    -------------------------------------------------------------------------------
    Neighbor        AS    Admin Oper State   Flaps    In Pfx   Act Pfx Out Pfx Up/Down
    -------------------------------------------------------------------------------
    192.168.10.1  65002   Up    Idle         5        0        0       0       0d00h05m10s
    192.168.10.5  65003   Up    Established  0        150000   150000  10      10d05h15m20s
    192.168.10.9  65004   Down  Down         1        0        0       0       0d00h00m00s
    ===============================================================================
    ```
*   **Prompt:** "Analyze the provided BGP summary output from a Nokia SR OS router. Identify any BGP neighbors with potential issues, describe the likely state or problem for each, and suggest initial troubleshooting steps."
*   **Expected Output:** Identifies neighbor 192.168.10.1 is in 'Idle' state despite being Administratively 'Up', indicating a connection or configuration issue (likely TCP connection failure, incorrect neighbor IP/AS, firewall). Suggests checking connectivity (ping), neighbor configuration on both sides, and firewall rules. Identifies neighbor 192.168.10.9 is Administratively 'Down', meaning it's configured but shut down locally. Suggests checking the local configuration (`config>router>bgp>neighbor>shutdown`). Mentions neighbor 192.168.10.5 is operational.
*   **Metrics:** Accuracy, Completeness, Clarity, Actionability.

---

**Test ID: ISP_LLM_002**
*   **Objective:** Interpret BGP route instability based on logs.
*   **Category:** Routing Problems, Log Analysis
*   **Protocols:** BGP
*   **Input Data:**
    ```
    tmnxSyslog: 1 2023/10/27 10:15:01.000 UTC Minor BGP #2004 Base Peer 198.51.100.2
    "Peer 198.51.100.2 (AS 65100) Up"

    tmnxSyslog: 1 2023/10/27 10:15:35.000 UTC Minor BGP #2005 Base Peer 198.51.100.2
    "Peer 198.51.100.2 (AS 65100) Down: BGP Notification received, code: Hold Timer Expired (4), subcode: 0"

    tmnxSyslog: 1 2023/10/27 10:16:01.000 UTC Minor BGP #2004 Base Peer 198.51.100.2
    "Peer 198.51.100.2 (AS 65100) Up"

    tmnxSyslog: 1 2023/10/27 10:16:38.000 UTC Minor BGP #2005 Base Peer 198.51.100.2
    "Peer 198.51.100.2 (AS 65100) Down: BGP Notification received, code: Hold Timer Expired (4), subcode: 0"

    tmnxSyslog: 1 2023/10/27 10:17:05.000 UTC Minor BGP #2004 Base Peer 198.51.100.2
    "Peer 198.51.100.2 (AS 65100) Up"
    ```
*   **Prompt:** "Based on these Nokia SR OS syslog messages, describe the state of the BGP peering session with 198.51.100.2. What is the likely cause of the repeated 'Down' events, and what should be investigated?"
*   **Expected Output:** Describes the BGP session as flapping (repeatedly going up and down). Identifies the cause of failure as 'Hold Timer Expired'. Suggests investigating potential causes like high latency, packet loss on the path between peers, high CPU on either peer preventing keepalives processing, MTU mismatch issues, or mismatched hold timers (though less likely with explicit message). Recommends checking interface statistics, path latency/loss (ping/traceroute), and CPU utilization on both BGP peers.
*   **Metrics:** Accuracy, Relevance, Clarity, Actionability.

---

**Test ID: ISP_LLM_003**
*   **Objective:** Analyze potential MPLS LSP path issues.
*   **Category:** Routing Problems
*   **Protocols:** MPLS (RSVP-TE), IGP (Implicit)
*   **Input Data:**
    ```
    show router mpls lsp "to_PE3" path detail
    ===============================================================================
    MPLS LSP to_PE3 Path Detail
    ===============================================================================
    LSP Name        : to_PE3                     Path Name      : primary
    Path Oper State : Down                       Retry Timer    : 30 sec
    Path Admin State: Up                         Hop Limit      : 32
    To              : 192.168.1.3                From           : 192.168.1.1
    Primary         : primary
    Working Path    : primary (1)
    -------------------------------------------------------------------------------
    Path Information : primary (1)
    -------------------------------------------------------------------------------
    Received RRO : None
    Last Error   : No path found for LSP to_PE3, path primary

    Actual Path  : None
    Computed CSPF Path:
    Hop Address                             ERO Hop Type   MTU   LSP/Lnk/OutLbl M/B/I/L
    -------------------------------------------------------------------------------
    No path found
    ===============================================================================

    show router ospf database router 192.168.1.3 extensive
    (Output showing 192.168.1.3 is present in the OSPF database, but maybe with limited links or TE metric issues)
    ```
*   **Prompt:** "An MPLS LSP named 'to_PE3' destined for 192.168.1.3 is operationally down. The LSP path detail shows 'No path found'. Assuming the destination PE (192.168.1.3) is present in the IGP (OSPF) database, what are the most likely reasons for the CSPF failure, and what should be checked on this Nokia SR OS router?"
*   **Expected Output:** Explains that CSPF failure means the router couldn't compute a path satisfying the LSP's constraints using the TE database derived from the IGP. Likely reasons include:
    1.  Lack of TE-enabled links along any potential path towards 192.168.1.3.
    2.  Insufficient available bandwidth on required links (if bandwidth constraint is configured on LSP).
    3.  Admin-group (color) constraints on the LSP that cannot be met by available links.
    4.  Explicit Route Object (ERO) constraints that specify unreachable or non-TE hops.
    5.  IGP reachability issue *despite* the router LSA being present (e.g., broken ABR connection, partitioning).
    Suggests checking: LSP configuration constraints (`config>router>mpls>lsp>...`), TE parameters on interfaces (`show router ospf TEdatabase`, `show router interface detail`), IGP topology and TE link state (`show router ospf database extensive`), and potentially running `tools perform router mpls test-cspf lsp "to_PE3" path "primary"`.
*   **Metrics:** Accuracy, Completeness, Actionability.

---

**Test ID: ISP_LLM_004**
*   **Objective:** Correlate link errors with potential service impact.
*   **Category:** Link Failures/Errors
*   **Protocols:** Ethernet, (Implicit: Services riding over the link)
*   **Input Data:**
    ```
    show port 1/1/1 detail
    ===============================================================================
    Ethernet Interface
    ===============================================================================
    Description        : Link_to_Core_Router_2
    Interface          : 1/1/1                  Oper Speed       : 10 Gbps
    Admin State        : up                     Oper Duplex      : full
    Oper State         : up                     Link State       : Up
    Phys Address       : 00:11:22:aa:bb:cc      Configured Mode  : network
    [...]
    Input packets      : 1500000000             Output packets     : 1200000000
    Input bytes        : 1800000000000          Output bytes       : 1500000000000
    Input errors       : 5                      Output errors      : 0
      Runts            : 0                        Collisions       : 0
      Giants           : 0                        Late Collisions  : 0
      CRC/Align errors : 5                        Excessive Collis : 0
      FIFO Overruns    : 0
      Input Discards   : 10                       Output Discards    : 2
        In Discard Other : 10                       Out Discard Other: 2
    [...]
    ===============================================================================
    (Assume logs show intermittent BGP or IGP flaps involving neighbor reachable over this link)
    ```
*   **Prompt:** "Interface 1/1/1 on this Nokia SR OS router shows 5 CRC/Align errors and 10 Input Discards recently. Although the counts are low, assume there have also been intermittent protocol flaps (like BGP or OSPF) with the neighbor connected via this link. Correlate the interface errors with the potential protocol instability. What could be the underlying cause, and what is the recommended action?"
*   **Expected Output:** Explains that CRC/Align errors indicate physical layer problems (Layer 1). Even low counts can corrupt packets, potentially including critical protocol keepalives or updates. This corruption can lead to protocol packets being dropped (contributing to Input Discards if malformed) or timers expiring (like BGP hold timer or OSPF dead timer), causing the observed flaps. The underlying cause is likely a faulty optic (SFP/QSFP) on either end, a damaged fiber patch cable, a dirty connector, or potentially issues with the physical port itself. Recommends checking/replacing optics and patch cords, cleaning fiber connectors, and if the issue persists, potentially moving the connection to a different port. Monitoring the error counters after action is crucial.
*   **Metrics:** Accuracy, Correlation ability, Actionability.

---

**Test ID: ISP_LLM_005**
*   **Objective:** Diagnose LDP session issues.
*   **Category:** Routing Problems
*   **Protocols:** MPLS (LDP), IGP (Implicit)
*   **Input Data:**
    ```
    show router ldp session
    ===============================================================================
    LDP Sessions
    ===============================================================================
    Peer LDP Id      State        Adv Labels     Adj Type Msg Sent    Msg Recv
                                  Sent   Recv               Sent Recv Up Time
    -------------------------------------------------------------------------------
    192.168.1.2:0    Operational  9500   9800     Link     50000 50000 25d10h
    192.168.1.4:0    Nonexistent  0      0        --       0     0     --
    ===============================================================================

    show router ldp discovery
    ===============================================================================
    LDP Discovery Sources
    ===============================================================================
    Interface Name                     Transport Addr Type       Active Sessions
      Adj Type                         LDP Id         Hold Time  Remaining Time
    -------------------------------------------------------------------------------
    intf_to_R2                         192.168.10.2   Interface  Yes
      Link                             192.168.1.2:0  45         42
    intf_to_R4                         192.168.10.4   Interface  No
      Link                             192.168.1.4:0  45         N/A
    ===============================================================================
    (Assume ping 192.168.10.4 source 192.168.10.1 is successful)
    ```
*   **Prompt:** "Analyze the LDP session and discovery output from this Nokia SR OS router. The LDP session to peer 192.168.1.4:0 is 'Nonexistent', but LDP discovery shows the peer is being heard via interface 'intf_to_R4'. Basic IP connectivity to the interface IP (192.168.10.4) is confirmed. What are the likely reasons the LDP TCP session failed to establish?"
*   **Expected Output:** Explains that LDP uses Hello messages (UDP) for discovery on the link and then attempts to establish a TCP session between the LDP identifiers (typically router IDs, using the transport addresses). Since discovery is working but the session isn't established, the issue lies with the TCP connection on port 646. Potential reasons:
    1.  Mismatched LDP transport addresses (router IDs) configured or used for session establishment. Router might be trying to connect to 192.168.1.4, but the remote router expects connection from a different source IP.
    2.  Firewall/ACL blocking TCP port 646 between the LDP transport addresses (192.168.1.1 and 192.168.1.4).
    3.  LDP is not enabled or configured correctly on the remote router (192.168.1.4) for session establishment, or specifically disabled towards this peer.
    4.  Underlying routing issue for the LDP transport addresses (e.g., 192.168.1.1 cannot reach 192.168.1.4 via routing table, even if link IPs are pingable).
    Suggests verifying LDP configuration on both routers (transport address, interface enablement), checking ACLs/firewalls, and confirming routing table reachability for the LDP identifiers.
*   **Metrics:** Accuracy, Completeness, Clarity, Actionability.

---

**Test ID: ISP_LLM_006**
*   **Objective:** Troubleshoot SRv6 Segment Routing path reachability.
*   **Category:** Routing Problems
*   **Protocols:** SRv6, IS-IS/OSPF (Implicit), BGP (Implicit for inter-domain)
*   **Input Data:**
    ```
    Scenario Description: An SRv6 TE policy is configured to steer traffic towards PE5 via a specific mid-point router R3.
    The policy uses a Segment List (SID List): <SID for R3 (End.X)>, <SID for PE5 (End.DT4)>

    show router segment-routing srv6-policy "Policy_to_PE5_via_R3" detail
    ===============================================================================
    SRv6 Policy "Policy_to_PE5_via_R3" Detail
    ===============================================================================
    Name            : Policy_to_PE5_via_R3       Endpoint IP    : 2001:db8:5::1
    Admin State     : Up                         Oper State     : Down
    Binding SID     : cafe:5::                     Color          : 100
    Preference      : 100
    Segment List    : SL_PE5_via_R3              Oper SL Index  : None
    Last Error      : Segment List SL_PE5_via_R3 invalid: Next SID unreachable
    Next SID Addr   : 2001:db8:3::ad:0               Next SID Type  : End.X
    ===============================================================================

    show router isis srv6-locator "DEFAULT" node 2001:db8:3:: detail
    (Output confirms the locator 2001:db8:3::/64 exists for node R3, but perhaps the specific End.X function SID 2001:db8:3::ad:0 is not advertised or reachable in the RIB)

    show router route-table ipv6 2001:db8:3::ad:0 /128 detail
    (Output shows no route or perhaps a less specific route)
    ```
*   **Prompt:** "An SRv6 TE Policy 'Policy_to_PE5_via_R3' is operationally down on this Nokia SR OS router. The error indicates the Segment List is invalid because the 'Next SID unreachable', specifically identifying SID 2001:db8:3::ad:0 (an End.X SID for router R3). Assuming R3's locator (2001:db8:3::/64) is present in the IS-IS SRv6 database, what are the likely reasons this specific SID is unreachable and how should it be investigated?"
*   **Expected Output:** Explains that SRv6 reachability depends on routing to the SIDs in the segment list. Even if R3's main locator is known via IS-IS, the specific End.X SID (2001:db8:3::ad:0) might be unreachable due to:
    1.  The SID 2001:db8:3::ad:0 is not actually advertised by R3 in IS-IS (configuration error on R3).
    2.  There is no valid entry in the IPv6 routing table (specifically `route-table ipv6`) on the current router for the exact SID 2001:db8:3::ad:0/128. Reachability might only exist for the /64 locator, but not the specific function SID if it requires a more specific path or configuration.
    3.  An underlying IPv6 routing problem prevents reachability to R3's loopback or interface associated with the SID, even if the locator prefix is advertised.
    4.  Filtering or policy is preventing the SID advertisement from being accepted or installed in the RIB.
    Suggests verifying the SRv6 SID advertisement configuration on R3 (`config>router>isis>segment-routing>srv6>locator>...`), checking the IS-IS SRv6 SID database on the local router (`show router isis database srv6-sid ...`), examining the IPv6 route table for the specific SID (`show router route-table ipv6 2001:db8:3::ad:0 /128 detail`), and checking for any relevant route policies or filters.
*   **Metrics:** Accuracy, SRv6 Knowledge, Actionability.

---

**Test ID: ISP_LLM_007**
*   **Objective:** Identify the cause of a network outage from link failure logs.
*   **Category:** Link Failures/Errors, Log Analysis
*   **Protocols:** Ethernet, (Affected: All protocols over the link)
*   **Input Data:**
    ```
    tmnxSyslog: 1 2023/10/27 11:30:05.000 UTC Major ETH_CFM #2001 Base MA:100 MEP:10
    "Remote MEP 20 for MEGID 'LinkCheck-A-B', MAID 'MA100' is Down, Interface Port 1/1/5"

    tmnxSyslog: 1 2023/10/27 11:30:06.000 UTC Critical PORT #2004 Base Port 1/1/5
    "Port 1/1/5 Link Down - Loss of Signal"

    tmnxSyslog: 1 2023/10/27 11:30:07.000 UTC Minor OSPF #2001 Base Interface intf-to-B
    "OSPF Interface intf-to-B (1/1/5) state change to Down, reason: Link Down"

    tmnxSyslog: 1 2023/10/27 11:30:07.000 UTC Minor LDP #2001 Base Interface intf-to-B
    "LDP Interface intf-to-B (1/1/5) Down"

    tmnxSyslog: 1 2023/10/27 11:30:10.000 UTC Minor BGP #2005 Base Peer 192.168.20.2
    "Peer 192.168.20.2 (AS 65002) Down: Interface flap or address removed"
    ```
*   **Prompt:** "Analyze this sequence of Nokia SR OS syslog messages. What is the root cause of the BGP peer 192.168.20.2 going down, and what sequence of events led to it?"
*   **Expected Output:** Identifies the root cause as a physical link failure on Port 1/1/5, indicated by the "Port 1/1/5 Link Down - Loss of Signal" message. Explains the sequence:
    1.  The physical link failed (Loss of Signal).
    2.  Ethernet CFM detected the failure almost immediately ("Remote MEP ... Down").
    3.  The port state change triggered the IGP (OSPF) interface to go down.
    4.  The LDP interface associated with the port also went down.
    5.  The BGP session with peer 192.168.20.2, which likely used this link for connectivity (either directly or via IGP learned route), failed because its underlying reachability was lost.
*   **Metrics:** Accuracy, Correlation ability, Clarity.

---

**Test ID: ISP_LLM_008**
*   **Objective:** Analyze impact of interface flapping.
*   **Category:** Link Failures/Errors
*   **Protocols:** Ethernet, IGP, BGP, MPLS
*   **Input Data:**
    ```
    show port 1/1/2 | match "Link State|Oper State|Flaps"
    Oper State         : up                     Link State       : Up/Flapping(5)

    (Combined with intermittent log messages similar to Test ID ISP_LLM_007, but the link comes back up quickly each time)
    ```
*   **Prompt:** "Port 1/1/2 on a Nokia SR OS router is reported as 'Up/Flapping(5)', indicating 5 recent link state changes. Describe the likely impact of this flapping on routing protocols (like OSPF/ISIS, BGP) and MPLS LSPs potentially using this link. Why is flapping often more disruptive than a hard down failure?"
*   **Expected Output:** Explains that interface flapping causes repeated link down/up events. Each down event triggers:
    1.  IGP (OSPF/ISIS) adjacency tear-down, flooding of topology updates (LSAs/LSPs) indicating link removal, and SPF recalculations across the area/level.
    2.  LDP/RSVP sessions over the link to tear down.
    3.  BGP sessions using the link directly or indirectly to potentially drop (if hold timers expire during down period or due to next-hop invalidation).
    4.  MPLS LSPs using the link to fail and potentially attempt rerouting (if FRR or secondary paths configured).
    Each subsequent up event triggers the reverse: IGP adjacency forms, new updates flood, SPF runs again, LDP/RSVP/BGP attempt to re-establish, LSPs may revert.
    Flapping is often worse than a hard down because it causes continuous network instability: constant recalculations consume router CPU, topology churn prevents convergence, traffic may blackhole briefly during transitions, and protocols may enter dampening states (e.g., BGP flap dampening) further delaying restoration. A hard down allows the network to converge to a stable alternate state.
*   **Metrics:** Accuracy, Depth of Explanation, Clarity.

---

**Test ID: ISP_LLM_009**
*   **Objective:** Interpret BGP Notification message details from logs.
*   **Category:** Routing Problems, Log Analysis
*   **Protocols:** BGP
*   **Input Data:**
    ```
    tmnxSyslog: 1 2023/10/27 12:05:15.000 UTC Minor BGP #2005 Base Peer 203.0.113.5
    "Peer 203.0.113.5 (AS 65200) Down: BGP Notification received, code: Update Message Error (3), subcode: Invalid Next Hop Attribute (8), data: [Relevant part of BGP update causing error, e.g., showing the faulty next-hop IP]"
    ```
*   **Prompt:** "A BGP session went down with peer 203.0.113.5. The Nokia SR OS log indicates a Notification was *received* with error code 3, subcode 8 (Invalid Next Hop Attribute). What does this specific error code/subcode combination imply about the cause of the session failure? Whose configuration is likely at fault?"
*   **Expected Output:** Explains that receiving a BGP Notification means the *remote* peer (203.0.113.5, AS 65200) detected an error in an UPDATE message sent *by the local router*. The specific error "Update Message Error / Invalid Next Hop Attribute" (Code 3, Subcode 8) means the local router sent a BGP update containing a next-hop IP address that the remote router considers invalid according to BGP rules (e.g., for eBGP, the next-hop must typically be the IP address of the sending router on the peering subnet, unless `next-hop-self` or specific policies are used). This implies the configuration fault likely lies on the *local* router, specifically in its route policies or BGP export configuration that determines the next-hop attribute for advertised routes.
*   **Metrics:** Accuracy, BGP Protocol Knowledge, Clarity.

---

**Test ID: ISP_LLM_010**
*   **Objective:** Predict impact of a planned link maintenance.
*   **Category:** Proactive Analysis
*   **Protocols:** All relevant (IGP, BGP, MPLS/SRv6)
*   **Input Data:**
    ```
    Scenario Description: Network engineers plan to perform maintenance on the fiber link connecting Core_Router_A (this router) Port 1/1/3 to Core_Router_C. This is a primary path for several critical MPLS LSPs and carries significant transit BGP traffic. Assume Core_Router_A and Core_Router_C are OSPF neighbors over this link, and there is an alternate path via Core_Router_B. Assume MPLS FRR (Fast Reroute) is configured for affected LSPs.

    show router ospf interface "intf_to_C" detail
    (Output showing high OSPF cost, e.g. cost 10)

    show router ospf interface "intf_to_B" detail
    (Output showing higher OSPF cost, e.g. cost 50)

    show router mpls lsp transit-services | match TunnelID | count
    (Output showing e.g., 50 transit LSPs)

    show router bgp neighbor 192.168.30.1 | match "Hold Time|Up/Down"
    (Output showing neighbor C's BGP session info)
    ```
*   **Prompt:** "Based on the scenario description and typical ISP network behavior, predict the sequence of events and impact when the link on Port 1/1/3 is administratively shut down for maintenance. Consider OSPF, MPLS LSPs (with FRR), and BGP sessions. Mention the role of the alternate path via Core_Router_B."
*   **Expected Output:** Predicts the following sequence and impact:
    1.  Admin shutdown of Port 1/1/3 causes immediate link down.
    2.  MPLS FRR (if protecting LSPs using this link) should activate within milliseconds (typically <50ms), rerouting traffic onto pre-calculated backup paths (likely via Router_B if it's the Loop-Free Alternate). Minimal traffic loss expected for protected LSPs.
    3.  OSPF adjacency over Port 1/1/3 will drop. Router_A will flood updated LSAs removing the link. Routers in the area will recalculate SPF. Traffic previously routed via the direct A-C link based on IGP cost will now use the higher-cost path via Router_B.
    4.  LDP/RSVP adjacencies over the link will drop.
    5.  BGP session(s) directly over the link (if any) will drop. BGP sessions relying on the IGP path via C might also reconverge if next-hops change, potentially causing brief BGP instability. Traffic will shift to alternate BGP paths.
    6.  Transit LSPs not using the link directly but relying on the IGP path over A-C might also trigger make-before-break or standard rerouting onto the new shortest path via B.
    7.  Overall impact: Brief reconvergence event. Protected traffic (MPLS FRR) sees minimal impact. Unprotected traffic follows IGP/BGP reconvergence, potentially seeing slightly longer outage/reroute time. Network stabilizes using path via Router_B. Latency might increase due to potentially longer/higher-cost path.
*   **Metrics:** Accuracy of Prediction, Completeness, Understanding of Inter-protocol Dependencies.

---

**Test ID: ISP_LLM_011**
*   **Objective:** Correlate events across multiple log snippets for root cause analysis.
*   **Category:** Log Analysis, Routing Problems
*   **Protocols:** OSPF, BGP
*   **Input Data:**
    ```
    Log Snippet 1 (Router A):
    tmnxSyslog: 1 2023/10/27 14:00:10.000 UTC Minor OSPF #2001 Base Interface intf-to-X
    "OSPF Interface intf-to-X (1/1/9) state change to Down, reason: Link Down"

    tmnxSyslog: 1 2023/10/27 14:00:15.000 UTC Minor BGP #2005 Base Peer 192.168.50.1
    "Peer 192.168.50.1 (AS 65050) Down: Next hop resolution failed"

    Log Snippet 2 (Router B - connected to Router A):
    tmnxSyslog: 1 2023/10/27 14:00:12.000 UTC Info OSPF #2004 Base Neighbor 192.168.0.1
    "OSPF Rtr 192.168.0.1 (Router A) neighbor state change to Down, Intf: intf-to-A"

    tmnxSyslog: 1 2023/10/27 14:00:18.000 UTC Info BGP #2006 Base Peer 192.168.0.1
    "Peer 192.168.0.1 (Router A) Path Rcvd: Withdrawn route for prefix 198.18.1.0/24"
    ```
*   **Prompt:** "Router A experiences an OSPF interface down event followed by a BGP peer down event ('Next hop resolution failed'). Router B logs show its OSPF neighbor (Router A) going down and a BGP route withdrawal from Router A around the same time. Correlate these events. What is the likely relationship between the OSPF failure on Router A and the BGP session failure also reported by Router A?"
*   **Expected Output:** Correlates the events showing a dependency. The OSPF interface down on Router A (intf-to-X) likely removed the route Router A was using to reach the next-hop for its BGP peer 192.168.50.1. The BGP process detected it could no longer resolve the necessary next-hop IP address in the routing table (learned via OSPF), causing it to declare the peer down due to 'Next hop resolution failed'. The events on Router B confirm the OSPF adjacency drop and subsequent BGP updates resulting from Router A's topology change. The root cause appears to be the failure on Router A's interface 1/1/9 ('intf-to-X').
*   **Metrics:** Correlation Accuracy, Understanding of IGP-BGP Interaction.

---

**Test ID: ISP_LLM_012**
*   **Objective:** Summarize a complex network incident from verbose logs.
*   **Category:** Log Analysis
*   **Protocols:** Mixed (e.g., Interface, OSPF, BGP, MPLS)
*   **Input Data:** A longer sequence of interleaved log messages (15-20 lines) showing:
    1.  Interface CRC errors increasing on Port X.
    2.  Interface X flapping (Down/Up).
    3.  OSPF adjacency flapping over Interface X.
    4.  MPLS LSP rerouting attempts (FRR activation, then maybe path error).
    5.  BGP neighbor flapping (Hold Timer Expired).
    6.  Interface X finally stays down (Loss of Signal).
    7.  OSPF converges stable state without Interface X.
    8.  MPLS LSP establishes successfully on alternate path.
    9.  BGP session re-establishes (if alternate path exists) or stays down.
*   **Prompt:** "Review the provided sequence of Nokia SR OS log messages detailing a network incident. Provide a concise summary (2-3 sentences) describing the root cause, the key events, and the final state."
*   **Expected Output:** A concise summary similar to: "An incident occurred triggered by physical layer issues (CRC errors, flapping) on Port X. This caused instability in OSPF, MPLS, and BGP protocols using the link. The issue culminated in a hard failure of Port X, after which the network reconverged using alternate paths, although some services might remain impacted depending on redundancy."
*   **Metrics:** Conciseness, Accuracy of Summary, Identification of Root Cause and Outcome.

---

**Test ID: ISP_LLM_013**
*   **Objective:** Analyze a BGP configuration snippet for potential issues.
*   **Category:** Routing Problems, Configuration Analysis
*   **Protocols:** BGP
*   **Input Data:**
    ```nokia-conf
    configure router bgp
        group "EBGP-Peer-AS65300"
            type external
            neighbor 192.0.2.1
                peer-as 65300
                description "Peering with AS65300"
                import "POLICY-IMPORT-AS65300"
                export "POLICY-EXPORT-AS65300"
                local-address 198.51.100.1 // Assumes this is our loopback, not interface IP
                local-as 65001 no-prepend-global-as
            exit
        exit
    exit

    configure policy-options
        policy-statement "POLICY-EXPORT-AS65300"
            entry 10
                from
                    protocol direct
                exit
                action accept
                exit
            exit
            entry 20
                from
                    prefix-list "CUSTOMER-PREFIXES"
                exit
                action accept
                    community add "65001:100"
                exit
            exit
            default-action reject
            exit
        exit
    exit
    ```
*   **Prompt:** "Review this Nokia SR OS BGP configuration snippet for the neighbor 192.0.2.1 in AS 65300. Identify potential configuration issues or areas for concern in a typical ISP eBGP peering context."
*   **Expected Output:** Identifies potential issues:
    1.  **`local-address`**: Using a loopback address (assumed 198.51.100.1) for an eBGP session usually requires `ebgp-multihop` to be configured (unless the peer is directly connected via the loopback's /32 route, which is rare). This session might fail to establish if multihop isn't set or if the peer doesn't have a route to the local-address.
    2.  **Export Policy**: Exporting `protocol direct` routes (entry 10) to an eBGP peer is generally discouraged and potentially dangerous, as it leaks infrastructure interface addresses.
    3.  **Default Action Reject**: While explicit rejection is good, ensure that *all* intended prefixes are covered by entries 10 and 20 or other entries. A missing prefix-list entry could block legitimate advertisements.
    4.  **Missing `ebgp-multihop`**: As noted in point 1, this is likely needed if the `local-address` is not on the directly connected interface IP used to reach the peer.
    5.  **(Minor)** No explicit MED setting or import policy manipulation shown, which might be needed depending on peering agreements.
*   **Metrics:** Accuracy of Analysis, Relevance to ISP practices, Clarity.

---

**Test ID: ISP_LLM_014**
*   **Objective:** Explain sub-optimal BGP path selection based on attributes.
*   **Category:** Routing Problems
*   **Protocols:** BGP
*   **Input Data:**
    ```
    show router bgp routes 203.0.113.0/24 extensive
    ===============================================================================
    BGP Route Entries for 203.0.113.0/24
    ===============================================================================
    -------------------------------------------------------------------------------
    Entry 1 (Path ID 1) - Used, Best
    -------------------------------------------------------------------------------
    From : 192.168.10.5 (192.168.1.5)        Peer AS : 65003
    Received : 2023/10/26 08:00:00           Protocol : ebgp
    Origin : igp                           Local Pref : 100
    AS Path : 65003 65500 I                  Next Hop : 192.168.10.5
    MED : 100                               Communities : 65003:100 65500:50
    AIGP : N/A                              Add Path RX : No Add Path TX : N/A

    -------------------------------------------------------------------------------
    Entry 2 (Path ID 2) - Not Used (Higher IGP cost for next-hop)
    -------------------------------------------------------------------------------
    From : 192.168.20.5 (192.168.1.6)        Peer AS : 65004
    Received : 2023/10/26 08:05:00           Protocol : ebgp
    Origin : igp                           Local Pref : 100
    AS Path : 65004 65500 I                  Next Hop : 192.168.20.5
    MED : 50                                Communities : 65004:200 65500:50
    AIGP : N/A                              Add Path RX : No Add Path TX : N/A
    Reason Not Best: Higher IGP cost for next-hop resolution
    ===============================================================================
    (Assume IGP cost to 192.168.10.5 is 10, IGP cost to 192.168.20.5 is 50)
    ```
*   **Prompt:** "Analyze the BGP routes received for prefix 203.0.113.0/24 on this Nokia SR OS router. Explain why the path received from peer 192.168.10.5 (AS 65003) is selected as best, even though the path from 192.168.20.5 (AS 65004) has a lower MED (50 vs 100)."
*   **Expected Output:** Explains the standard BGP best path selection process. It should state that while MED is considered, it comes *after* attributes like Local Preference (which is equal here at 100) and AS Path Length (which is also equal here, length 2). Crucially, BGP prefers paths with a reachable next-hop, and among those, it prefers eBGP over iBGP, and *then* considers the IGP cost to reach the next-hop *before* evaluating the MED (by default, unless `always-compare-med` is enabled). The output explicitly states the reason for non-selection of Path 2 is 'Higher IGP cost for next-hop resolution'. Therefore, even though Path 2 has a better MED, it's discarded earlier in the process because the cost to reach its next-hop (192.168.20.5) via the IGP is higher than the cost to reach the next-hop (192.168.10.5) for Path 1.
*   **Metrics:** Accuracy of BGP Path Selection Explanation, Clarity.

---

**Test ID: ISP_LLM_015**
*   **Objective:** Interpret the effect of a BGP community in a route policy.
*   **Category:** Routing Problems, Configuration Analysis
*   **Protocols:** BGP
*   **Input Data:**
    ```nokia-conf
    configure policy-options
        community "NO-EXPORT-SUBCONFEDS" members "no-export-subconfed"
        policy-statement "POLICY-IMPORT-TRANSIT"
            entry 10
                from
                    community "NO-EXPORT-SUBCONFEDS"
                exit
                action reject
                exit
            exit
            entry 20
                # ... other accept actions ...
            exit
            default-action accept
            exit
        exit
    exit

    configure router bgp group "IBGP-RR-Clients"
        import "POLICY-IMPORT-TRANSIT"
        # ... other group config ...
    exit
    ```
*   **Prompt:** "This Nokia SR OS configuration snippet shows part of an import policy 'POLICY-IMPORT-TRANSIT' applied to an iBGP group 'IBGP-RR-Clients'. Explain the purpose and effect of entry 10, which references the BGP community 'NO-EXPORT-SUBCONFEDS' (standard well-known community 'no-export-subconfed')."
*   **Expected Output:** Explains that the `no-export-subconfed` community (often represented numerically as `0xFFFFFF03`) is a well-known BGP community. When attached to a route, it signals that the route should not be advertised outside the local autonomous system (AS) *or* to any external BGP peers (eBGP), *and* crucially, it should also not be advertised to BGP peers belonging to other member ASes within the same BGP confederation. Entry 10 in the import policy explicitly `reject`s any route received from iBGP peers (in the 'IBGP-RR-Clients' group) that carries this community. This acts as an import filter, preventing the router from accepting or using routes that are tagged to not leave their originating AS or confederation sub-AS, enforcing the intended scope of the community.
*   **Metrics:** Accuracy of Community Explanation, Understanding Policy Logic.

---
