# Assignment A4 — Reproducing and Analyzing IEEE 802.11e EDCA (Mangold et al., 2003) in ns-3.48

**Reference Paper:** S. Mangold, S. Choi, G. R. Hiertz, O. Klein, and B. Walke, *"Analysis of IEEE 802.11e for QoS Support in Wireless LANs"*, IEEE Wireless Communications, vol. 10, no. 6, pp. 40–50, Dec. 2003.

## 1. Goal and Experiment Plan

The goal of this assignment is to reproduce a key baseline result from Mangold et al. (2003) using `ns-3.48`. In their paper, the authors evaluated how **IEEE 802.11e EDCA** (*Enhanced Distributed Channel Access*) prioritizes time-sensitive traffic (Voice and Video) over standard data traffic (Best Effort and Background) as more stations contend for the wireless channel ($N = 1$ to $16$).

Because `ns-3.48` implements the finalized IEEE 802.11 standard rather than the 2003 802.11e Draft 4.0 used in the paper, exact numerical matches are not expected. To reproduce the main trend and verify QoS behavior under congestion, I ran **two experiments**:

1. **Experiment 1 (Baseline Throughput vs. Station Density):** I recreated the single-hop 802.11a/e topology without RTS/CTS and measured the aggregate throughput (in Mbps) of each Access Category (`AC_VO`, `AC_VI`, `AC_BE`, `AC_BK`) from $N = 1$ to $16$ stations, plotting my curves directly against Mangold et al.'s original results.

2. **Experiment 2 (End-to-End Access Delay under Congestion):** To verify that Voice and Video maintain low latency even when the channel saturates, I enabled RTS/CTS (`RtsCtsThreshold = 0`) and measured the mean end-to-end packet delay (in milliseconds, on a logarithmic scale) using `FlowMonitor` across the same station densities ($N = 1$ to $16$).

## 2. Simulation Setup and Reproducibility Parameters

To ensure statistical validity and eliminate single-seed bias, every station density ($N = 1$ to $16$) was simulated across **10 independent random runs** (`160 runs` for Experiment 1 and `160 runs` for Experiment 2, totaling **320 simulation runs**). All plotted points represent the sample mean with **95% confidence intervals** ($t_{0.975, 9} \cdot s / \sqrt{10}$).

### General Simulation Parameters

| Parameter | `ns-3.48` Configuration | Rationale | 
| ----- | ----- | ----- | 
| **Simulator & OS** | `ns-3.48` (macOS) | Configured using current `ns-3.48` attributes (`FrameRetryLimit`, `WifiMacQueue::MaxDelay`) | 
| **PHY / MAC Standard** | IEEE 802.11a (5 GHz) with QoS enabled | Matches the 5 GHz OFDM physical layer studied by Mangold et al. | 
| **Data & Control Rates** | Data: `24 Mbps` / Control: `6 Mbps` | Fixed via `ConstantRateWifiManager` to prevent dynamic rate adaptation from skewing saturation behavior | 
| **Frame Aggregation** | Disabled (`MaxAmpduSize = 0`, `MaxAmsduSize = 0`) | Reflects the 2003 802.11e draft, prior to 802.11n frame aggregation | 
| **Retry Limit** | `FrameRetryLimit = 4` | Prevents head-of-line blocking from repeatedly colliding frames | 
| **Topology** | 1 AP + $N \in [1, 16]$ STAs ($1\text{ m}$ spacing) | Eliminates hidden-node effects and path-loss drops so contention is the sole bottleneck | 
| **Offered Traffic** | 4 UDP CBR flows per STA (`1 Mbps` each, `512 B` payload) | Each station generates $4\text{ Mbps}$ total ($1\text{ Mbps}$ per Access Category) | 
| **Simulation Duration** | $10.0\text{ s}$ total ($1.0\text{ s}$ warm-up + $9.0\text{ s}$ measurement) | Discards initial ARP/association transients before recording throughput and delay | 
| **PRNG Seed & Runs** | `RngSeedManager::SetSeed(1)`, `SetRun(1..10)` | Provides 10 independent substreams per $N$ for exact repeatability | 

### EDCA Access Category Parameters

| Access Category | IP TOS | `AIFSN` | $CW_{\min}$ | $CW_{\max}$ | Queue `MaxDelay` | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| **Voice (`AC_VO`)** | `0xC0` | 2 | 3 | 7 | $100\text{ ms}$ | 
| **Video (`AC_VI`)** | `0x80` | 2 | 7 | 15 | $100\text{ ms}$ | 
| **Best Effort (`AC_BE`)** | `0x00` | 3 | 15 | 1023 | $500\text{ ms}$ (default) | 
| **Background (`AC_BK`)** | `0x20` | 7 | 15 | 1023 | $500\text{ ms}$ (default) | 

## 3. Results and Analysis

### 3.1 Experiment 1: Aggregate Throughput vs. Number of Stations

<img width="1167" height="740" alt="image" src="https://github.com/user-attachments/assets/14687d0e-44bf-4f8f-991c-b233ad941b6c" />

Overlaying the `ns-3.48` results (solid lines with 95% CI bars) onto Mangold et al.'s original curves (dashed lines) reveals three distinct operating regimes:

1. **Under-Saturated Regime ($N = 1$ to $3$):** At $N = 1$ and $N = 2$, channel capacity is sufficient for all four classes to achieve their full offered load ($1.0\text{ Mbps}$ and $2.0\text{ Mbps}$ per class). At $N = 3$ ($12\text{ Mbps}$ total offered load), `AC_BK` is the first to yield, dropping to $0.78\text{ Mbps}$ due to its longer arbitration inter-frame space (`AIFSN = 7`), which matches the paper's behavior.

2. **Saturation Onset ($N = 4$ to $6$):** At $N = 4$ ($16\text{ Mbps}$ offered load), the channel saturates. `AC_BE` drops sharply to $1.29\text{ Mbps}$ and `AC_BK` falls to $0.13\text{ Mbps}$, while `AC_VO` ($3.85\text{ Mbps}$) and `AC_VI` ($3.84\text{ Mbps}$) continue scaling nearly linearly. Between $N = 4$ and $N = 6$, `AC_VI` slightly edges out `AC_VO` ($5.35\text{ Mbps}$ vs. $5.15\text{ Mbps}$ at $N = 6$) because both share `AIFSN = 2`, and `AC_VI`'s slightly larger contention window ($CW_{\min}=7$ vs. $3$) occasionally avoids collisions while still winning channel access ahead of `AC_BE`.

3. **Heavy Contention & Starvation ($N = 7$ to $16$):** Beyond $N = 6$, `AC_BK` ($0.00\text{ Mbps}$) and `AC_BE` ($< 0.03\text{ Mbps}$) are completely starved, closely matching Mangold et al.'s curves. `AC_VO` overtakes `AC_VI` at $N = 7$ and stabilizes at $5.90 \pm 0.10\text{ Mbps}$, while `AC_VI` stabilizes at $5.20 \pm 0.15\text{ Mbps}$.

### 3.2 Experiment 2: End-to-End Access Delay

<img width="3000" height="1800" alt="image" src="https://github.com/user-attachments/assets/e3a4d3dc-1660-48c4-9640-966a6e937313" />

Plotting mean packet delay on a logarithmic scale (with RTS/CTS enabled, `RtsCtsThreshold = 0`) highlights the QoS separation under congestion:

* **Low-Priority Delay Explosion & Queue Capping:** `AC_BK` jumps from $1.8\text{ ms}$ ($N=1$) to $71\text{ ms}$ ($N=2$) and hits $\approx 490\text{ ms}$ by $N=3$. `AC_BE` follows one step later, jumping from $3.6\text{ ms}$ ($N=2$) to $\approx 480\text{ ms}$ ($N=3$). For $N \ge 3$, both classes hover between $200\text{ ms}$ and $500\text{ ms}$ with wide confidence intervals (especially for `AC_BK`) because `FlowMonitor` only averages *delivered* packets—any packet waiting longer than `MaxDelay = 500 ms` is dropped from the MAC queue, artificially capping the measured delay of the few packets that sneak through.

* **Bounded Delay for Voice and Video:** Even at $N = 16$ stations, `AC_VO` maintains a mean delay of $\approx 28\text{–}33\text{ ms}$ and `AC_VI` stays around $\approx 43\text{–}47\text{ ms}$, well below their $100\text{ ms}$ queue timeout and acceptable for real-time audio and video.

## 4. Explaining the Differences from Mangold et al. (2003)

While the qualitative behavior—saturation at $N=4$, starvation of `AC_BE`/`AC_BK`, and protection of `AC_VO`/`AC_VI`—matches the 2003 paper closely, three specific modeling differences explain the numerical gap between the solid and dashed curves in Figure 1:

1. **Traffic Model Mismatch for `AC_VI` ($\approx 5.2\text{ Mbps}$ vs. $\approx 3.0\text{ Mbps}$):** In my simulation, every station generated identical `1 Mbps` constant-bit-rate (CBR) UDP traffic for all four classes, allowing `AC_VI` to scale up to $16\text{ Mbps}$ of offered load and claim all airtime left over by `AC_VO`. In Mangold et al. (2003), Video was modeled with a lower/bounded offered load profile (or bursty trace capping around $3\text{ Mbps}$ aggregate), which left a small amount of residual capacity for `AC_BE` between $N=4$ and $N=10$ (seen in the dashed blue curve tapering gradually rather than dropping immediately to zero).

2. **MAC Control Overhead & PHY Timing (`11.1 Mbps` vs. `8.4 Mbps` total):** Experiment 1 disabled RTS/CTS (`RtsCtsThreshold` left at default), whereas the 2003 study's baseline included higher MAC/PHY framing overhead (such as RTS/CTS control handshakes at the $6\text{ Mbps}$ basic rate and Draft 4.0 post-backoff rules), reducing total saturation throughput by roughly $2.7\text{ Mbps}$.

3. **Draft 4.0 vs. Final IEEE 802.11e Standard:** Mangold et al. simulated an early 2003 draft of 802.11e (prior to final ratification in 2005), where contention window reset rules, TXOP burst behavior, and internal virtual collision handling between queues inside the same station differed slightly from `ns-3.48`'s finalized `QosTxop` implementation.

## 5. How to Reproduce These Results

To repeat Experiment 1 and Experiment 2 in `ns-3.48`, run the simulation script across $N \in \{1..16\}$ and `run` $\in \{1..10\}$:

```bash
# Experiment 1: Baseline EDCA throughput without RTS/CTS
for n in $(seq 1 16); do
  for run in $(seq 1 10); do
    ./ns3 run "a4-edca-reproduction --nStations=$n --rtsCts=false --RngRun=$run"
  done
done

# Experiment 2: EDCA end-to-end access delay with RTS/CTS enabled
for n in $(seq 1 16); do
  for run in $(seq 1 10); do
    ./ns3 run "a4-edca-reproduction --nStations=$n --rtsCts=true --RngRun=$run"
  done
done
```

