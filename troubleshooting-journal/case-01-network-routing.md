# Case 01 — Network Connectivity Restoration After RAS Misconfiguration

**Category:** Networking / Remote Access Services  
**Difficulty:** Medium  
**Time to Resolve:** ~25 minutes  

---

## Situation

A Windows 11 client workstation (`CLIENT2`) had successfully joined the
`mydomain.com` domain and could communicate with the Domain Controller
on the internal network (`172.16.0.1` responded to pings). However,
the machine had no internet access — any attempt to reach an external
address timed out.

The Domain Controller itself had full internet access through NIC 1 (NAT).
The DHCP lease on the client was valid and showed the correct gateway (`172.16.0.1`).
Everything looked configured correctly on paper.

---

## Task

Identify why internal-to-external routing was failing on the client
and restore full internet connectivity while keeping the machine
secured behind the Domain Controller as the gateway.

---

## Action

**Step 1 — Confirm the client's network configuration**

Ran `ipconfig /all` on CLIENT2. Output showed:
- IP Address: `172.16.0.87` (within DHCP scope ✅)
- Default Gateway: `172.16.0.1` ✅
- DNS Server: `172.16.0.1` ✅

Network configuration on the client side was correct.

**Step 2 — Test connectivity layer by layer**

```
ping 172.16.0.1        → Reply ✅  (can reach DC on internal network)
ping 8.8.8.8           → Timeout ❌ (cannot reach internet)
ping google.com        → Timeout ❌ (not a DNS issue since IP also failed)
```

The problem was confirmed to be at the **routing layer**, not DNS.
The client was reaching the DC fine but the DC wasn't forwarding traffic outward.

**Step 3 — Inspect RAS on the server**

Opened **Routing and Remote Access** console on the DC.
The service showed as running (green arrow) but when I expanded
**IPv4 → NAT**, the public interface was not listed correctly.

<!-- YOU WRITE HERE — Describe what you specifically saw in your RAS console.
Example:
"The NAT configuration showed my internal interface (Ethernet 2) listed
as the public interface — it was backwards. This meant the server was
trying to NAT traffic out through the internal adapter instead of the
external one." -->

**Step 4 — Identify the root cause**

<!-- YOU WRITE HERE — What was the actual cause in your lab?
Example:
"The RAS NAT interface assignment had become stale — the external adapter
(Ethernet/NIC 1) was no longer listed as the internet interface.
This may have happened because I had renamed the adapters after
initially configuring RAS, causing it to lose the reference." -->

**Step 5 — Apply the fix**

<!-- YOU WRITE HERE — Exactly what did you do to fix it?
Example:
"I right-clicked the RAS server → All Tasks → Restart.
After the service restarted I re-confirmed the NAT interface assignments.
Alternatively: disabled and re-enabled RAS, re-ran the configuration
wizard and selected the correct external interface." -->

**Step 6 — Verify the fix**

Back on CLIENT2:
```
ping 8.8.8.8       → Reply ✅
ping google.com    → Reply ✅
```

Opened a browser on the client → successfully loaded a webpage. ✅

---

## Result

Full internet connectivity restored to the client workstation.
Root cause was a misconfiguration in the RAS NAT interface assignment
on the Domain Controller. The internal network remained isolated and
all client traffic continued to route through the DC as intended.

**Key lesson:** When a client can reach the DC but not the internet,
the problem is almost always at the routing/NAT layer on the gateway,
not on the client itself. Always test connectivity layer by layer
(internal → gateway → external) to isolate where the chain breaks.

---

## Commands Used

```powershell
ipconfig /all          # Verify client network config
ping 172.16.0.1        # Test internal connectivity to DC
ping 8.8.8.8           # Test external IP connectivity (bypasses DNS)
ping google.com        # Test DNS + external connectivity
```
