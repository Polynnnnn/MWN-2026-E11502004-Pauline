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
