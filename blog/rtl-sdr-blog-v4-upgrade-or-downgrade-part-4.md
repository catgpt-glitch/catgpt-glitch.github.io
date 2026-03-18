**Final part**


In this series, I explored whether upgrading to the RTL-SDR Blog V4
improves ADS-B reception under real-world conditions.

Based on my observations, I arrived at two working hypotheses.


**Hypothesis 1**


When using the official RTL-SDR Blog V4 driver,
even lower-tier compatible dongles (in my case, an RTL2832-based clone)
may show improved performance compared to
the standard driver installed via apt repositories.


**Hypothesis 2**


As a result of Hypothesis 1,
previously lost weak signals may now be received.

This could explain the appearance of unusual long-distance reception
from the western sector in my location.

This raises the possibility of terrain-related effects,
such as reflection or diffraction caused by Mount Fuji or surrounding mountain ranges.


These hypotheses are not yet confirmed.

I plan to conduct further validation and reproducibility tests,
and will report the results in a future update.


*****


Regarding the FR24 ranking,
the improvement in reception performance
resulted in a noticeable jump in ranking.


![ranking2](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202603/ranking2.png)


For those working with limited equipment,
such as small antennas and low-cost setups,
there is another effective approach:

software-based reliability improvements.


FR24 ranking includes a metric called:

Uptime (% of available time)

In my case, this value is consistently close to 100%.


![ranking1](https://raw.githubusercontent.com/catgpt-glitch/catgpt-blog-assets/main/posts/202603/ranking1.png)


This is achieved using a simple watchdog script
that monitors the receiver status and recovers from stalls.

*Use at your own risk.
*This does not handle hardware failures such as router outages.


```batch
sudo tee /usr/local/bin/adsb-watchdog.sh >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

# ---- configuration ----
STALE_SEC="${STALE_SEC:-660}"      # Detects a “stuck” state in 11 minutes (candidate for reboot)
HARD_SEC="${HARD_SEC:-1800}"       # Reboot candidates in 30 minutes
MAX_REBOOTS="${MAX_REBOOTS:-2}"    # Up to two automatic reboots
STATE_DIR="/var/lib/adsb-watchdog"
STAMP_FILE="$STATE_DIR/last_ok_epoch"
REB_FILE="$STATE_DIR/reboot_count"
LOGTAG="adsb-watchdog"

mkdir -p "$STATE_DIR"

now=$(date +%s)

# Reading the Stats Timestamp from fr24feed-status
ts_line=$(fr24feed-status 2>/dev/null | grep -E '^FR24 Stats Timestamp:' || true)
if [[ -n "$ts_line" ]]; then
  # 例: "FR24 Stats Timestamp: 2026-01-30 05:55:19."
  ts=$(echo "$ts_line" | sed -E 's/^FR24 Stats Timestamp: ([0-9-]+) ([0-9:]+).*/\1 \2/')
  epoch=$(date -d "$ts" +%s 2>/dev/null || echo 0)
else
  epoch=0
fi

# Record it as OK if the epoch has been obtained and updated
if [[ "$epoch" -gt 0 ]]; then
  echo "$epoch" > "$STAMP_FILE"
  exit 0
fi

# If you get to this point and “can't get a timestamp,” it's likely a deadlock.
last_ok=$(cat "$STAMP_FILE" 2>/dev/null || echo 0)
age=$(( now - last_ok ))

# If `last_ok` is missing or 0, try restarting the service first to see what happens
if [[ "$last_ok" -le 0 ]]; then
  logger -t "$LOGTAG" "no last_ok stamp; restarting services"
  systemctl restart dump1090-fa || true
  sleep 5
  systemctl restart fr24feed || true
  exit 0
fi

# Minor issue: Restart the service
if [[ "$age" -ge "$STALE_SEC" && "$age" -lt "$HARD_SEC" ]]; then
  logger -t "$LOGTAG" "stale ${age}s; restarting services"
  systemctl restart dump1090-fa || true
  sleep 5
  systemctl restart fr24feed || true
  exit 0
fi

# Severe: Reboot (subject to a limit on the number of times)
if [[ "$age" -ge "$HARD_SEC" ]]; then
  reboots=$(cat "$REB_FILE" 2>/dev/null || echo 0)
  if [[ "$reboots" -ge "$MAX_REBOOTS" ]]; then
    logger -t "$LOGTAG" "reboot limit reached ($reboots). watchdog will stop trying."
    exit 0
  fi
  reboots=$((reboots + 1))
  echo "$reboots" > "$REB_FILE"
  logger -t "$LOGTAG" "hard stale ${age}s; rebooting ($reboots/$MAX_REBOOTS)"
  /sbin/reboot
fi
EOF

sudo chmod +x /usr/local/bin/adsb-watchdog.sh
```


The script is executed periodically using systemd service and timer.


```batch
#/etc/systemd/system/adsb-watchdog.service

[Unit]
Description=ADSB watchdog (restart/reboot on stall)
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
Environment=STALE_SEC=660
Environment=HARD_SEC=1800
Environment=MAX_REBOOTS=2
ExecStart=/usr/local/bin/adsb-watchdog.sh
```


```batch
#/etc/systemd/system/adsb-watchdog.timer

[Unit]
Description=Run ADSB watchdog every 5 minutes

[Timer]
OnBootSec=3min
OnUnitActiveSec=5min
Persistent=true

[Install]
WantedBy=timers.target
```


```batch
sudo systemctl daemon-reload
sudo systemctl enable --now adsb-watchdog.timer
systemctl list-timers | grep adsb-watchdog
```


It is also recommended to enable FR24 alert emails:

Offline Email: 1 hour ✉️


Reliable operation often matters more than peak performance.


*****


Next, the focus shifts beyond the Earth🌍️

The next series will explore
building a reception system for signals from the International Space Station🚀 (ISS),
including tool development and signal tracking.

Further observations are required to determine the dominant mechanism.


[← Back to Home](/)
