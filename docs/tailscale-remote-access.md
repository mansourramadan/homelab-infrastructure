# Tailscale Remote Access Lab

## Objective

Configure secure remote access to my Ubuntu homelab using Tailscale and SSH.

The goal was to access the Ubuntu iMac from my iPad even when both devices were on different networks.

## Environment

- Ubuntu 24.04.5 LTS
- Apple iMac14,4
- Linux kernel 6.14
- iPad running Termius
- OpenSSH
- Tailscale

## Tailscale Installation

Tailscale was installed on the Ubuntu iMac using the official Linux installation script.

The system initially did not have `curl` installed, so I installed it with:

```bash
sudo apt install curl
```

I then installed Tailscale and authenticated the Ubuntu iMac with my Tailscale account.

## Tailscale Network

After authentication, both devices appeared on the same private Tailscale network:

```text
iPad
  |
  | Tailscale
  |
Ubuntu iMac
```

Each device received a private Tailscale IP address in the `100.x.x.x` range.

The iMac now has two useful addresses:

```text
LAN address       -> 10.0.0.x
Tailscale address -> 100.x.x.x
```

The LAN address is used when communicating directly over my home network.

The Tailscale address allows communication through the private Tailscale network.

## SSH Over Tailscale

I created a second Termius host specifically for Tailscale.

Configuration:

```text
Protocol: SSH
Port: 22
Username: mansour-ramadan
Authentication: Existing ED25519 SSH key
```

I reused my existing `iPad-Homelab` SSH key instead of creating another key.

The SSH connection successfully reached the Ubuntu iMac through its Tailscale address.

## Verifying the Connection

I used:

```bash
echo $SSH_CONNECTION
```

This showed that both the SSH client and server were communicating through their Tailscale `100.x.x.x` addresses.

This confirmed that the SSH session was using Tailscale rather than the normal LAN connection.

## Remote Network Test

For the final test, I disconnected the iPad from my home Wi-Fi and connected it to a cellular hotspot.

The Ubuntu iMac remained connected to the home network.

I then connected using the Tailscale Termius host.

The SSH connection succeeded.

This demonstrated that I can securely access the Ubuntu homelab while the client device is outside my home network.

## Network Path

```text
iPad
  |
  | Cellular hotspot
  |
Internet
  |
  | Encrypted Tailscale tunnel
  |
Ubuntu iMac
  |
  | SSH
  |
Linux shell
```

## Security

This setup does not require exposing SSH port 22 to the public Internet through router port forwarding.

SSH authentication is performed using an ED25519 key.

Tailscale provides the private network path between the devices.

## What I Learned

- SSH can provide remote command-line access to a Linux server.
- SSH commonly uses TCP port 22.
- SSH keys provide authentication without relying on password-only login.
- Tailscale creates a private network between authorized devices.
- A computer can have both LAN and Tailscale IP addresses.
- Tailscale allows devices on different physical networks to communicate.
- Remote access can be tested by moving the client onto a cellular hotspot.
- `$SSH_CONNECTION` can be used to inspect the endpoints of an active SSH session.
- Remote homelab access does not require exposing SSH directly to the public Internet.
