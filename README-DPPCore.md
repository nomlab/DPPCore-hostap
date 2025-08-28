## What is DPPCore-hostap
DPPCore-hostap extends hostap to enable the inclusion of additional information in the data passed to the Enrollee when provisioning using DPP.

## Installation
1. Install dependencies.
   
   DPPCore-hostap requires libnl.
   Example for Debian or Ubuntu case:
   ```bash
    sudo apt-get install -y libnl-3-dev libnl-genl-3-dev libnl-route-3-dev
   ```

2. Clone code and Checkout blanch.
   ```bash
   https://github.com/nomlab/DPPCore-hostap.git
   ```
   ```bash
   cd /path/to/your/DPPCore-hostap
   git checkout DPPCore
   ```

3. Make hostapd.
   ```bash
   cd /path/to/your/DPPCore-hostap/
   cd hostapd
   cp defconfig .config
   make
   ```
   `./hostapd` is created.


## Setup environment
Create a virtual network interface using NetworkManager.
The Interface name is arbitrarily, but here it is described as qrdia.


1. Remove from NetworkManager management.
   `/etc/NetworkManager/conf.d/99-unmanage-qrdia.conf`
   ```
   [keyfile]
     unmanaged-devices=interface-name:qrdia
   ```
2. Create hostapd service.
   `/etc/systemd/system/hostapd-local.service`
   ```
   [Unit]
   Description=Custom Hostapd with interface setup
   After=network.target
   
   [Service]
   Type=simple
   ExecStartPre=/path/to/your/setup-iface.sh
   ExecStart=/path/to/your/hostapd /path/to/your/hostapd_dpp.conf
   Restart=on-failure
   
   [Install]
   WantedBy=multi-user.target
   ```
3. Create a shell script to launch a virtual network interface.
   `setup-iface.sh`
   ```
   #!/bin/bash
   set -e
   
   ip link delete qrdia 2>/dev/null || true
   
   iw phy phy0 interface add qrdia type __ap
   
    # Set MAC address 02:11:22:33:44:55 (optional), but avoid unusable ranges
    # Create it from the locally administered unicast range
    # Refer to https://en.wikipedia.org/wiki/MAC_address
   ip link set qrdia address 02:11:22:33:44:55
   
   ip link set qrdia up
   ```

## How to use
1. Start DPPCore-hostap.
   ```
   sudo systemctl start hostapd-local.service
   ```


