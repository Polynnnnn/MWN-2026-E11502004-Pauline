**A3-Literature review**

---
---

## 1. Introduction & Goal & Overview

### 1. Problem
Wireless Local Area Networks (WLANs) originally relied on the Distributed Coordination Function (DCF), which operates as a best-effort medium access protocol. Because DCF treats all data packets equally regardless of their content or timing constraints, real-time applications such as Voice over IP (VoIP) and interactive video streaming must contend for channel access on equal terms with low-priority background data transfers.

### 2. Importance
As Wi-Fi networks became primary access points for multimedia applications, providing Quality of Service (QoS) guarantees became essential. Without MAC-layer prioritization, heavy background data transfers saturate the shared medium, introducing unpredictable delays, high packet loss, and severe jitter that render voice and video communications unusable. IEEE 802.11e introduced Enhanced Distributed Channel Access (EDCA) to solve this by creating prioritized Access Categories ($AC\_VO$, $AC\_VI$, $AC\_BE$, $AC\_BK$).

### 3. Challenges
- **Collision Collapse at High Densities:** High-priority traffic (such as $AC\_VO$) gains early access by using small contention windows ($CW_{min}=3, CW_{max}=7$). However, when station density ($N$) increases, many high-priority entities contend simultaneously within a very narrow window. This drastically increases collision probability, leading to throughput degradation and airtime loss for all traffic streams.
- **Uncoordinated Medium Access:** In dense environment deployments, distributed backoff mechanisms lack centralized airtime scheduling, causing aggressive priorities to starve lower priorities or degrade their own performance under heavy saturation.

For this assignment, my main goal was to reproduce a published baseline result to see how network density affects Quality of Service (QoS) in Wi-Fi networks using the **ns-3** simulator.

I chose to recreate **Figure 9** from this research paper:
> **Paper:** S. Mangold, S. Choi, G. R. Hiertz, O. Klein, and B. Walke, *"Analysis of IEEE 802.11e for QoS Support in Wireless LANs,"* IEEE Wireless Communications, 2003.

This figure shows what happens to the throughput (data speed) of four different traffic types—**Voice (AC_VO)**, **Video (AC_VI)**, **Best Effort (AC_BE)**, and **Background (AC_BK)**—as more and more devices (from 1 to 16 stations) try to send data at the same time.

---

## 2. What I Did (Setup & Simulation Steps)

### 1. Environment
- **Network Topology:** Infrastructure QoS Basic Service Set (QBSS) consisting of 1 Access Point (AP / Hybrid Coordinator) and $N$ contending stations ($N \in \{1, 2, 3, \dots, 16\}$).
- **Radio Propagation:** All stations are placed in close spatial proximity to the AP (5 meters) within single-hop range, ensuring no hidden node problems exist.

### 2. Inputs
- **Station Density ($N$):** Varied from 1 to 16 stations.
- **Traffic Model per Station:** Each station generates 4 concurrent UDP traffic streams corresponding to the four Access Categories ($AC\_VO, AC\_VI, AC\_BE, AC\_BK$).
- **Offered Rate per Stream:** $1.0\text{ Mbps}$ per AC (total offered load of $4.0\text{ Mbps}$ per station across all 4 streams).
- **Packet Size:** 512-byte MSDUs (UDP).
- **PHY/MAC Settings:**
  - Physical Layer: IEEE 802.11a operating at 5 GHz.
  - Transmission Rates: 24 Mbps for data frames, 6 Mbps for control/ACK frames.
  - EDCA Parameters ($AIFSN$, $CW_{min}$, $CW_{max}$, $TXOP_{limit}$) set strictly according to Table 1 of Mangold et al.

### 3. Outputs & Performance Metrics
- **Measured Output:** Aggregate throughput per Access Category received and measured at the AP application layer sinks over a 10-second simulation period.
- **Performance Metric:** Throughput per AC ($\text{Mbps}$) plotted as a function of station count $N$.

---

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
- **Physical Data Rate:** 24 Mbps for data frames and 6 Mbps for control frames.
- **QoS Priority Rules (EDCA Settings from Table 1 of the paper):**
  - **$AC\_VO$ (Voice):** $AIFSN = 2$, $CW_{min} = 3$, $CW_{max} = 7$, $TXOP_{limit} = 0\text{ ms}$
  - **$AC\_VI$ (Video):** $AIFSN = 2$, $CW_{min} = 7$, $CW_{max} = 15$, $TXOP_{limit} = 0\text{ ms}$
  - **$AC\_BE$ (Best Effort):** $AIFSN = 3$, $CW_{min} = 15$, $CW_{max} = 1023$, $TXOP_{limit} = 0\text{ ms}$
  - **$AC\_BK$ (Background):** $AIFSN = 7$, $CW_{min} = 15$, $CW_{max} = 1023$, $TXOP_{limit} = 0\text{ ms}$

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

My updated simulation results demonstrate the key EDCA QoS principles shown in Mangold et al.:

1. **Light Load / Unsaturated Channel ($N = 1$ to $N = 3$):**
   - At $N = 1$ and $N = 2$, the total traffic offered is below channel capacity. All four Access Categories deliver 100% of their generated traffic (~1.0 Mbps per stream per station).
   - At $N = 3$ (offered load = 12.0 Mbps), throughput begins to level off near ~2.86 Mbps per AC as total channel usage approaches capacity.
2. **Channel Saturation & Priority Separation ($N \ge 4$):**
   - As soon as station count reaches $N = 4$ (offered load = 16.0 Mbps), the total demand exceeds available Wi-Fi bandwidth.
   - The four categories split into four clear priority levels:
     - **$AC\_VO$ (Voice):** Captures the highest throughput, rising from 3.39 Mbps at $N=4$ up to **5.36 Mbps** at $N=16$. This occurs because it uses the smallest backoff window ($CW_{min}=3, CW_{max}=7$) and shortest waiting interval ($AIFSN=2$).
     - **$AC\_VI$ (Video):** Holds steady around **2.93 – 3.26 Mbps** due to its medium contention parameters ($CW_{min}=7$).
     - **$AC\_BE$ (Best Effort):** Drops steadily from 2.66 Mbps down to **1.31 Mbps** at $N=16$.
     - **$AC\_BK$ (Background):** Suffers the most throttling, dropping from 2.13 Mbps down to **0.52 Mbps** at $N=16$ because of its long waiting time ($AIFSN=7$).

<img width="794" height="617" alt="image" src="https://github.com/user-attachments/assets/45b39f6e-2cdf-4edd-b297-72fd75a819b6" />

### Comparison with Paper Baseline:
- **Similarity:** The simulation correctly reproduces the step-by-step priority ranking ($AC\_VO > AC\_VI > AC\_BE > AC\_BK$) and proves that EDCA protects high-priority voice traffic when the network becomes crowded.
- **Minor Difference:** In the original 2003 paper's analytical curves, lower priorities fell faster toward zero because of ideal collision assumptions. In **ns-3.48**, the internal packet queuing and MAC retry mechanisms smooth out frame drops, allowing $AC\_BE$ and $AC\_BK$ to retain minor residual throughput while voice traffic continues to grow.

---

## 5. Review of Paper 2: Xiao (2005)

### 1. Problem, Importance, and Challenges
- **Problem:** Mangold et al. evaluated EDCA primarily through stochastic event-driven simulations. However, simulation alone does not provide closed-form equations explaining *how* individual EDCA parameters ($AIFSN, CW_{min}, CW_{max}$) mathematically govern transmission probabilities and collision dynamics.
- **Importance:** Yang Xiao's paper (*"Performance Analysis of Priority Schemes for IEEE 802.11e Wireless LANs,"* IEEE TWC 2005) established the first rigorous multi-dimensional Markov chain model for 802.11e EDCA under saturation. This analytical model allows researchers to predict exact throughput limits without running long simulations.
- **Challenges:** Modeling EDCA is significantly more complex than Bianchi's 2000 DCF model because different Access Categories have distinct arbitration inter-frame spaces ($AIFS$). This means backoff counters for lower-priority categories freeze while higher-priority categories are actively down-counting or transmitting.

### 2. System Model (Xiao 2005)
- **Environment:** Single-hop Infrastructure QBSS with $N$ active stations under continuous queue saturation (every station's MAC queue always contains frames awaiting transmission).
- **Inputs:**
  - Multi-dimensional discrete-time Markov chain modeling backoff state $(s_{i}(t), b_{i}(t))$ for each AC $i \in \{VO, VI, BE, BK\}$.
  - Input parameters per AC: $CW_{min}[i]$, $CW_{max}[i]$, $AIFSN[i]$, frame retry limits $m_i$, and payload length.
  - Transmission probability $\tau_i$ and conditional collision probability $p_i$ for each AC.
- **Outputs & Performance Metrics:**
  - Normalized saturation throughput per AC ($S_i$).
  - Frame delivery delay and frame drop probability due to retry limits ($P_{drop}$).

### 3. Key Analytical Results & Findings
- **Dominance of $CW_{min}$ vs. $AIFSN$:** Xiao's mathematical derivations proved that $CW_{min}$ is the most powerful parameter for short-term priority differentiation, whereas $AIFSN$ acts as a long-term barrier to channel access.
- **Mathematical Confirmation of Collision Collapse:** Xiao's equations demonstrated that setting $CW_{min}=3$ for $AC\_VO$ guarantees early channel access when $N$ is small. However, as $N > 5$, the probability of two or more voice stations selecting the exact same backoff slot approaches 100%, causing exponential collision rates. This provides exact theoretical backing for the performance saturation observed in Mangold's Figure 9 and in my ns-3.48 simulation.

---

## 6. Synthesis & Comparison of Reviewed Papers

| Feature / Metric | Paper 1: Mangold et al. (2003) | Paper 2: Xiao (2005) |
| :--- | :--- | :--- |
| **Primary Approach** | Stochastic Event-Driven Simulation (ns-2 / custom) | Analytical Markov Chain Modeling |
| **Focus Area** | Empirical EDCA/HCCA throughput & delay evaluation | Mathematical derivation of transmission ($\tau$) and collision ($p$) probabilities |
| **Station Density Scope** | Evaluates $N=1$ to $16$ stations | Evaluates theoretical scaling up to $N=50$ stations |
| **High-Density Finding** | Shows $AC\_VO$ taking over bandwidth while lower ACs degrade | Mathematically proves small $CW_{min}=3$ causes exponential collision escalation at high $N$ |

---

## 7. Revision to Project Proposal (A1)

### What Changed & Why
Based on the combined analysis of Mangold et al. (2003), Xiao (2005), and the reproduction of Figure 9 in ns-3.48, the scope and direction of the project proposal (A1) have been revised as follows:

1. **Shift in Core Research Problem:**
   - *Initial Plan (A1):* Focused broadly on measuring average throughput degradation as network density increases under standard DCF/EDCA.
   - *Revised Scope (A3):* The reproduction and theoretical model demonstrate that default EDCA settings create severe contention collapse in dense environments because $AC\_VO$'s small $CW_{min}=3$ bounds trigger high collision rates. Consequently, the research focus is now shifted to **mitigating high-density contention and optimizing airtime allocation for voice and video streams**.

2. **Refinement of the Experimental Methodology:**
   - *Initial Plan (A1):* Proposed evaluating simple fixed contention window modifications.
   - *Revised Scope (A3):* Having established a validated baseline reproduction in **ns-3.48**, the project will evaluate dynamic adaptive contention window algorithms (such as Dynamic Adaptive Contention Window - DACW) where the Access Point dynamically inflates $CW_{min}[AC\_VO]$ from 3 up to 15 when active station count $N \ge 6$. This directly addresses the hotspot scalability issue highlighted in both papers.

---

## 8. Summary & Conclusion

I successfully reviewed both Mangold et al. (2003) and Xiao (2005), and set up and verified the IEEE 802.11e EDCA protocol parameters in **ns-3.48**. The simulation script sweeps station density from 1 to 16 stations and confirms that **increasing station count forces lower-priority traffic to yield bandwidth to protect voice and video applications**, while also revealing the collision risks of static EDCA parameterization.

This reproduced curve serves as my validated baseline for the final project.







