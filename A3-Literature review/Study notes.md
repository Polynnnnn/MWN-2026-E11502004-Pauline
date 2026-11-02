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

### Step 2: Traffic Profile & Rates
- Each station sends **4 different data streams** at the same time (one for each QoS priority level).
- Initially, I tried sending 250 kbps per stream, but I realized the original paper used a higher load. So, I updated my script to send **1 Mbps per stream** (meaning each station generates **4 Mbps total**).
- Packets were set to 512 bytes, sent using UDP.

### Step 3: Wi-Fi & EDCA Parameters
- **Wi-Fi Standard:** IEEE 802.11a operating at 5 GHz.
- **Physical Data Rate:** 24 Mbps for data frames and 6 Mbps for ACK frames.
- **QoS Priority Rules (EDCA Settings):**
  - **AC_VO (Voice):** Highest priority ($AIFSN = 2$, $CW_{min} = 7$, $CW_{max} = 15$)
  - **AC_VI (Video):** High priority ($AIFSN = 2$, $CW_{min} = 15$, $CW_{max} = 31$)
  - **AC_BE (Best Effort):** Medium priority ($AIFSN = 3$, $CW_{min} = 15$, $CW_{max} = 1023$)
  - **AC_BK (Background):** Lowest priority ($AIFSN = 7$, $CW_{min} = 15$, $CW_{max} = 1023$)

### Step 4: Running Multiple Seeds for Accuracy
To make sure the results were reliable and accurate, I wrote a Bash script (`run_density_experiment.sh`) to run the simulation **5 times with different random seeds** for every station count ($N$). Then, I averaged the results across all 5 runs.

---
## 3. My Simulation Results

Here are the final average throughput values (in Mbps) that I got from my ns-3 runs:

| Stations ($N$) | AC_VO (Voice) | AC_VI (Video) | AC_BE (Best Effort) | AC_BK (Background) |
| :---: | :---: | :---: | :---: | :---: |
| **1** | 0.9992 Mbps | 0.9992 Mbps | 0.9992 Mbps | 0.9989 Mbps |
| **2** | 1.9991 Mbps | 1.9991 Mbps | 1.9990 Mbps | 1.9986 Mbps |
| **3** | 2.9985 Mbps | 2.9986 Mbps | 2.9992 Mbps | **0.9110 Mbps** |
| **4** | 3.9907 Mbps | 3.9887 Mbps | **1.3191 Mbps** | 0.3143 Mbps |
| **5** | 4.9206 Mbps | 4.9146 Mbps | 0.3716 Mbps | 0.1687 Mbps |
| **6** | 5.7573 Mbps | 5.7820 Mbps | 0.3207 Mbps | 0.1608 Mbps |
| **8** | 6.2817 Mbps | 5.7496 Mbps | 0.1943 Mbps | 0.1095 Mbps |
| **12** | 6.1534 Mbps | 5.7312 Mbps | 0.2238 Mbps | 0.1116 Mbps |
| **16** | 6.4255 Mbps | 5.6523 Mbps | 0.0815 Mbps | 0.0458 Mbps |

---
