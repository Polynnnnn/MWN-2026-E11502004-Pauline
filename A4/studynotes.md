# Assignment A4 — Reproducing and Analyzing IEEE 802.11e EDCA (Mangold et al., 2003) in ns-3.48

**Reference Paper:** S. Mangold, S. Choi, G. R. Hiertz, O. Klein, and B. Walke, *"Analysis of IEEE 802.11e for QoS Support in Wireless LANs"*, IEEE Wireless Communications, vol. 10, no. 6, pp. 40–50, Dec. 2003.

---

## 1. Goal of This Assignment and Experiment Plan

The goal of this assignment is to reproduce the experiments from the paper by Mangold et al. (2003) using a modern network simulator (`ns-3.48`). In their paper, the authors tested how **IEEE 802.11e EDCA** (*Enhanced Distributed Channel Access*) gives priority to important traffic (like Voice and Video) over regular traffic (like Best Effort and Background) when more wireless devices join a Wi-Fi network ($N = 1$ to $16$ stations).

Because `ns-3.48` is a modern simulator and the 2003 paper used an older draft version of the Wi-Fi standard, some numbers naturally differ. To make sure my results are solid and to prove *why* those differences happen, I ran **two complete experiments**:

1. **Experiment 1 (Baseline — Throughput vs. Number of Stations):** I built the exact same network setup without RTS/CTS and measured the total throughput (in Mbps) of all four traffic classes (`AC_VO`, `AC_VI`, `AC_BE`, `AC_BK`) as the number of stations grew from $1$ to $16$, then plotted my results directly on top of Mangold's curves.
2. **Experiment 2 (Testing RTS/CTS Overhead & Measuring Delay):** To prove why my total throughput in Experiment 1 was higher than in the 2003 paper, I turned on **RTS/CTS** (`RtsCtsThreshold = 0`) and also measured the **average packet delay** (in milliseconds, on a log scale) using `FlowMonitor`. This checks whether Voice and Video still stay fast when the network gets crowded.

---

## 2. Simulation Setup and Exact Parameters

To make sure my results are reliable and not just lucky on one random run, I ran every station count ($N = 1$ to $16$) **10 times with 10 different random seeds** (`160 runs` for Experiment 1 + `160 runs` for Experiment 2 = **320 simulations total**). For every point on my graphs, I calculated the average and the **95% confidence interval**.

| Setting | What I Used in `ns-3.48` | Why I Set It This Way |
| :--- | :--- | :--- |
| **Simulator** | `ns-3.48` (macOS) | Updated code to use `FrameRetryLimit` and `WifiMacQueue::MaxDelay` |
| **Wi-Fi Standard** | IEEE 802.11a (5 GHz) with QoS enabled | Matches the 5 GHz OFDM radio used in the 2003 paper |
| **Data & Control Speeds** | Data: `24 Mbps` / Control ACKs: `6 Mbps` | Fixed using `ConstantRateWifiManager` so rate adaptation doesn't change results |
| **Frame Aggregation** | Turned off (`MaxAmpduSize = 0`, `MaxAmsduSize = 0`) | Back in 2003 (Draft 4.0), Wi-Fi didn't bundle multiple frames together |
| **Max Retries** | `FrameRetryLimit = 4` | Drops frames after 4 failed tries so collided packets don't block the queue forever |
| **Network Layout** | 1 Access Point + $N = 1..16$ Stations ($1\text{ m}$ apart) | Placing everyone close together avoids hidden-node problems |
| **Traffic Sent** | 4 UDP streams per station (`1 Mbps` each, `512 B` packets) | Each station tries to send $4\text{ Mbps}$ total ($1\text{ Mbps}$ per priority class) |
| **Simulation Time** | $10\text{ seconds}$ total (first $1\text{ second}$ ignored) | Gives the network $1\text{ s}$ to warm up, then measures steady traffic for $9\text{ s}$ |
| **Random Seeds** | `SetSeed(1)`, `SetRun(1..10)` | 10 independent runs per density to compute 95% confidence intervals |

### EDCA Settings for Each Traffic Class

| Traffic Class | IP TOS Tag | `AIFSN` (Wait Time) | $CW_{\min}$ | $CW_{\max}$ | Queue Timeout (`MaxDelay`) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Voice (`AC_VO`)** | `0xC0` | 2 | 3 | 7 | $100\text{ ms}$ |
| **Video (`AC_VI`)** | `0x80` | 2 | 7 | 15 | $100\text{ ms}$ |
| **Best Effort (`AC_BE`)** | `0x00` | 3 | 15 | 1023 | $500\text{ ms}$ (default) |
| **Background (`AC_BK`)** | `0x20` | 7 | 15 | 1023 | $500\text{ ms}$ (default) |

---

## 3. Results and What They Show

### 3.1 Experiment 1: Throughput vs. Number of Stations (`A4_overlay_comparison.png`)

When I put my `ns-3.48` curves (solid lines with 95% error bars) on top of Mangold's curves (dashed lines), the network clearly goes through three stages:

1. **Low Traffic ($N = 1$ to $3$ stations):** The Wi-Fi channel isn't full yet, so almost every class gets the full $1\text{ Mbps}$ per station it asks for ($1\text{ Mbps}$ at $N=1$, $2\text{ Mbps}$ at $N=2$). Only `AC_BK` starts slowing down at $N=3$ ($\approx 0.78\text{ Mbps}$) because it has to wait longer (`AIFSN = 7`) before transmitting.
2. **Saturation Point ($N = 4$ stations):** At 4 stations ($16\text{ Mbps}$ total offered load), the Wi-Fi channel hits its limit. EDCA immediately protects Voice and Video: `AC_VO` ($\approx 3.85\text{ Mbps}$) and `AC_VI` ($\approx 3.84\text{ Mbps}$) keep going up, while `AC_BE` drops to $\approx 1.29\text{ Mbps}$ and `AC_BK` falls to $\approx 0.13\text{ Mbps}$.
3. **Crowded Network ($N = 6$ to $16$ stations):** From 6 stations onward, low-priority traffic gets completely starved (`AC_BK = 0.00 Mbps` and `AC_BE < 0.03 Mbps`), which matches Mangold's paper very closely. Meanwhile, Voice stays steady around **$5.90 \pm 0.10\text{ Mbps}$** and Video stays around **$5.20 \pm 0.15\text{ Mbps}$**.

### 3.2 Experiment 2: Adding RTS/CTS and Measuring Packet Delay (`A4_exp2_rts_throughput.png` & `A4_exp2_access_delay.png`)

To go deeper than a single graph, Experiment 2 turns on **RTS/CTS** and measures how long packets actually wait before arriving at the Access Point:
* **Throughput Drop from RTS/CTS (`Experiment 2A`):** Sending RTS and CTS handshake frames before every `512 B` data packet uses extra airtime. This pulls the maximum combined throughput down from ~11.1 Mbps toward Mangold's ~8.4 Mbps ceiling, confirming my hypothesis about control overhead.
* **Delay Protection for Voice and Video (`Experiment 2B`):** Looking at the delay graph (log scale), when $N = 1$ or $2$, every class has very low delay (under $4\text{ ms}$, except `AC_BK` rising to $\sim 70\text{ ms}$ at $N=2$). As soon as $N \ge 3$, `AC_BE` and `AC_BK` shoot up by $100\times$ and get stuck around **$300\text{–}450\text{ ms}$** (hitting the $500\text{ ms}$ queue limit) with huge error bars. On the other hand, **Voice (`AC_VO`) stays super stable between $24\text{ ms}$ and $32\text{ ms}$** all the way to $N = 16$ (and Video stays around **$45\text{ ms}$**), which is well under the $100\text{ ms}$ limit needed for smooth calls and video.

---

## 4. Why My Curves Don't Match 100% (Error Analysis)

Even though my simulation gets the exact same priority order ($\text{AC\_VO} > \text{AC\_VI} > \text{AC\_BE} > \text{AC\_BK}$) and the exact same starvation point ($N \ge 6$), there are two main differences in Experiment 1:

1. **Video (`AC_VI`) gets more throughput (~5.2 Mbps vs. ~3.0 Mbps in the paper):**
   * *Why:* In my `802.11a` setup, both `AC_VO` and `AC_VI` use the exact same wait time (**`AIFSN = 2`**) and send one frame at a time (**`TXOP Limit = 0`**). Once `AC_BE` and `AC_BK` are pushed off the channel, having $CW \in [7,15]$ for Video versus $CW \in [3,7]$ for Voice is a small difference, so Video wins almost as much airtime as Voice.
2. **Higher total throughput (~11.1 Mbps in Exp. 1 vs. ~8.4 Mbps in the 2003 paper):**
   * *Why:* Mangold et al. (2003) simulated an early draft (**802.11e Draft 4.0**) with longer physical headers, RTS/CTS enabled, and different traffic loads per class, while `ns-3.48` uses the finished IEEE 802.11 standard with constant `512 B` UDP streams. Experiment 2A proves that turning on RTS/CTS directly explains why the 2003 paper had a lower total throughput cap.
3. **Random Variance (Confidence Intervals):**
   * Across all 10 random seeds, the error bars for Voice and Video are very small ($\pm 0.08$ to $\pm 0.22\text{ Mbps}$ for throughput, and $\pm 2$ to $\pm 5\text{ ms}$ for delay), showing that 10 runs are plenty to get solid averages. The large error bars on `AC_BK` delay happen because almost all background packets get dropped, so only a few lucky packets make it through.

---

## 5. Conclusion and Link to My Final Project (DACW)

Overall, my experiments prove that **IEEE 802.11e EDCA** works well at protecting Voice and Video throughput and keeping Voice delay around **$30\text{ ms}$** even when 16 stations are competing at once. However, my graphs also show two clear flaws with fixed EDCA numbers:
1. `Best Effort` and `Background` traffic get **completely starved** once 6 or more stations are active.
2. Even Voice delay still climbs from **$1.7\text{ ms}$** (at $N=3$) up to **$\sim 30\text{ ms}$** (at $N=16$) because all 16 stations use the same small contention window ($CW_{\min} = 3$) and collide with each other.

This directly sets up my final project, **DACW (*Density-Aware Contention Window*)**, which will automatically adjust $CW_{\min}$ based on how many stations are active—reducing Voice collisions when the network is crowded while still letting some `Best Effort` traffic through.
