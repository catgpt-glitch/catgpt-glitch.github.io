## 🚀 What Is the International Space Station (ISS)? – Part 2


![sstv](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202604/sstv.png)


## 🛰️ Visibility of the ISS


Under the right conditions, the International Space Station (ISS) can be seen with the naked eye.

At its brightest, it can reach about magnitude −2, comparable to Jupiter.

It is typically visible for a short period:

**within about two hours after sunset or before sunrise**

This is when:

the sky is dark enough, and
the ISS is still illuminated by sunlight and reflects it toward Earth


---


## 🔭 Observation Passes (Predicted Data)


The timing and direction of ISS visibility and radio communication vary depending on your location.

A pass refers to the period when the ISS is above the horizon and:

radio communication is possible, and
sometimes it is also visible

Key terms:

AOS (Acquisition of Signal): the moment the ISS rises above the horizon
LOS (Loss of Signal): the moment it sets below the horizon
TCA (Time of Closest Approach): when the ISS is closest to the observer

One of the most important factors is the maximum elevation angle.

**Higher elevation = better chances for both:**

radio reception (VHF/UHF line-of-sight)
visual observation

To predict passes, you can use:

smartphone satellite tracking apps 📱
websites such as N2YO.com

I also plan to add pass prediction features to my own ISS tracking tool.


---


## 👩‍🚀 Internal Factors (Crew Activity)


Another critical factor is the daily schedule of the ISS crew.

Some systems, such as:

repeaters
SSTV (image transmission)

may operate automatically and can sometimes be received continuously.

However:

**Voice communication on 145.800 MHz FM is usually manual**

This means:

**If the crew is asleep, there will be no voice transmission.**

The ISS operates on Coordinated Universal Time (UTC).

Typical schedule:

Wake-up: around 06:00 UTC
Daily planning conference: 07:30 UTC
Work period: 08:00–12:00 UTC
Afternoon: continued work, maintenance, and exercise
Sleep: around 21:30 UTC

Their daily routine is managed in 5-minute increments.

## 📡 Radio Operations on the ISS

As mentioned above, the ISS does not continuously transmit voice signals.

SSTV transmissions occur only during special events
The FM crossband repeater is activated intermittently

There is no fully public, fixed schedule because operations may change due to:

spacewalks (EVAs)
maintenance
mission priorities

Therefore, it is important to check:

**ARISS (Amateur Radio on the International Space Station)**


**X (formerly Twitter)**

for the latest updates.


---


## 📻 Equipment Used on the ISS


The ISS is equipped with amateur radio systems such as:

**KENWOOD TM-D710GA**


![tm-d710ga](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202604/tm-d710ga.png)


These radios are used for:

direct voice communication
school contacts (scheduled communication with students)
communication with amateur radio operators worldwide

Typical transmit power:

**10W to 25W**

Main signals of interest:

**Voice (direct): 145.800 MHz FM 👈**


**SSTV (image transmission): 145.800 MHz FM 👈**


**👉 These are the signals I will be targeting.**

🐈 Summary

The ISS orbits the Earth about 16 times per day,
so reception depends on catching the right pass.

For reception, the following are recommended:

handheld or mobile radios
directional antennas

Indoor antennas may make reception significantly more difficult.


---


## 🛠️ Next Step


To improve my chances, I decided to build a tool for Raspberry Pi that will:

automatically generate ISS pass schedules
track Doppler shift
record received audio


👉 This tool will be the core of my experiment.


[← Back to Home](/)
