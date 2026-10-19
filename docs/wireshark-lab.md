# Wireshark Network Analysis Lab

## Overview

This lab documents my introduction to packet capture and network traffic analysis using Wireshark on my Ubuntu homelab server.

The goal of this lab was to learn how network traffic can be captured, filtered, and inspected to better understand how devices and services communicate across a network.

## Environment

- Ubuntu Linux
- Wireshark
- Wi-Fi interface: `wlp2s0`
- Docker
- Pi-hole DNS server
- Tailscale
- GNOME Remote Desktop

## Wireshark Setup

Wireshark was installed on the Ubuntu system and configured so that my user account could capture network traffic without running the entire Wireshark application as root.

The `wireshark` group was added to my user account, and the new group membership was activated.

This allowed Wireshark to access local network interfaces such as:

- `wlp2s0`
- `tailscale0`
- `docker0`
- Docker bridge interfaces
- `any`

The `any` interface was especially useful because it allowed traffic from multiple network interfaces to be observed in the same capture.

## DNS Analysis

One of the first protocols analyzed was DNS.

The following Wireshark display filter was used:

```text
dns
