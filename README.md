# update-resolv-conf-mageia
DNS server extraction/substitution script for OpenVPN (Mageia Linux)

The original script: [update-resolv-conf.sh](https://github.com/alfredopalhares/openvpn-update-resolv-conf/blob/master/update-resolv-conf.sh)
The author of the original script: [Alfredo Palhares](https://github.com/alfredopalhares)

Use in the *.ovpn configuration:
```
script-security 2
up /etc/openvpn/update-resolv-conf.sh
down /etc/openvpn/update-resolv-conf.sh
```
Use in Startup parameters:
```
--script-security 2 --up /etc/openvpn/update-resolv-conf.sh \
--down /etc/openvpn/update-resolv-conf.sh
```
The original script `update-resolv-conf.sh` It is intended for Ubuntu and other operating systems that have the `openresolv` package in their repository. However, Mageia does not have an `openresolv` package, so the option [-x](https://github.com/alfredopalhares/openvpn-update-resolv-conf/issues/18) unavailable. The only thing that `resolvconf` can do well is to add (not replace) DNS records to the contents of `/etc/resolv.conf`, which the OpenVPN client receives. As a result, if initially the `/etc/resolv.conf` file contained a DNS record pointing to the default gateway (for example, 192.168.0.1), then it will not be destroyed. On the contrary, this entry will be added to the end of the `/etc/resolv.conf` file:
```
# Generated by resolvconf
nameserver 209.222.18.222
nameserver 209.222.18.218
nameserver 192.168.0.1
```
This DNS configuration [is vulnerable](https://github.com/alfredopalhares/openvpn-update-resolv-conf/issues/14) and any DNS leak tester will easily report your true IP address, despite the fact that the IP DNS was obtained from inside the VPN above. Thus, the effectiveness of VPN is limited only by the visible change of the external IP address (location). `NetworkManager` is also no exception and by default provides an insecure, vulnerable OpenVPN connection.  
  
Therefore, the original script `update-resolv-conf.sh` it was modified in such a way that `resolvconf` is only used to return the settings of the original `/etc/resolv.conf`. Depending on the settings coming from the VPN server, the number of nameservers can be: 0 - absence (you need to replace with a pair of your own), 1 - the only DNS (you need to add another one of your own) and 1> - at least 2. In addition, when returning DNS settings, `systemd-resolved` can be used, and not `resolvconf`. I tried to take all this into account in this modification of the script - `update-resolv-conf-mageia.sh`.  
  
**Note:** the script is already used in [OpenVPN-GUI](https://github.com/AKotov-dev/OpenVPN-GUI), [ProtonVPN-GUI](https://github.com/AKotov-dev/protonvpn-gui) and [Luntik](https://github.com/AKotov-dev/luntik). 
