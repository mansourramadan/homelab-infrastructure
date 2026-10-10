# Network Basics Lab 

## Objective 

Pracrice fundemental Linux networking commands and learn how my Ubuntu homelab communicates with the local network and the Internet. 

## Enviroment 

- Ubuntu 24.04.5 LTS
- Apple iMac14,4
- Linux kernel 6.14.0-33-generic
- Wi-Fi interface: wlp2s0
- Local network: 10.0.0.0/24
- Default gateway 10.0.0.1

## Commands Practiced 

### System Information 

```bash
hostnamectl
uptime
lscpu
free -h 
lsblk
df -h /

## Troubleshooting Notes

During the lab, the standard traceroute command did not initially reach the final GitHub destination and several hops displayed `* * *`.

I learned that this does not automatically mean the network connection is broken. Some routers and firewalls do not respond to the UDP probes normally used by traceroute.

I tested an alternative using ICMP:

```bash
sudo traceroute -I github.com
