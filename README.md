# README — ICMP Attack Lab (ICMP Flood & Smurf Concept)  
**Monitoring with `iptraf-ng` & `nload`, mitigation with `iptables` (2-host / 2-VM setup)**

> **Ethics & safety note (important):**  
> This lab is intended for **authorized educational environments only**. Do **not** run flooding, spoofing, or packet-crafting tools on networks you do not own or without explicit permission. Keep tests isolated (e.g., local hotspot / internal lab LAN) and use conservative rates.

---

## 1) Overview

This project demonstrates how ICMP traffic can be abused to reduce availability (DoS) and how the impact can be **observed** and **mitigated** using common Linux tools:

- **Server/Target (Linux)**  
  Monitoring: `iptraf-ng`, `nload`  
  Defense: `iptables` (block/rate-limit ICMP)

- **Peer/Attacker (Linux / “friend”)**  
  Generates ICMP traffic for testing (e.g., controlled `ping` usage; optional packet generator tools depending on course instructions)

The main deliverables are:
1) Baseline monitoring screenshots  
2) Normal ICMP (non-flood) screenshot  
3) ICMP flood condition screenshots (traffic spike evidence)  
4) Smurf **concept** discussion + **defensive control**  
5) Comparison: before vs under attack  
6) Comparison: under attack **with** vs **without** `iptables`

---

## 2) Lab Topology (Typical)

You can run this lab in several safe ways:

- **Same LAN (home router)**
- **Mobile hotspot / Wi‑Fi tethering**
- **VirtualBox Bridged Adapter** (VM appears as a device on the LAN)

Both machines must be in the **same subnet** and able to ping each other.

---

## 3) Requirements

**Host / Environment**
- Windows / Linux host (if using VMs)
- VirtualBox (optional)  
- 2 Linux systems (physical or VM)

**Software (Server/Target)**
- `iptraf-ng`
- `nload`
- `iptables` (usually preinstalled)

**Software (Peer/Attacker)**
- `ping` (installed by default)
- Optional packet generator tools if required by your class materials

---

## 4) Step 1 — Network Validation

### 4.1 Identify IP address and interface
On **each** machine:

```bash
ip a
ip route
```

Note:
- Your active interface might be `enp0s3`, `enp0s8`, `wlan0`, etc.
- Use the interface that is actually connected to the shared LAN/hotspot.

### 4.2 Ping test (connectivity)
From Peer/Attacker → Server:

```bash
ping <server_ip>
```

From Server → Peer/Attacker:

```bash
ping <peer_ip>
```

If both succeed, proceed.

---

## 5) Step 2 — Install Monitoring Tools (Server/Target)

```bash
sudo apt update
sudo apt install iptraf-ng nload -y
```

### 5.1 Run iptraf-ng
```bash
sudo iptraf-ng
```

Recommended views:
- **IP traffic monitor**
- **TCP/UDP/ICMP statistics (depending on menu options)**

### 5.2 Run nload
```bash
nload
```

If you have multiple interfaces, specify one:

```bash
nload <interface_name>
```

---

## 6) Step 3 — Baseline Measurements (No Attack)

1) Start `iptraf-ng` on the server and observe traffic for ~1–2 minutes  
2) Start `nload` on the server and record “Current / Max / Total” values  
3) Take **Baseline screenshots**:
   - Baseline `iptraf-ng`
   - Baseline `nload`

Expected baseline behavior:
- Very low bandwidth (near idle)
- Few flows
- Minimal ICMP entries

---

## 7) Step 4 — Normal ICMP Test (Non-Flood)

Run a **normal ping** from Peer/Attacker to Server:

```bash
ping -c 5 <server_ip>
```

Observe on Server:
- `iptraf-ng` should show ICMP echo request/reply activity
- `nload` should remain low (no major spike)

Take a screenshot for:
- “Normal ICMP (not flood)”

---

## 8) Step 5 — Attack Demonstration (Controlled)

### 8.1 ICMP Flood (concept)
An ICMP flood is when the target is overloaded by a high volume of ICMP echo requests.

**Safety guidance:**  
This README does **not** provide high-rate flood commands. If your instructor requires a flood demo, follow your **official lab instructions** and keep the test strictly isolated.

What to capture during the (authorized) test:
- `iptraf-ng` shows repeated ICMP lines / higher packet activity
- `nload` shows a **sharp bandwidth spike**
- (Optional) higher latency / reduced responsiveness

### 8.2 “Ping 1000 bytes” requirement (safe diagnostic example)
If your assignment asks for an ICMP request with ~1000-byte payload, do it in a **low-rate, limited-count** way.

**Option A: ping (payload size)**
```bash
ping -c 3 -s 1000 <server_ip>
```

**Option B: packet generator tools**
Use the tool’s option for payload/data size and keep count/rate limited.  
Refer to the official manual page below.

```text
https://linux.die.net/man/8/hping3
```

---

## 9) Step 6 — Smurf Attack (Concept) + Defensive Control

### 9.1 Smurf concept (high-level)
A Smurf-style attack abuses **broadcast addressing** and **spoofed sources** to amplify ICMP responses toward a victim.

### 9.2 Defensive controls (recommended)
**Goal:** prevent or reduce broadcast-based ICMP amplification.

On Linux networks and routers (where applicable):
- Disable directed broadcasts on routers
- Prevent hosts from responding to broadcast ICMP requests
- Filter broadcast-directed ICMP at boundaries

On the server (host-based examples):

**Rate-limit ICMP echo requests**
```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

**Block ICMP from a known malicious source IP**
```bash
sudo iptables -A INPUT -p icmp -s <attacker_ip> -j DROP
```

> Replace `<attacker_ip>` with the peer/attacker IP in your lab.

---

## 10) Step 7 — Compare With vs Without iptables (Assignment Item)

To compare results:

1) **Without iptables rules**
   - Run baseline → run authorized test → capture `iptraf-ng` + `nload`

2) **With iptables rules enabled**
   - Add rate-limit or block rule(s)
   - Repeat the same authorized test
   - Capture `iptraf-ng` + `nload` again

Document:
- Change in bandwidth (nload)
- Change in visible ICMP activity (iptraf-ng)
- Any improvement in responsiveness

---

## 11) Cleanup

### 11.1 Remove tools (optional)
```bash
sudo apt remove iptraf-ng nload -y
sudo apt autoremove -y
sudo apt clean
```

### 11.2 Remove iptables rules (if this is a lab machine)
**Warning:** only do this if you understand the impact.

List current rules:
```bash
sudo iptables -S
```

Flush rules (reset):
```bash
sudo iptables -F
sudo iptables -X
```

---

## 12) Evidence Checklist (Screenshots)

Recommended evidence set (match your report figures):

- Peer/Attacker pinging server (connectivity)
- Server ping statistics / reachability
- Baseline `iptraf-ng`
- Baseline `nload`
- Normal ICMP (non-flood) in `iptraf-ng`
- Under ICMP flood: `iptraf-ng` evidence
- Under ICMP flood: `nload` spike evidence
- Smurf-style: `nload` / monitoring evidence (concept demonstration)
- With vs without `iptables` (comparison evidence)

---

## 13) Source Material

Slide deck used as reference in this project:
- `6_ICMP_attack.pptx.pdf`
