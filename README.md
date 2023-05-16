OpenVPN Server and certificate management on MikroTik. Only works for versions 6.XX




I've also found that there is an error raised by the script on routeros version 7.8. At least one of the issues is that Mikrotik has change the names of the aes ciphers. When running the script you will receive an error on this section:
setup OpenVPN server

/interface ovpn-server server
set auth=sha1 certificate="server@$CN" cipher=aes128,aes192,aes256
default-profile=VPN-PROFILE mode=ip netmask=24 port="$PORT"
enabled=yes require-client-certificate=yes

To fix the error you need to change the names used in the "cipher=" portion. If you use "aes128-cbc,aes192-cbc,aes256-cbc" then that portion of the script will not throw an error.


/interface ovpn-server server set auth=sha1 certificate="server@$CN" cipher=aes128-cbc,aes192-cbc,aes256-cbc default-profile=VPN-PROFILE mode=ip netmask=24 port="$PORT" enabled=yes require-client-certificate=yes



openvpn-install.sh
_________________________________________
sudo chmod +x openvpn-install.sh
sudo bash openvpn-install.sh
sudo systemctl cat openvpn-iptables.service
sudo more /etc/openvpn/server/server.conf
sudo systemctl stop openvpn-server@server.service
sudo systemctl start openvpn-server@server.service
sudo systemctl restart openvpn-server@server.service
sudo systemctl status openvpn-server@server.service
sudo find / -type f -name "iphone.ovpn"
sudo find / -type f -name "*.ovpn" -ls
