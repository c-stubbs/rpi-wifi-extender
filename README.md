# Raspberry Pi WiFi Extender

A quick reference repository to help setup a Raspberry Pi (or any Ubuntu system) to function as a wifi extender/hotspot using an external USB wifi adapter.

## Description


This procedure will setup a Raspberry Pi (or any Ubuntu system) to act as a WiFi hotspot (broadcasting a unique WiFi SSID allowing connection into a separate WiFi network). Other than a Raspberry Pi, you'll need a USB WiFi adapter that supports access point mode.

Putting all of this information here so I can remember how to set it up again in the future...

## Getting Started

### Requirements

* A Raspberry Pi
* A USB WiFi adapter that supports functioning as an access point
  * Verify this by running `nmcli -f all dev show wlan0 | grep WIFI-PROPERTIES.AP` which should show `WIFI-PROPERTIES.AP:   YES`
* NetworkManager

### Setup

On your RaspberryPi, make sure that NetworkManager is running
```
sudo service NetworkManager status
```

If it isn't
  
  1. Install NetworkManager

  ```
  sudo apt install network-manager
  ```
  2. Start NetworkManager and make it run automatically

  ```
  sudo service NetworkManager start
  sudo systemctl enable NetworkManager
  ```
Create a new connection on the interface you want to act as the hotspot
  ```
  sudo nmcli device wifi hotspot ssid <hotspot_name> password <hotspot_password> ifname <hotspot_interface>
  ```
Modify a few things in the connection using nmcli
  ```
  sudo nmcli con modify hotspot connection.autoconnect false
  sudo nmcli con modify hotspot 802-11-wireless.pmf disable
  sudo nmcli con modify hotspot ipv4.method shared
  sudo nmcli con modify hotspot 802-11-wireless.mode ap
  sudo nmcli con modify hotspot 802-11-wireless.band bg
  ```
Enable IP forwarding on the Pi
  ```
  sudo sysctl -w net.ipv4.ip_forward=1
  ```
Modify the IP tables to forward traffic from one interface to the other
  ```
  sudo iptables -t nat -A POSTROUTING -o <internet_interface> -j MASQUERADE
  sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
  sudo iptables -A FORWARD -i <hotspot_interface> -o <internet_interface> -j ACCEPT
  ```
***NOTE: The modifications to the IP tables won't persist with out additional steps, so they're included in the run script to start up the network.***

### Running It
The network isn't configured to run automatically on boot (although that can be done), and can be manually activated using the `run-hotspot.sh` script.
```
./run-hotspot.sh
```
***NOTE: `wlan0` and `wlan1` interfaces in the `./run-hotspot.sh` script should be replaced with the `<hotspot_interface>` and `<internet_interface>` used in the setup portion.***

## Authors

[Chandler Stubbs](https://github.com/c-stubbs)

## License

This project is licensed under the MIT License - see the LICENSE file for details
