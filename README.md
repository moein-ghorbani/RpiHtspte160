# RpiHtspte160
Setting Up a Raspberry Pi Hotspot with Huawei E160 USB Modem
Sure, I will provide a step-by-step guide suitable for placing on GitHub. The guide will include the necessary commands and explanations for each step.

---

## Setting Up a Raspberry Pi Hotspot with Huawei E160 USB Modem

This guide explains how to set up a Raspberry Pi to act as a WiFi hotspot using a Huawei E160 USB modem for internet connectivity. The steps include configuring the necessary network interfaces, setting up the DHCP and DNS services, and creating a startup script to automate the process.

### Prerequisites

- A Raspberry Pi with Raspbian installed
- A Huawei E160 USB modem
- A WiFi dongle

### Step 1: Install Required Packages

Install the necessary packages for managing the WiFi hotspot and the USB modem.

```bash
sudo apt update
sudo apt install -y hostapd dnsmasq usb-modeswitch wvdial iptables-persistent
```

### Step 2: Configure USB Modeswitch

Create a configuration file for the Huawei E160 modem.

```bash
sudo nano /etc/usb_modeswitch.conf
```

Add the following content:

```
DisableSwitching=0
DisableMBIMGlobal=0
```

### Step 3: Configure wvdial

Configure the wvdial settings to establish a connection with the modem.

```bash
sudo nano /etc/wvdial.conf
```

Add the following content:

```
[Dialer Defaults]
Init1 = ATZ
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
Stupid Mode = 1
Modem Type = Analog Modem
Baud = 460800
New PPPD = yes
Modem = /dev/ttyUSB0
ISDN = 0
Phone = *99#
Password = password
Username = username
```

### Step 4: Configure Network Interfaces

Assign a static IP address to the WiFi interface.

```bash
sudo nano /etc/network/interfaces
```

Add the following lines:

```
auto wlan0
iface wlan0 inet static
    address 192.168.4.1
    netmask 255.255.255.0
```

### Step 5: Configure DHCP and DNS with dnsmasq

Set up dnsmasq to manage DHCP and DNS for the hotspot.

```bash
sudo nano /etc/dnsmasq.conf
```

Add the following content:

```
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```

### Step 6: Configure the Access Point with hostapd

Create the configuration file for hostapd.

```bash
sudo nano /etc/hostapd/hostapd.conf
```

Add the following content:

```
interface=wlan0
driver=nl80211
ssid=SweetCar
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=13661368
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

Specify the hostapd configuration file in the default configuration.

```bash
sudo nano /etc/default/hostapd
```

Uncomment and set the DAEMON_CONF line as follows:

```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### Step 7: Create Startup Script

Create a startup script to configure iptables, start the modem connection, and restart services.

```bash
sudo nano /etc/start_hotspot.sh
```

Add the following content:

```bash
#!/bin/bash

# Wait for the modem to be recognized
sleep 10

# Run usb_modeswitch
usb_modeswitch -c /etc/usb_modeswitch.conf

# Configure iptables
iptables -t nat -A POSTROUTING -o ppp0 -j MASQUERADE
iptables -A FORWARD -i ppp0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o ppp0 -j ACCEPT

# Start wvdial to connect to the internet
wvdial &

# Wait for wlan0 to be ready
while ! ifconfig wlan0 | grep -q "inet "; do
    sleep 1
done

# Assign static IP to wlan0
ifconfig wlan0 192.168.4.1 netmask 255.255.255.0

# Restart services
systemctl restart dnsmasq
systemctl restart hostapd
```

Make the script executable:

```bash
sudo chmod +x /etc/start_hotspot.sh
```

### Step 8: Add Startup Script to cron

Ensure the script runs at startup by adding it to cron.

```bash
sudo crontab -e
```

Add the following line:

```
@reboot /etc/start_hotspot.sh
```

### Step 9: Reboot and Test

Reboot the Raspberry Pi to apply all changes.

```bash
sudo reboot
```

After the reboot, check if the WiFi network "SweetCar" is visible and connect to it with the password "13661368". Verify that devices connected to this network can access the internet through the Huawei E160 modem.

---

This setup should now allow your Raspberry Pi to function as a WiFi hotspot, providing internet connectivity through the Huawei E160 USB modem.
