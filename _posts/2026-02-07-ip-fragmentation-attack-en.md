---
layout: post
title: "[Network Security] Bypassing Checks by Splitting Packages? A Complete Guide to IP Fragmentation Attacks"
date: 2026-02-07 09:00:00 +0900
categories: [Security, Network]
tags: [IP, Fragmentation, Teardrop, TinyFragment, ComputerScience, CyberSecurity]
---

# IP Fragmentation Attacks: Manipulating the Assembly Manual to Fool Servers

In the world of networking, data travels in "packets." However, just as not every road can accommodate a massive dump truck, every network path has a limit on the maximum size of a packet it can carry, known as the **MTU (Maximum Transmission Unit)**.

When a packet is too large for the MTU, the network splits it into smaller pieces. This process is called **Fragmentation**. The danger arises when attackers exploit the logic servers use to reassemble these "fragments."

---

## 0. What is Fragmentation?

Think of fragmentation as a dump truck trying to pass through a narrow tunnel. To get through, the cargo is offloaded onto several smaller vans. Once they pass through the tunnel, the cargo is reassembled back into its original form at the destination.



### ❓ "Is fragmentation unique to IP (Layer 3)?"
**No, but the attacks we discuss mostly target the unique way IP handles it.**

* **L4 (TCP - Segmentation):** TCP is cautious. It checks the MTU at the source and cuts the data into pieces *before* sending them.
* **L3 (IP - Fragmentation):** IP is reactive. If a router in the middle of the path finds a packet too big for the next hop, it splits it **in real-time**.

Attackers exploit the flexibility (and the subsequent reassembly chaos) of this **real-time IP fragmentation**.



---

## 1. The Assembly Manual: Key IP Header Fields

For a server to reassemble fragments, it needs an "assembly manual" found in the IP header:

1.  **Identification (ID):** "We belong together" (Fragments with the same ID are part of the same original packet).
2.  **MF (More Fragment) Flag:** "There's more coming!" (1 means more fragments follow; 0 means this is the last one).
3.  **Fragment Offset:** "Where do I fit?" (Tells the server the exact byte position where this fragment starts).



---

## 2. Tiny Fragment: "Splitting the Contraband to Hide It"

Firewalls typically check only the **first fragment ($Offset = 0$)** to determine where the packet is going (the Port number). Attackers exploit this by pushing the port information into the second fragment.

### ❓ Why does the IP Header persist while the TCP Header gets split?
* **IP Header (The Shipping Label):** Every box needs a destination address to be delivered. Therefore, the IP layer creates a **new shipping label (IP Header)** for every single fragment.
* **TCP Header (The Content):** To the IP layer, the TCP header is just **"data"** inside the box. It doesn't care if it's a letter (TCP Header) or a gift (Data); it will cut it wherever it needs to meet the size limit.

### 💻 Packet Structure Example
| Fragment | ID | Offset | Payload Length | Contents |
| :--- | :--- | :--- | :--- | :--- |
| **Fragment #1** | 0x1111 | 0 | **8 bytes** | IP Header + first 8 bytes of TCP Header (**No Port Info!**) |
| **Fragment #2** | 0x1111 | 1 | 100 bytes | IP Header + **TCP Port Info (The Target)** + Data |

**The Result:** The firewall sees Fragment #1, notices no "dangerous" port info, and lets it through. It sees Fragment #2 as a harmless follow-up and lets it through as well. The server reassembles them, and suddenly, a connection to a forbidden port (e.g., Port 21 for FTP) is successful.



---

## 3. Teardrop: "The Curse of Overlapping Puzzle Pieces"

This attack crashes the system (Kernel Panic) by confusing the server's reassembly math.

### 🛠️ The "Overlapping" Trick
* **Fragment 1:** $Offset = 0$, Length $= 100$ (Data for bytes 0–99)
* **Fragment 2:** $Offset = 60$, Length $= 30$ (Data for bytes 60–89)

**Server's Logic:** "Fragment 2 starts at 60, but I already have data up to 100. Let me subtract the overlap to see what's new."
> **The Formula:** $$(Current\ End\ Point) - (Previous\ End\ Point) = New\ Data\ Length$$
> $$89 - 100 = -11$$

**The Result:** The server calculates a **negative data length ($-11$)**. Older operating systems that didn't expect negative numbers would suffer a memory error and crash immediately (Blue Screen of Death).



---

## 4. Bonk & Boink: "The Broken Record"

These attacks aim to stall the system or exhaust its resources.

* **Bonk:** Sends every fragment with an $Offset$ of **1**. The server keeps waiting for the "actual" next piece that never arrives, eventually running out of memory.
* **Boink:** Sends fragments normally (0, 100, 200...), then suddenly sends **an offset it already received** (e.g., 100 again). This confuses the reassembly routine, causing the system to hang or crash.

---

## 📊 Comparison of Attack Types

| Attack | Targeted Field | The Trick | Goal |
| :--- | :--- | :--- | :--- |
| **Tiny Fragment** | Length, Offset | Split fragment smaller than headers | **Firewall Bypass** |
| **Teardrop** | Offset | Overlap fragment ranges | **System Crash (Panic)** |
| **Bonk / Boink** | Offset | Constant or duplicate offsets | **System Denial/Stall** |

---

## 🛡️ Defense Strategies

1.  **Tiny Fragment Blocking:** Configure firewalls to drop any first fragment ($Offset=0$) that is smaller than the minimum TCP header size (20 bytes).
2.  **Stateful Inspection:** Use modern firewalls that wait for all fragments to arrive and reassemble them *before* performing security checks.
3.  **OS Hardening:** Ensure systems are patched. Modern OS kernels have robust logic to handle overlapping offsets and negative length calculations.

---

> **Pro-Tip**: These concepts are favorites for network security certification exams (like ISP) because they demonstrate a deep understanding of how the L3 and L4 layers interact—or fail to interact!