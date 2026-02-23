## Looking Back (with Polar Plot – Jan 22, 2026)


![01-22-2026](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202603/2026-01-23%20081202.png)


![01-22-2026 polar](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202603/2026-01-23%20081202-2.png)

Before talking about the V4, let’s review the previous configuration.
(See the polar plot image from 01-22-2026.)


### Measured Results (Early Configuration)



There was a brief issue caused by a damaged SMA cable 
(possibly due to UV exposure — the bundled cable degraded quickly).

After replacing the cable, performance slightly improved.



Statistics:

- Aircraft seen: 674  
- Max range: 85 nm  
- Gain: 38 dB  
- Antenna height: FL + 70 inches (1788 mm)

My ranking reached 39,324 globally.

Performance was stable. I could have stopped there.

At this point, I started thinking about a hardware upgrade.



### Current Dongle (Clone RTL2832 SDR)
![clone](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202602/2026-02-20%20221135.png)
- Aluminum enclosure  
- 25 MHz – 1760 MHz coverage  
- VHF/UHF support  
- AM / NFM / FM / DSB / USB / LSB / CW  
- ±0.5 PPM TCXO

To decide objectively, I compared both models.


| Feature               | Nooelec v5             | RTL-SDR Blog V4          |
| --------------------- | ---------------------- | ------------------------ |
| ADS-B (1090 MHz)      | Excellent              | Good                     |
| VHF Airband           | Excellent              | Good                     |
| HF (Shortwave)        | △ Requires upconverter | ◎ Direct samplding      |
| Noise resistance      | Excellent: Low noise   | Good: sensitive          |
| Heat                  | Low                    | Moderate                 |
| 24/7 Raspberry Pi use | Excellent              | Good: Possible, needs tuning |


I chose the RTL-SDR Blog V4 for its versatility.

HF direct sampling was especially attractive.

Click.


In the next post, I will show what happened after swapping the dongle.
Unexpected results followed.


What happened next was unexpected.