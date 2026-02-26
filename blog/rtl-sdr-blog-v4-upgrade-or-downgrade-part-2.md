**I installed the RTL-SDR Blog V4 on January 30.  
The following morning (January 31), I compared the results against the baseline from January 28.  
For this setup, I chose the USB Type-C version.**

![RTL-SDL-V4](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202603/rtlsdrv4.png)

One important note: **the V4 requires the official V4-compatible driver** from the RTL-SDR Blog website.[https://www.rtl-sdr.com/V4/](https://www.rtl-sdr.com/V4/)


```
Updating the Drivers on the FlightRadar24, FlightAware and OpenWebRX Raspberry Pi4 Images

Set up your FlightRadar24 image as normal, then either log into SSH or attach a monitor. 
The FlightRadar24 image has default credentials pi/raspberry, pi/flightware for FlightAware and pi/adsb123 for ADSBExchange. Not on the FlightAware to enable SSH, you need to create a file with filename "ssh" on the boot folder of the SDcard.

Once logged into the terminal, run the following commands to update the librtlsdr packages.
```

```bash
sudo apt update
sudo apt install libusb-1.0-0-dev git cmake build-essential
sudo apt install debhelper

git clone https://github.com/osmocom/rtl-sdr
cd rtl-sdr
sudo dpkg-buildpackage -b --no-sign
cd ..

sudo dpkg -i librtlsdr0_*
sudo dpkg -i librtlsdr-dev_*
sudo dpkg -i rtl-sdr_*

sudo reboot
```
The guide says “Now reboot your Pi4 and it should work.”  
Even though it mentions Raspberry Pi 4, **it also worked on my Raspberry Pi Zero 2 W**.


After installing the driver, I searched for the best gain setting.  
I tested from **38 dB up to 49 dB**, observing a few minutes at each step.  
I ended up using **48 dB**.

However, compared to my previous clone dongle, the V4 didn’t perform as I expected.  


### Short comparison (same time of day)


To reduce time-of-day bias, I compared the morning window on Jan 31 with the same morning window on Jan 28.


The difference becomes clearer when viewed visually.
**Example (simplified):**


![Jan28vsJan31](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202603/28vs31.png)


Same morning window (UTC 00:00–06:00) used for comparison.


Note: Y-axis differs because these are FR24 screenshots.


- Jan 28: HITS 1000 / POSITIONS 500
    
- Jan 31: HITS 400 / POSITIONS 200  
    (Weather conditions were comparable on both days, reducing environmental bias.)
    

On Saturday morning, with many aircraft expected, I decided to revert.  
Because the V4 driver is backward compatible, I **kept the driver installed**, kept **48 dB**, and swapped back to the **clone dongle**, then rebooted.

I was in a hurry—thinking “I’ll fine-tune it later.”  
But when I checked the PiAware map, I paused.

**“Why am I suddenly seeing aircraft over Toyohashi?”** 
That had never happened under the same conditions before. 
(About **200 km / 124 miles** from my location.)

_(Next post: what actually happened, and what I changed after that.)_


[← Back to Home](/)
