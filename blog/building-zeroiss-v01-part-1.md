## Building ZeroISS v0.1


![iss004](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202606/iss004.png)


I recently started learning programming and Raspberry Pi (Linux).

This time, I created this tool with help from ChatGPT.

By the way, the GitHub name `catgpt-glitch` was actually a typo.  
Originally, it was supposed to be `catudp-glitch`.

However, my partner ChatGPT said:

> “This name (`catgpt`) sounds cooler 🐈️”

So I decided not to fix it.

---

The goals for ZeroISS were:

- A simple and lightweight tool that can run even on a Raspberry Pi Zero
- Written in Python
- Headless SDR operation (controlled remotely from another terminal)
- Automatic ISS pass scheduling using `cron`
- Doppler-shift-aware recording using `rtl_fm`

To begin with, I decided to create the following files:

```
zeroiss.py   (main Python application)
config.ini   (configuration file)
record.sh    (recording script)
schedule.csv (pass schedule file)
```

Originally, I planned to stream and record the `rtl_fm` audio from a web page or VLC on another device.

However, for simplicity and usability, I changed the design so everything runs directly on the Raspberry Pi Zero itself.

In other words:

👉 Recorded audio files are stored locally on the Raspberry Pi Zero as WAV files.

Real-time monitoring is still useful for manual tuning, so I kept the web audio streaming feature.

---

## ZeroISS Architecture

### A. Orbit Calculation

Responsible for ISS tracking calculations:

- TLE retrieval
- Pass prediction
- Doppler shift calculation

---

### B. Receiver

Responsible for audio and recording:

- `rtl_fm`
- ALSA or `ffmpeg`

HTTP monitoring stream:

- `nc`
- `cvlc`

---

### C. Controller

Responsible for scheduling and automation:

- `cron`
- `systemd timer`
- Python loop processing

---

For v0.1, I decided to simplify the orbit calculation system.

The first priority was making the Receiver and Controller sections work reliably.

That means the initial version uses manually entered pass times and frequencies in `schedule.csv`.

Example:

```
unix_time,freq_hz
1772948100,145803000
1772948200,145800000
1772948300,145797000
1772948400,145796000
```

The first frequency switch worked correctly:

```
1772948100,145803000
```

However, when the next process started:

```
1772948200,145800000
```

the previous `rtl_fm` process had not completely terminated yet.

This caused:

```
usb_claim_interface error -6
```

To solve this, I created a dedicated kill script to fully terminate `rtl_fm` before restarting it.

---

The next problem was that restarting `rtl_fm` every time also created multiple WAV files.

To fix this, I changed the structure to a `tee / FIFO` pipeline system:

```
rtl_fm
   ↓
FIFO (pipe)
   ↓
ffmpeg (recording)
```

Now only `rtl_fm` restarts during frequency changes.

This allowed the recording process itself to remain active while only switching the receive frequency.

As a result:

👉 The recording stays as a single WAV file.

---

![code2](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202606/code2.png)


At the time, I did not fully understand Linux process management or pipe handling, so reaching this point took quite a bit of trial and error.

Next time, I will show the actual code structure and execution results.



[← Back to Home](/)

