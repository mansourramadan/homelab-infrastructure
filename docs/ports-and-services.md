# Ports and Services Lab

## Objective

Learn how Linux exposes network services through TCP/UDP ports and use
`ss` and Nmap to identify listening services on my Ubuntu homelab.

## Environment

- Ubuntu 24.04.5 LTS
- Apple iMac14,4
- Linux kernel 6.14
- Local IP: 10.0.0.x
- Nmap 7.94SVN

## Tools Used

- `ss`
- `nmap`

## Listening Services

I used:

```bash
sudo ss -tulpn
