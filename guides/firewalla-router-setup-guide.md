---
title: Firewalla Router Setup Guide
description: Guide on how to setup a Firewalla router for use with Olilo
category: Setup
author: Keith Lewis
lastUpdated: 04/07/2026
---

# Introduction

Firewalla manufacture a range of dedicated firewall and router appliances. For the purposes of this guide, we are focusing on the Firewalla Gold SE, though the basic setup in the Firewalla mobile app will be similar across their product line.

This guide assumes that you have switched to Olilo from another ISP (for example, using the One Touch Switch process), have connected your Openreach ONT to your Firewalla, and are able to access the Firewalla app on your mobile device. If you are using a 2.5GbE ONT, ensure it is plugged into a corresponding 2.5GbE port on your router (such as Port 4 on the Gold SE).

## Basic Setup

Once you are in the app, navigate to the Network menu and select WAN Rather than deleting your existing setup, you can simply edit your current WAN configuration. Depending on which physical network you are using, the following steps are slightly different. 

### Openreach

- For 'Connection Type', near the top of the page, select 'PPPoE'.
- Scroll down to Account Settings and input the Username and Password supplied by Olilo. 
- Please note: Unlike some legacy providers, your Olilo PPPoE credentials are highly personal and unique to you. They are usually emailed manually by the support team (e.g., Aydan) roughly 24 hours before your activation date.
- Ensure any VLAN settings are left blank or disabled, as Olilo does not require a VLAN on the Openreach network.
- Tap 'Save' at the top or bottom of the screen to apply the settings. Your static IPv4 connection should authenticate straight away.

### IPv6

- Setting up the prosumer static /48 block can be tricky. To begin, navigate to the IPv6 section within your WAN settings.
- First, set the 'IPv6 Connection Type' to 'DHCPv6' to allow the router to pull your initial IPv6 allocation.
- Wait for the Firewalla to generate a gateway address and make a note of it.
- Next, change the 'IPv6 Connection Type' to 'Static'.
- Set the 'Prefix' to '64' and input the gateway address you noted down.
- Modify your WAN IPv6 Address: You must add the word `feed` into your IPv6 address so that it becomes the third hextet. (e.g., if your assigned block starts with `2b12:2020:1289::`, you would adjust your WAN address to look like `2b12:2020:feed:1289::1`). See this WAN configuration example.
- Tap 'Save'.
- To assign IPv6 to your local networks, go to 'Network' -> LAN or VLAN.
- Enable IPv6 and set it to 'Static' (replacing the initial DHCPv6 setting if it was active).
- Use an address that matches your WAN block, replacing the final number to match your VLAN ID for easier segmentation. (e.g., `XX11:2XX6:1XX9:3::` for VLAN 3). See this Local LAN example.
- Tap 'Save' on each local network.

## Still Stuck?

Before contacting Olilo:
1. Try to restart the router, once you have followed the above steps and double check your personal PPPoE login details.
2. **Speed Test Fluctuations:** If you are on the 1.6Gbps package, you may notice the Firewalla's internal speed tests fluctuate wildly at first. This usually settles down after roughly 30 days. As long as your actual device throughput (like a NAS) is consistently high, the network is operating normally.
3. **Blocked Services (e.g., Sky Glass):** Because IPv4 addresses are occasionally reused, you might find that services like Sky Glass are blocked by your newly assigned static IPv4. If this happens, contact support to request a fresh static IPv4 address.
4. Get in touch on our discord channel:

- **Discord:** discord.gg/olilo
