**A3-Literature review**

---

## 1. Introduction & Executive Summary

The objective of this assignment is to reproduce a key published baseline result in **ns-3** to evaluate the impact of network density on Quality of Service (QoS) mechanisms in Wireless LANs.

The baseline study that I selected for reproduction is **Figure 9** from the landmark paper:
> **Reference:** S. Mangold, S. Choi, G. R. Hiertz, O. Klein, and B. Walke, *"Analysis of IEEE 802.11e for QoS Support in Wireless LANs,"* IEEE Wireless Communications, vol. 10, no. 6, pp. 40–50, Dec. 2003.

Figure 9 evaluates the achievable throughput per Access Category (AC) in an isolated QoS Basic Service Set (QBSS) under **Enhanced Distributed Channel Access (EDCA)** as the number of contending stations increases from 1 to 16.

---

## 2. Experimental Environment & Setup

The simulation environment was developed using **ns-3 (v3.48)** under macOS.

### 2.1 Topology & Traffic Profile
- **Topology:** Single Access Point (AP) with $N$ contending Stations (STAs), where $N \in \{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 14, 16\}$.
- **Distance:** All stations placed within close physical proximity to the AP (no hidden nodes).
- **Traffic Model:** 4 parallel UDP streams per STA (one stream per Access Category).
- **Offered Rate:** 1 Mbps per stream (Total offered load = $4 \times 1\text{ Mbps} = 4\text{ Mbps}$ per STA).
- **Packet Size (MSDU):** 512 bytes with Poisson inter-arrival times.

### 2.2 Protocol & Parameter Settings

| Parameter | Value / Configuration |
| :--- | :--- |
| **Simulator** | ns-3.48 |
| **Wi-Fi Standard** | IEEE 802.11a (5 GHz) |
| **Data Rate (PHY)** | 24 Mbps (`OfdmRate24Mbps`) |
| **Control/ACK Rate** | 6 Mbps (`OfdmRate6Mbps`) |
| **RTS/CTS Threshold** | Disabled (999999 bytes) |
| **TXOP Limit** | 1 packet per TXOP |

### 2.3 EDCA Contention Parameters (Draft 5.0 Defaults)

| Access Category (AC) | Priority | AIFSN | $CW_{min}$ | $CW_{max}$ |
| :--- | :--- | :--- | :--- | :--- |
| **AC_VO** (Voice) | Highest | 2 | 7 | 15 |
| **AC_VI** (Video) | High | 2 | 15 | 31 |
| **AC_BE** (Best Effort) | Medium | 3 | 15 | 1023 |
| **AC_BK** (Background) | Lowest | 7 | 15 | 1023 |

---

## 3. Repeatability: Seeds & Simulation Runs

To obtain statistically stable throughput averages, each data point ($N$) was executed across **5 independent random seeds** using standard ns-3 `RngSeedManager` seeds ($1 \dots 5$).

### Executed Runs Data (`edca_density_results.dat`)

```text
# STAs  AC_VO (Mbps)  AC_VI (Mbps)  AC_BE (Mbps)  AC_BK (Mbps)
1       0.9992        0.9992        0.9992        0.9989
2       1.9991        1.9991        1.9990        1.9986
3       2.9985        2.9986        2.9992        0.9110
4       3.9907        3.9887        1.3191        0.3143
5       4.9206        4.9146        0.3716        0.1687
6       5.7573        5.7820        0.3207        0.1608
7       6.1112        5.6081        0.2299        0.1207
8       6.2817        5.7496        0.1943        0.1095
9       6.4995        5.6429        0.1381        0.0817
10      6.3156        5.7107        0.1353        0.0695
11      6.2550        5.5504        0.1433        0.0808
12      6.1534        5.7312        0.2238        0.1116
14      6.1967        5.4823        0.1373        0.0794
16      6.4255        5.6523        0.0815        0.0458
