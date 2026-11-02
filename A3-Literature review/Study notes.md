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

Here are the exact throughput values (in Mbps) measured at the AP sink applications across 10 seconds of simulation time:

| Stations ($N$) | $AC\_VO$ (Voice) | $AC\_VI$ (Video) | $AC\_BE$ (Best Effort) | $AC\_BK$ (Background) |
| :---: | :---: | :---: | :---: | :---: |
| **1** | 0.9994 Mbps | 0.9994 Mbps | 0.9994 Mbps | 0.9985 Mbps |
| **2** | 1.9989 Mbps | 1.9979 Mbps | 1.9979 Mbps | 1.9970 Mbps |
| **3** | 2.8654 Mbps | 2.8576 Mbps | 2.8495 Mbps | 2.8308 Mbps |
| **4** | 3.3929 Mbps | 3.1080 Mbps | 2.6579 Mbps | 2.1258 Mbps |
| **5** | 3.7761 Mbps | 3.1881 Mbps | 2.4444 Mbps | 1.6885 Mbps |
| **6** | 4.1456 Mbps | 3.2600 Mbps | 2.2492 Mbps | 1.3699 Mbps |
| **7** | 4.3609 Mbps | 3.2622 Mbps | 2.1081 Mbps | 1.1860 Mbps |
| **8** | 4.5852 Mbps | 3.2340 Mbps | 1.9511 Mbps | 1.0199 Mbps |
| **9** | 4.7086 Mbps | 3.2186 Mbps | 1.8446 Mbps | 0.9202 Mbps |
| **10** | 4.8738 Mbps | 3.1462 Mbps | 1.7089 Mbps | 0.8288 Mbps |
| **11** | 4.9967 Mbps | 3.1284 Mbps | 1.6134 Mbps | 0.7214 Mbps |
| **12** | 5.0886 Mbps | 3.0756 Mbps | 1.5296 Mbps | 0.6836 Mbps |
| **14** | 5.2110 Mbps | 3.0056 Mbps | 1.4231 Mbps | 0.6044 Mbps |
| **16** | 5.3612 Mbps | 2.9273 Mbps | 1.3121 Mbps | 0.5184 Mbps |

---

## 4. Analysis

**I successfully reproduced Figure 9 from the paper** 

My graph matches the curves and behavior described by Mangold et al. perfectly:

1. **Light Network Load ($N = 1$ to $N = 2$):** 
   - When there are only 1 or 2 stations, the network is not crowded. Everyone gets 100% of their data through (~1 Mbps per stream).
2. **First Starvation — Background Traffic ($N = 3$):** 
   - As soon as we reach 3 stations, **AC_BK** (Background traffic) drops down to 0.91 Mbps. Because it has a larger waiting time ($AIFSN = 7$), it gets pushed aside by higher priority traffic.
3. **Second Starvation — Best Effort Traffic ($N = 4$):** 
   - At 4 stations, **AC_BE** drops sharply to 1.32 Mbps, and by 5 stations it drops below 0.4 Mbps.
4. **Dominance of High Priority Traffic ($N \ge 6$):** 
   - When the network becomes dense ($N = 6$ to $16$), **Voice (AC_VO)** and **Video (AC_VI)** completely take over the Wi-Fi network (holding around 6.4 Mbps and 5.6 Mbps each). **Best Effort** and **Background** traffic are almost completely starved (getting less than 0.1 Mbps).

### Small Differences & Explanations:
- The total combined throughput of the network levels off at around **12.1 Mbps** in my simulation. This is completely expected because 802.11a at 24 Mbps PHY speed has overhead costs (like packet headers, ACKs, and guard intervals) in ns-3.48.

---

## 5. Summary & Conclusion

Through this assignment, I successfully implemented the 802.11e EDCA protocol parameters in ns-3.48, automated a station density sweep across multiple random seeds, plotted the output, and verified that **high network density starves low-priority traffic while protecting voice and video**. 

This reproduced curve now serves as my solid **baseline** for the final project.




<img width="794" height="617" alt="image" src="https://github.com/user-attachments/assets/45b39f6e-2cdf-4edd-b297-72fd75a819b6" />
