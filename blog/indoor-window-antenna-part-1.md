I started this project with a Raspberry Pi Zero 2 W and a cheap clone RTL-SDR dongle.

My goal was simple:
to see how far I could receive ADS-B aircraft signals
using a small indoor antenna.

For the antenna, I initially used a simple VHF/UHF window antenna attached indoors.
Later, I switched to a V-shaped dipole for ADS-B reception.


**This was the first antenna I tried.**


![window antenna](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202602/2025-12-16%20192545.jpg)


At that time, I did not fully understand how antenna placement affects performance.
I assumed any antenna near a window would work well.


**V-shaped dipole Antenna**


![dipole2](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202602/2026-02-20%20221047.png)


**Here is how it was installed.**


![diagram](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202602/diagram.png)


**Raspberry Pi Zero 2 W**

The Raspberry Pi Zero 2 W is a compact single-board computer
measuring only 65 mm × 30 mm (2.6inch × 1.18inch).

Despite its small size, it features a quad-core CPU (RP3A0),
providing roughly five times the performance of the original Raspberry Pi Zero.

Specifications include:

• 512 MB RAM  
• Built-in Wi-Fi and Bluetooth 4.2  
• microSD storage  
• mini HDMI output  
• micro-USB ports  
• CSI-2 camera connector  
• 40-pin GPIO header

Operating System:
Raspberry Pi OS (Bookworm, 64-bit Lite – headless configuration)


Applications:


• dump1090 – ADS-B signal decoder  
• fr24feed – FlightRadar24 feeder software  
• Tailscale – VPN for remote access and operation


![rpizero2w](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202602/rpizero2w.jpg)


[← Back to Home](/)
