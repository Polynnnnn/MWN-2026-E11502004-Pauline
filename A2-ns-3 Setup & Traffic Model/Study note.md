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
