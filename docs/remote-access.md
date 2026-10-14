# Remote Access Lab — Tailscale + RDP

## Goal

The goal of this lab was to remotely access and control my Ubuntu iMac from my iPad.

I wanted two methods of remote access:

- SSH with Termius for command-line access
- RDP with Microsoft Windows App for full graphical desktop access

Tailscale allows these connections to work securely when I am away from my home network.

---

## My Setup

### Ubuntu iMac
- OS: Ubuntu 24.04 LTS
- Remote desktop: GNOME Remote Desktop
- RDP port: TCP 3389
- LAN IP: 10.0.0.x (redacted)
- Tailscale IP: 100.x.x.x (redacted)

### iPad
- RDP client: Microsoft Windows App
- SSH client: Termius
- Tailscale installed for remote connectivity

---

## Network Architecture

### At Home

iPad
  |
  | Home Wi-Fi
  |
  v
10.0.0.x (redacted)
  |
  | TCP 3389
  |
  v
Ubuntu iMac
GNOME Remote Desktop

### Away From Home

iPad
  |
  | Cellular / Hotspot / Outside Wi-Fi
  |
  v
Tailscale
  |
  | Encrypted Tailscale network
  |
  v
100.x.x.x (redacted)
  |
  | TCP 3389
  |
  v
Ubuntu iMac
GNOME Remote Desktop

---

## RDP Configuration

GNOME Remote Desktop was configured as the RDP server.

I verified its status with:

    grdctl status

The server showed:

- RDP enabled
- Port 3389
- TLS certificate configured
- TLS private key configured
- RDP credentials configured

Originally the server was configured as:

    View-only: yes

This allowed me to see the Ubuntu desktop but prevented me from controlling it.

I enabled remote control with:

    grdctl rdp disable-view-only

Afterward:

    grdctl status

showed:

    View-only: no

I could then control Ubuntu from the iPad.

---

## Verifying the RDP Server

I checked whether something was actually listening on TCP port 3389:

    sudo ss -ltnp | grep ':3389'

The output showed that:

    gnome-remote-desktop-daemon

was listening on port 3389.

This proved that the RDP server was running and accepting network connections.

---

## LAN vs Tailscale Addresses

One major lesson from this lab was understanding the difference between a LAN IP and a Tailscale IP.

My iMac LAN address is:

    10.0.0.x (redacted)

This address is useful while connected to my home network.

My iMac Tailscale address is:

    100.x.x.x (redacted)

This address allows my other Tailscale devices to reach the iMac through the Tailscale network.

When I was away from home, trying to use 10.0.0.x in Windows App failed because that address belongs to my private home network.

Using the Tailscale address allowed the connection attempt to reach the iMac remotely.

---

## Troubleshooting

The remote RDP connection initially failed even though SSH through Tailscale worked.

Instead of randomly changing settings, I tested each layer separately.

### 1. Verified GNOME Remote Desktop

I confirmed that GNOME Remote Desktop was active and RDP was enabled.

### 2. Verified Port 3389

I used `ss` to confirm GNOME Remote Desktop was listening on TCP port 3389.

### 3. Verified Firewall Configuration

UFW was active.

The RDP server was reachable during testing, proving that the connection was reaching the service.

### 4. Verified TLS

GNOME Remote Desktop was configured with a TLS certificate and private key.

I verified that the certificate and private key matched and that the certificate was currently valid.

### 5. Captured Network Traffic

I used `tcpdump` to watch RDP traffic traveling through the Tailscale interface:

    sudo tcpdump -ni tailscale0 host <TAILSCALE-CLIENT-IP> and tcp port 3389

The capture showed packets traveling between the iPad and iMac.

This proved:

- Tailscale traffic was reaching the iMac
- TCP port 3389 was reachable
- The iPad and iMac were exchanging RDP data

This helped eliminate basic routing and connectivity as the only possible causes.

### 6. Checked GNOME RDP Logs

I inspected GNOME Remote Desktop logs with:

    journalctl --user -u gnome-remote-desktop

During failed attempts, the logs included authentication/security negotiation errors such as:

    Message Integrity Check (MIC) verification failed
    SEC_E_MESSAGE_ALTERED
    client authentication failure

Other attempts produced:

    BIO_read retries exceeded
    ERRCONNECT_CONNECT_TRANSPORT_FAILED

These logs showed that some failures occurred after the network connection had already reached the RDP service.

### 7. Tested Locally

Once I was physically home, I connected Windows App directly to:

    10.0.0.x (redacted)

The RDP session worked.

This was an important troubleshooting step because it proved that:

- Windows App worked
- GNOME Remote Desktop worked
- The RDP credentials worked
- The Ubuntu graphical session worked
- The RDP server configuration was functional

### 8. Tested Tailscale Again

I then connected Windows App to:

    100.x.x.x (redacted)

The connection succeeded.

### 9. Tested From an Outside Network

For the final test, I removed the iPad from my home Wi-Fi and connected it through a hotspot.

The iMac remained connected to my home network.

The final path was:

    iPad
      -> Hotspot / Cellular Internet
      -> Tailscale
      -> Home Internet
      -> Ubuntu iMac
      -> RDP

The Windows App successfully connected to the Ubuntu desktop.

This proved that remote graphical access works outside my home network.

---

## Final Remote Access Setup

I now have two remote administration methods.

### SSH

Termius
  -> Tailscale
  -> Ubuntu iMac
  -> SSH

This gives me command-line access and is useful for administration and troubleshooting.

### RDP

Microsoft Windows App
  -> Tailscale
  -> Ubuntu iMac
  -> GNOME Remote Desktop

This gives me graphical access to the Ubuntu desktop.

---

## Performance

RDP worked over a cellular hotspot but was noticeably slower than when connected through my home Wi-Fi.

This demonstrated that a successful connection does not necessarily mean identical performance.

RDP performance can be affected by:

- Latency
- Cellular signal quality
- Hotspot performance
- Upload/download bandwidth
- Screen resolution
- Graphical activity

---

## What I Learned

This lab taught me how several technologies work together:

- SSH
- RDP
- TCP ports
- Private LAN addressing
- Tailscale addressing
- VPN-style overlay networks
- UFW firewall rules
- TLS certificates
- GNOME Remote Desktop
- systemd user services
- Network packet capture with tcpdump
- Log analysis with journalctl

The biggest troubleshooting lesson was to test one layer at a time.

Instead of assuming that "RDP is broken," I separated the problem into:

    Client
      -> Network
      -> Tailscale
      -> TCP
      -> Firewall
      -> RDP Server
      -> Authentication
      -> Desktop Session

By testing each layer individually, I could determine what was working before changing configuration.

---

## Security Notes

Passwords, private keys, authentication tokens, and other credentials should never be committed to a public GitHub repository.

The RDP TLS private key remains stored locally on the Ubuntu server.

Tailscale provides the private network path used for remote access, so I do not need to expose RDP port 3389 directly to the public Internet.
