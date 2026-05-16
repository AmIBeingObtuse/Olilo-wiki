---
title: L2TP Guide
description: Guide on the basics of the L2TP Service
category: Setup
author: FingerlessGloves
lastUpdated: 17/05/2026
---

# L2TP Guide

The Olilo **L2TP service** gives you a real, static, publicly routable IP address over **any UK internet connection**.

This is **not a privacy VPN** like PIA or NordVPN. It won't hide your traffic or change your country for streaming. Instead, it tunnels one or more static IPs back to **your own router or environment**, your traffic then exits via Olilo's IP space and internet peering, while your existing broadband stays exactly as it is.

## What you can use it for

- **Escape CGNAT** - many 4G/5G, Starlink, altnet and budget connections put you behind Carrier-Grade NAT, blocking inbound traffic. L2TP gives you a proper public IP again.
- **Get a static IP** - your line may only offer a dynamic IP, or none at all. L2TP provides a fixed IPv4 address with custom reverse DNS.
- **Plenty of IPv6** - you also get a **/48** IPv6 prefix. That's **65,536** `/64` subnets, so every VLAN or network you run can have its own.
- **Need more IPs** - extra static addresses can be routed to you for self-hosting, mail, game servers and monitoring.
- **No port restrictions** - run services needing reliable inbound connectivity without your ISP's NAT or port restrictions.
- **Use Olilo's peering** - route traffic out via Olilo's network rather than your current ISP's, known to fix poor routing with small ISPs.
- **Failover without renumbering** - the IP lives on the tunnel, so you can switch lines (e.g. fibre to 5G backup) and keep the same IP.

## Compatibility

Works over **any UK internet connection**, including Openreach and altnet fibre, cable, copper, 4G/5G, and Starlink. If it can reach the internet, it can carry the tunnel.

Most routers and firewalls that support L2TP work as the client, for example MikroTik, OPNsense, pfSense, VyOS, Ubiquiti EdgeOS, and Linux (xl2tpd).

## Logging in

The Olilo L2TP server (LNS endpoint) is:

```text
l2tp.olilo.co.uk
```

Your **username and password are provided by email after you place your order**. The username looks like:

```text
firstname.lastname@olilo.l2tp
```

On your router or firewall, create an L2TP client using:

```text
Server / LNS:  l2tp.olilo.co.uk
Username:      firstname.lastname@olilo.l2tp
Password:      supplied by Olilo after ordering
```

Olilo's service does **not** use IPsec, ensure IPsec is disabled on your L2TP client.

Once the tunnel is up, your primary static IPv4 `/32` is assigned automatically. The /48 IPv6 needs to be requested via DHCPv6 Prefix Delegation, if you want SLAAC with static routing of the `/48`, please contact Olilo Support. For most users the default of DHCPv6 PD will work just fine.

> **Note:** The L2TP/IPv4 tunnel uses an MTU of **1460**. Set this on the tunnel interface and clamp TCP MSS on your egress L2TP traffic, or some sites may hang or partially load.  
> If your WAN MTU is lower than 1500, e.g. 1492, you will need to adjust the MTU of the L2TP and the MSS value you're clamping to, dropping it by the difference, in this case 8 bytes.

## Config examples

> **Note:** These are starting points, not the only way to do it. How you use it is up to you (single device, whole-network gateway, failover, a DMZ for servers, and so on). Treat the examples below as a baseline and adjust to suit your own network.

Replace the username, password, WAN interface and IPv6 prefix with your own details.

### MikroTik (RouterOS)

This assumes a stock RouterOS default config, which already has `WAN`/`LAN` interface lists and a `srcnat ... action=masquerade` rule for the `WAN` list.

```routeros
# L2TP client no IPsec, MTU 1460, assuming 1500 MTU parent
# default-route-distance=1 so the tunnel is the default route once up
/interface l2tp-client
add name=olilo connect-to=l2tp.olilo.co.uk \
    user="firstname.lastname@olilo.l2tp" password="your-password" \
    use-ipsec=no add-default-route=yes default-route-distance=1 \
    max-mtu=1460 max-mru=1460 disabled=no

# Push ether1's default route to distance 2 so it's only a fallback.
# Stock config leaves it at 1, which would tie with the tunnel.
/ip dhcp-client set [find interface=ether1] default-route-distance=2
/ipv6 dhcp-client set [find interface=ether1] default-route-distance=2

# Treat the tunnel as a WAN. This reuses the default config's masquerade
# NAT rule (so IPv4 LAN clients get out) and its WAN firewall protection.
/interface list member
add list=WAN interface=olilo comment="Olilo L2TP"

# Add DHCPv6 Client, for our /48 and Tunnel IP
# This requests our /48 route to be routed over the L2TP tunnel
/ipv6 dhcp-client
add interface=olilo pool-name=olilo request=address,prefix

# Clamp TCP MSS on traffic leaving the tunnel, or some sites stall
# IPv4: 1460 - 20 IP - 20 TCP = 1420
/ip firewall mangle
add chain=forward action=change-mss new-mss=1420 passthrough=yes \
    protocol=tcp tcp-flags=syn tcp-mss=1421-65535 out-interface=olilo
# IPv6: 1460 - 40 IP - 20 TCP = 1400
/ipv6 firewall mangle
add chain=forward action=change-mss new-mss=1400 passthrough=yes \
    protocol=tcp tcp-flags=syn tcp-mss=1401-65535 out-interface=olilo

# Assign a /64 from your /48 to the LAN bridge
/ipv6 address
add address=2001:db8:abcd:1::1/64 interface=bridge advertise=yes
```

Go to https://test-ipv6.com. You should now see ISP appearing as `OLILO` and hopefully an IPv4 and IPv6 address.  
If you only see an IPv4 address, ensure your client has an IPv6 address. You may need to reconnect to your WiFi or Ethernet.

### OPNsense

L2TP is added as a point-to-point device, then assigned as an interface.

```text
Interfaces -> Devices -> Point-to-Point
Press +

Type:              L2TP
Link interface:    Your WAN
Description:       Olilo L2TP
Username:          firstname.lastname@olilo.l2tp
Password:          Supplied by Olilo after ordering
Local IP:          Leave Default
Gateway:           82.39.94.1 (OPNsense doesn't support using a domain name)
```

Then assign and enable it:

```text
Interfaces -> Assignments
Assign a new interface, and select our new L2TP interface name it Olilo L2TP

Edit the new Interface
Enable Interface
IPv6 Configuration Type:  DHCPv6

DHCPv6 client configuration
Prefix delegation size:   48
Request DNS configuration: Untick

Save and Apply Changes
```

Setup the gateways

```text
1. Go to System -> Settings -> General
   Enable `Gateway switching`

2. Now we config the Gateways, go to System -> Gateways -> Configuration, edit each Olilo Gateway setting it as a `Upstream Gateway`
   On the IPv4 gateway only untick `Disable Gateway Monitoring`

3. Press Apply
```

Then set up IPv6 on each LAN:

```text
1. Interfaces -> [LAN] -> IPv6 Configuration Type: Static IPv6
   IPv6 address: a /64 from your /48, e.g. 2001:db8:abcd:1::1/64
   Save and Apply Changes

2. Services -> Router Advertisements
   Press +
   Interface: LAN
   Mode: Stateless
   Save and Apply
```

Clamp TCP MSS to **1420** so large packets don't stall.

```
Go to **Firewall -> Settings -> Normalization**
Press +

Interface:      OliloL2TP
Direction:      Any
Protocol:       IPv4
Description:    Olilo L2TP MSS Clamp IPv4
Max mss:        1420

Create a second rule

Interface:      OliloL2TP
Direction:      Any
Protocol:       IPv6
Description:    Olilo L2TP MSS Clamp IPv6
Max mss:        1400
```

Finally we restart the L2TP session, to ensure all changes take effect

```
Go to Interfaces -> Devices -> Point-to-Point, look for Olilo, and then resave the configuration

Wait 15 seconds
```

Go to https://test-ipv6.com. You should now see ISP appearing as `OLILO` and hopefully an IPv4 and IPv6 address.  
If you only see an IPv4 address, ensure your client has an IPv6 address. You may need to reconnect to your WiFi or Ethernet.

## Still stuck?

Check that:

1. The server address is exactly `l2tp.olilo.co.uk`.
2. Your username includes the full `@olilo.l2tp` suffix.
3. IPsec is **disabled** on the L2TP client.
4. Your underlying connection has working internet access.

Then ask in the Olilo Discord: **discord.gg/olilo**
