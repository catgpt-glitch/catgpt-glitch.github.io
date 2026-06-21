## Building ZeroISS v0.1 – Part 2

In the previous article, **Building ZeroISS v0.1 – Part 1**, I briefly explained the basic design of the tool and the problems I encountered while building it.

In this part, I will show the actual file structure, the code, and how the tool works.

First, here are the completed files.

---

### Completed Files

```
zeroiss.py   (main Python application)
config.ini   (configuration file)
kill_sdr.sh  (kill script)
tune.sh      (rtl_fm tuning script)
record.sh    (recording script)
schedule.csv (pass schedule file)
zeroiss.log  (log file)
```

I am still a beginner in both Python programming and Linux.

If you are also a beginner, please do not think too hard about it.  
The basic idea is actually simple, and I will try to explain it as clearly as possible.

Python runs on Windows and macOS as well, but I chose a small computer called the Raspberry Pi because it can run 24 hours a day with very low power consumption.

The operating system I use is Raspberry Pi OS, which is a Linux distribution.

A Linux distribution is, simply put, a packaged version of Linux.

Windows is usually distributed as one standard product, but Linux is different.  
Many organizations use the same Linux kernel and then package it with their own desktop environments, applications, and tools.

Using Linux, Raspberry Pi, and an SDR USB receiver, I built a system to record ISS radio signals.

I wrote more about SDR in my FlightRadar24 series.

The basic flow is:

- Use Linux scheduling with `cron`
- Start my Python program, `zeroiss.py`
- Receive radio signals with an SDR dongle
- Save the audio as WAV files inside the Raspberry Pi
- Change the receive frequency according to `schedule.csv`
- Follow the Doppler shift during the ISS pass

The main problem I encountered was that the audio file was not created correctly when switching frequencies.

Before explaining each part in detail, please take a look at the full code.

It is okay if you do not understand everything yet.

This tool uses not only Python, but also Linux shell scripts (`.sh` files).

At first glance, it may look complicated.

However, there is a reason why the code is divided into several files.

In the next parts, I will explain:

- why the code is separated
- what shell scripts are
- how each part works

---

### Code Structure

#### zeroiss.py

```
#!/usr/bin/env python3

# 1. Import modules
import configparser
import csv
import os
import signal
import subprocess
import sys
import time
from pathlib import Path
from datetime import datetime

BASE_DIR = Path.home() / "ZeroISS"
CONFIG_FILE = BASE_DIR / "config.ini"
SCHEDULE_FILE = BASE_DIR / "schedule.csv"

rtl_process = None

# 2. Define functions

# Load config.ini
def load_config():
    cfg = configparser.ConfigParser()
    cfg.read(CONFIG_FILE)
    return cfg

# Simple interactive prompt
def ask(prompt, default=""):
    val = input(f"{prompt} [{default}] : ").strip()
    return default if val == "" else val

# Run a Linux shell command
def sh(cmd):
    return subprocess.run(
        cmd, shell=True, text=True,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT
    ).stdout.strip()

# Stop rtl_fm process safely
def stop_rtl():
    global rtl_process
    if rtl_process and rtl_process.poll() is None:
        os.killpg(os.getpgid(rtl_process.pid), signal.SIGTERM)
        try:
            rtl_process.wait(timeout=2)
        except subprocess.TimeoutExpired:
            os.killpg(os.getpgid(rtl_process.pid), signal.SIGKILL)
            rtl_process.wait()
    rtl_process = None

# Live monitor function
def start_monitor(freq, gain, ppm):
    global rtl_process
    stop_rtl()

    cmd = (
        f"/usr/local/bin/rtl_fm -f {freq} -M fm -s 48k -r 48k -E deemp "
        f"-g {gain} -p {ppm} - | "
        f"cvlc - --demux=rawaud "
        f"--rawaud-channels 1 "
        f"--rawaud-samplerate 48000 "
        f"--sout '#transcode{{acodec=mp3,ab=64,channels=1,samplerate=48000}}:"
        f"http{{mux=mp3,dst=:4687/aaa.mp3}}' "
        f"vlc://quit"
    )

    rtl_process = subprocess.Popen(
        cmd,
        shell=True,
        preexec_fn=os.setsid
    )
    print(f"[MONITOR] started at {freq} Hz")
    print("Stream: http://<ZeroISS-IP>:4687/aaa.mp3")

# Load schedule.csv
def load_schedule():
    rows = []
    with open(SCHEDULE_FILE, newline="") as f:
        reader = csv.reader(f)
        for row in reader:
            if not row:
                continue
            first = row[0].strip()
            if first.startswith("#"):
                continue
            try:
                ts = int(first)
                freq = int(row[1].strip())
                rows.append((ts, freq))
            except (ValueError, IndexError):
                continue
    rows.sort()
    return rows

# Record one ISS pass
def record_pass(gain, ppm, rec_dir):
    global rtl_process

    schedule = load_schedule()
    if not schedule:
        print("schedule.csv is empty or invalid")
        return

    rec_dir = Path(rec_dir).expanduser()
    rec_dir.mkdir(parents=True, exist_ok=True)

    out_file = rec_dir / f"ISS_{datetime.now().strftime('%Y%m%d_%H%M%S')}.wav"
    current_freq = None
    record_proc = None

    print(f"[REC] output -> {out_file}")

    try:
        # Start recording only once
        record_proc = subprocess.Popen(
            ["/bin/bash", str(BASE_DIR / "record.sh"), str(out_file)],
            preexec_fn=os.setsid
        )
        time.sleep(1.0)

        while True:
            now = int(time.time())

            next_freq = None
            for ts, freq in schedule:
                if ts <= now:
                    next_freq = freq
                else:
                    break

            if next_freq is None:
                time.sleep(0.5)
                continue

            if now > schedule[-1][0]:
                print("[REC] finished")
                break

            if next_freq != current_freq:
                stop_rtl()

                rtl_process = subprocess.Popen(
                    ["/bin/bash", str(BASE_DIR / "tune.sh"), str(next_freq), str(gain), str(ppm)],
                    preexec_fn=os.setsid
                )

                current_freq = next_freq
                print(f"[REC] tuned -> {next_freq}")

            time.sleep(0.5)

    except KeyboardInterrupt:
        print("\n[REC] interrupted")

    finally:
        stop_rtl()
        subprocess.run([str(BASE_DIR / "kill_sdr.sh")], check=False)

        if record_proc and record_proc.poll() is None:
            os.killpg(os.getpgid(record_proc.pid), signal.SIGTERM)
            try:
                record_proc.wait(timeout=3)
            except subprocess.TimeoutExpired:
                os.killpg(os.getpgid(record_proc.pid), signal.SIGKILL)
                record_proc.wait()

# Apply cron settings
def cron_apply():
    """
    Minimal version for overwriting the ZeroISS cron block.
    """
    python_bin = sys.executable
    zeroiss_py = BASE_DIR / "zeroiss.py"

    marker_start = "# >>> ZeroISS >>>"
    marker_end = "# <<< ZeroISS <<<"

    block = f"""{marker_start}
39 21 12 3 * {python_bin} {zeroiss_py} --record >> {BASE_DIR}/zeroiss.log 2>&1
{marker_end}
"""

    current = sh("crontab -l 2>/dev/null")
    lines = current.splitlines()

    new_lines = []
    inside = False
    for line in lines:
        if line.strip() == marker_start:
            inside = True
            continue
        if line.strip() == marker_end:
            inside = False
            continue
        if not inside:
            new_lines.append(line)

    new_lines.append(block.strip())
    tmp = BASE_DIR / ".cron.tmp"
    tmp.write_text("\n".join(new_lines) + "\n")

    os.system(f"crontab {tmp}")
    tmp.unlink(missing_ok=True)
    print("[CRON] applied")

# Pass calculation placeholder
def calc_pass_placeholder():
    """
    Not implemented yet.

    For now, this tool assumes that schedule.csv is created manually.
    In the future, I plan to add TLE-based pass calculation here.
    """
    print("[CALC] placeholder")
    print("Next step: auto-generate schedule.csv from ISS TLE")

# Menu
def menu():
    print("""
===== ZeroISS =====
1) ISS pass Calculation🚀
2) cron Schedule (Overwrite)
3) Live Monitor
4) Automatic Recording Test
Q) Quit
""")


def main():
    cfg = load_config()

    gain = cfg.get("radio", "gain", fallback="20")
    ppm = cfg.get("radio", "ppm", fallback="0")
    base_freq = cfg.get("radio", "base_freq", fallback="145800000")
    rec_dir = cfg.get("record", "dir", fallback=str(BASE_DIR / "records"))

    if len(sys.argv) > 1:
        if sys.argv[1] == "--calc":
            calc_pass_placeholder()
            return

        elif sys.argv[1] == "--record":
            record_pass(gain, ppm, rec_dir)
            return

    while True:
        menu()
        sel = input("> ").strip().lower()

        if sel == "1":
            calc_pass_placeholder()

        elif sel == "2":
            cron_apply()

        elif sel == "3":
            freq = ask("Monitor freq (Hz)", base_freq)
            start_monitor(freq, gain, ppm)
            input("Press Enter to stop monitor...")
            stop_rtl()

        elif sel == "4":
            record_pass(gain, ppm, rec_dir)

        elif sel == "q":
            stop_rtl()
            break


if __name__ == "__main__":
    main()
```

---

#### config.ini

```
# Band = FM

[station]
lat = 35.68
lon = 139.76
alt_m = 20

[radio]
ppm = 0
gain = 30
base_freq = 145800000

[record]
dir = ~/ZeroISS/records
pre_start_sec = 60
post_end_sec = 20

[scheduler]
enabled = true
```

---

#### kill_sdr.sh

```
#!/usr/bin/env bash
pkill -TERM rtl_fm 2>/dev/null || true
pkill -TERM rtl_tcp 2>/dev/null || true
sleep 1
pkill -KILL rtl_fm 2>/dev/null || true
pkill -KILL rtl_tcp 2>/dev/null || true
```

---

#### tune.sh

```
#!/usr/bin/env bash
set -euo pipefail

BASE_DIR="$(cd "$(dirname "$0")" && pwd)"
FREQ="$1"
GAIN="$2"
PPM="$3"

/bin/bash "$BASE_DIR/kill_sdr.sh"

exec /usr/local/bin/rtl_fm \
  -f "$FREQ" \
  -M fm \
  -s 48000 \
  -r 48000 \
  -E deemp \
  -g "$GAIN" \
  -p "$PPM" \
  - | aplay -q -D plughw:Loopback,0,0 -f S16_LE -r 48000 -c 1
```

---

#### record.sh

```
#!/usr/bin/env bash
set -euo pipefail

OUTFILE="$1"

exec arecord \
  -q \
  -D plughw:Loopback,1,0 \
  -f S16_LE \
  -r 48000 \
  -c 1 \
  "$OUTFILE"
```

---

#### schedule.csv

```
unix_time,freq_hz
1773407640,145804000
1773407700,145803000
1773407760,145802000
1773407820,145801000
1773407880,145800000
1773407940,145799000
1773408000,145798000
1773408060,145797000
1773408120,145796000
#cron 03-13 22:13
#REC 03-13 22:14:00-22:22:00 Test
```

---

In this part, I mainly showed the full structure of ZeroISS v0.1.

At this stage, it may look complicated.

However, each file has a simple role:

- `zeroiss.py` controls the whole process
- `config.ini` stores settings
- `kill_sdr.sh` stops old SDR processes
- `tune.sh` tunes `rtl_fm`
- `record.sh` records audio
- `schedule.csv` defines when to change frequency

The most important idea is this:

👉 The recording process starts once and keeps running.  
👉 Only the receiver process is restarted when the frequency changes.

This makes it possible to create one continuous WAV recording while still following Doppler shift.

In the next part, I will explain each file step by step.



[← Back to Home](/)
