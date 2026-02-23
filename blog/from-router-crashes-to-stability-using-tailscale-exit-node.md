### The Problem


My home router started crashing frequently after running FR24 Feed.


- 6–12 crashes per day
- Netflix interruptions (family complaints 😅)
- Raspberry Pi running FR24 + SDR workload


Environment:
- MAP-E (IPv4 over IPv6)
- CGNAT-like behavior
- Frequent keepalive / UDP traffic


### Why It Happens


In MAP-E / DS-Lite / CGNAT environments:


- NAT tables fill quickly
- UDP keepalive increases session load
- Retransmissions multiply traffic
- Router CPU usage spikes



### The Solution: Tailscale Exit Node on VPS



Instead of using a commercial VPN, I used my own VPS as an Exit Node.


Step 1 – Enable forwarding on VPS:


```batch
sudo nano /etc/sysctl.d/99-tailscale.conf
net.ipv6.conf.all.forwarding=1
net.ipv4.ip_forward=1
sudo sysctl --system
```


Step 2 – Advertise Exit Node:


```batch
sudo tailscale up --advertise-exit-node --accept-dns=false --ssh
```


Step 3 – Raspberry Pi:


```batch
sudo tailscale up --exit-node=<VPS-name> --accept-dns=false --ssh
```


### Verification


```batch
curl -4 ifconfig.me
```


Returned:
160.xxx.xxx.xxx (VPS IP)
This confirmed that outbound traffic was routed via the VPS.


### Results


Before:
- 6–12 router crashes per day


After:
- ~0.5 crashes per day (almost stable)
- No more Netflix interruptions
- FR24 Feed stable


### Important Note


This may help if you use:


- MAP-E
- DS-Lite
- CGNAT
- Starlink
- Mobile broadband


If your router struggles with frequent UDP sessions, moving traffic through a stable Exit Node can significantly reduce load.


I did not keep exact logs, but based on router resets and family complaints, the crash frequency was approximately 6–12 times per day.