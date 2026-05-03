---
title: UniFi Network Setup Guide
description: Guide on how to setup a UniFi gateway for use with Olilo
category: Setup
author: Sam Tilling
lastUpdated: 22/04/2026
---

# Introduction

UniFi gateways and routers - including the UniFi Dream Router, Dream Machine, and Cloud Gateway range - all share the same web-based management interface, UniFi Network. For the purposes of this guide, the basic setup will be the same regardless of which UniFi device you are using.

If, by logging in, your dashboard looks like this, this guide is for you.

![The UniFi Network dashboard showing your gateway, ISP connection status, latency, and WAN IP addresses. The gear icon at the bottom of the left-hand icon bar opens Settings.](images/unifi/unifi-homepage.jpg)

This guide assumes that you have switched to Olilo from another ISP and can access the UniFi Network interface, either through the local web UI (typically at `https://192.168.0.1` or `https://192.168.1.1`) or via the UniFi Site Manager at `unifi.ui.com` if your gateway is reachable on the internet. If you are unsure how to access your device, please refer to [Ubiquiti's getting started documentation](https://help.ui.com), and come back here once you have logged in.

## Basic Setup

Once you're in, click the gear icon at the bottom of the left-hand icon bar to open Settings, then select **Internet** from the left menu as shown below.

![The UniFi Network Settings page, showing the left menu with WiFi, Networks, Internet, VPN, CyberSecure, High Availability, and System. Internet is highlighted. The main panel shows an overview of Networks and Internet connections.](images/unifi/unifi-settings-page.jpg)

Depending on which physical network you are using (Openreach, CityFibre, or Freedom Fibre) the following steps are slightly different. If you are not sure which physical network you are connected to, please refer to emails you have received from Olilo to confirm.

### Openreach

- On the Internet page, you will see your active internet connections listed. Click on **Internet 1** to open the configuration panel, as shown below.

  ![The UniFi Network Internet settings page showing the left menu with Internet selected, the gateway port diagram, WAN port assignments, and the Internet 1 configuration panel open on the right.](images/unifi/unifi-internet-settings.jpg)

- Scroll down to the **IPv4 Configuration** section and set **Connection** to **PPPoE**.
- Enter the **Username** and **Password** supplied by Olilo in the fields provided.
- The rest of the settings on this page can be left as default.
- Click **Apply Changes** at the bottom of the page to save the settings.

> **Note:** Your PPPoE credentials are case-sensitive and can be found in your Olilo welcome email.

### CityFibre

- On the Internet page, click on **Internet 1** to open the configuration panel, as shown below.

  ![The UniFi Network Internet settings page showing the left menu with Internet selected, the gateway port diagram, WAN port assignments, and the Internet 1 configuration panel open on the right.](images/unifi/unifi-internet-settings.jpg)

- In the configuration panel, scroll up to the **Advanced** section and switch the toggle from **Auto** to **Manual**.
- Tick **VLAN ID** and enter **911**, as shown below.

  ![The Internet 1 configuration panel showing the Advanced section with the Manual toggle selected, VLAN ID ticked and set to 911.](images/unifi/unifi-internet-config-1.jpg)

- Scroll down to the **IPv4 Configuration** section and set **Connection** to **DHCP**, as shown below.

  ![The IPv4 Configuration section of the Internet 1 panel, showing DHCP selected as the connection type, with Auto DNS Server unticked and primary DNS set to 1.1.1.1 and secondary DNS set to 8.8.8.8.](images/unifi/unifi-internet-config-2.jpg)

- The rest of the settings on this page can be left as default.
- Click **Apply Changes** at the bottom of the page to save the settings.

### Freedom Fibre

- On the Internet page, click on **Internet 1** to open the configuration panel, as shown below.

  ![The UniFi Network Internet settings page showing the left menu with Internet selected, the gateway port diagram, WAN port assignments, and the Internet 1 configuration panel open on the right.](images/unifi/unifi-internet-settings.jpg)

- Scroll down to the **IPv4 Configuration** section and set **Connection** to **DHCP**, as shown below.

  ![The IPv4 Configuration section of the Internet 1 panel, showing DHCP selected as the connection type, with Auto DNS Server unticked and primary DNS set to 1.1.1.1 and secondary DNS set to 8.8.8.8.](images/unifi/unifi-internet-config-2.jpg)

- The rest of the settings on this page can be left as default.
- Click **Apply Changes** at the bottom of the page to save the settings.

### IPv6

- To activate IPv6, remain on the **Settings, Internet** page and click on **Internet 1** to open the configuration panel.
- Scroll down past the IPv4 Configuration section to **IPv6 Configuration**.
- Set **Connection** to **DHCPv6**.
- Click **Apply Changes** at the bottom of the page to save your settings.

## DNS Servers

By default, UniFi will use your ISP's DNS servers. Some users prefer to use well-known public DNS servers instead. We suggest **1.1.1.1** (Cloudflare) and **8.8.8.8** (Google) as reliable, widely used options that are easy to remember.

To set these, click on **Internet 1** to open the configuration panel, scroll down to **IPv4 Configuration**, untick **Auto DNS Server**, and enter **1.1.1.1** as the Primary Server and **8.8.8.8** as the Secondary Server, as shown below.

![The IPv4 Configuration section of the Internet 1 panel, showing Auto DNS Server unticked and primary DNS set to 1.1.1.1 and secondary DNS set to 8.8.8.8.](images/unifi/unifi-internet-config-2.jpg)

## Still Stuck?

Before contacting us:

1. Try to restart the gateway once you have followed the above steps, and double check the PPPoE login details if you need to use them.
2. Navigate to **Settings, Internet** to check whether the gateway shows as connected, and make a note of any error messages.
3. Check the system log, which can be accessed by navigating to **Settings, System, System Log**. Make a note of any error messages.
4. Get in touch on our discord channel:
   - **Discord:** discord.gg/olilo

> **Last Resort:** If you are still unable to connect, you can factory reset your UniFi gateway by inserting a paperclip into the reset hole on the device and holding it for around 10 seconds until the LED flashes. Once reset, use the **UniFi mobile app** to run through the first-time setup wizard, which will guide you through the initial configuration step by step. You will then need to re-apply the settings in this guide once setup is complete.