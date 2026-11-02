**A3-Literature review**

---
---

## 1. Introduction & Goal

For this assignment, my main goal was to reproduce a published baseline result to see how network density affects Quality of Service (QoS) in Wi-Fi networks using the **ns-3** simulator.

I chose to recreate **Figure 9** from this research paper:
> **Paper:** S. Mangold, S. Choi, G. R. Hiertz, O. Klein, and B. Walke, *"Analysis of IEEE 802.11e for QoS Support in Wireless LANs,"* IEEE Wireless Communications, 2003.

This figure shows what happens to the throughput (data speed) of four different traffic types—**Voice (AC_VO)**, **Video (AC_VI)**, **Best Effort (AC_BE)**, and **Background (AC_BK)**—as more and more devices (from 1 to 16 stations) try to send data at the same time.

---

## 2. What I Did (Setup & Simulation Steps)

To get these results, I set up a custom simulation script in **ns-3.48**. Here is how I set everything up:

### Step 1: Network Topology & Distance
- I created 1 Access Point (AP) and tested it with $N$ stations (where $N = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 14, 16$).
- I placed all stations close to the AP so that every station could "hear" the others (no hidden node problems).


