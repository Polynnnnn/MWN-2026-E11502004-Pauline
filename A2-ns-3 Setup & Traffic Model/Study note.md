# Study Note: ns-3 Simulator & 3GPP XR Traffic Model

## 1. Installation and Build

- **ns-3 Version:** `ns-3.48`
- **Platform:** macOS 

### Install Steps

1. Downloaded the official source code archive (`ns-allinone-3.48.tar.bz2`).
2. Extracted the files and opened the terminal in the `ns-3.48` folder.
3. Configured the build using:
   ```bash
   ./ns3 configure --enable-examples
   ```
4. Compiled the simulator using:
   ```bash
   ./ns3 build
   ```

---

### What Broke 

*   **Homebrew Issue:** 
    First, I tried installing ns-3 using Homebrew. However, this version did not include the `scratch` folder, which is required to write and run custom simulation scripts. 
    * **Fix:** I had to delete it and download the full source code from the official website.

*   **Compilation Error:** 
    During my first code run, I got the following compilation error:
    > `no member named 'Install' in 'ns3::Ipv4AddressHelper'`
    * **Fix:** In version `3.48`, the `Install()` method for IP addresses was removed. I fixed this by updating the C++ code to use the `Assign()` method instead.

 ---

## 2. 3GPP Traffic Model (3GPP TS 26.926)

The **3GPP TR 26.926** specification defines how network traffic behaves for Extended Reality (XR) and Virtual Reality (VR) applications. This traffic has two main rules:

* **Constant Inter-Arrival Time:** The headset needs a steady frame rate (e.g., 60 FPS), which means a new frame is generated exactly every **16 milliseconds**.
* **Variable Frame Size:** Because some video frames are more complex to render than others, the size of each frame changes randomly following a **Log-Normal distribution**.

### Script Implementation

I used AI to help me write a custom ns-3 script to simulate this behavior. The script works by:

1. Using the `LogNormalRandomVariable` object in ns-3 to calculate the packet sizes.
2. Fragmenting the packets if they are too big for a single transmission.
3. Scheduling the UDP transmission exactly every **16 ms** to match the 3GPP model.

---

## 3. The Code and the Command

* **The Source Code:** My code file is named `xr-traffic-sim.cc`. 
* **The Run Command:** To run my own custom simulation, I typed this into my terminal:

  ```bash
  ./ns3 run scratch/xr-traffic-sim
  ```

---

## 4. Proof of My Simulation

### Wireshark Capture 
The Wireshark image below shows my `.pcap` file. You can clearly see that:
* My starting computer (IP address `10.1.1.1`) successfully sent data packets (UDP protocol).
* My receiving computer (IP address `10.1.1.2`) successfully received them. 
* **Conclusion:** The connection worked perfectly.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/04742ee0-ee8d-4113-ad57-d43063f0525f" />

### The Graphs (Math Proof)
I also asked the AI to write a short Python script. This script analyzed all the results from my simulation to draw these graphs:

* **Top graphs (Packet size):** You can see an asymmetrical bell-shaped curve (blue and red). This proves that my code correctly applies the size variations required by the 3GPP standard (the **Log-Normal curve**).
* **Bottom graphs (The timing):** You can see a single, straight vertical line (green and purple) planted exactly at zero (which corresponds to **16 ms**). This proves there is no delay: the images are sent with absolute precision to guarantee a steady **60 frames per second (FPS)**.

