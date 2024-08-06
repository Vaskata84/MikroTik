<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    <h1>OpenVPN Server and Certificate Management on MikroTik</h1>
    <p>Only works for versions 6.XX</p>
    <p>I've also found that there is an error raised by the script on routeros version 7.8. At least one of the issues is that Mikrotik has changed the names of the AES ciphers. When running the script you will receive an error on this section:</p>
    <h2>Setup OpenVPN Server</h2>
    <pre><code>/interface ovpn-server server
set auth=sha1 certificate="server@$CN" cipher=aes128,aes192,aes256
default-profile=VPN-PROFILE mode=ip netmask=24 port="$PORT"
enabled=yes require-client-certificate=yes
    </code></pre>
    <p>To fix the error you need to change the names used in the "cipher=" portion. If you use "aes128-cbc,aes192-cbc,aes256-cbc" then that portion of the script will not throw an error.</p>
    <pre><code>/interface ovpn-server server
set auth=sha1 certificate="server@$CN" cipher=aes128-cbc,aes192-cbc,aes256-cbc
default-profile=VPN-PROFILE mode=ip netmask=24 port="$PORT"
enabled=yes require-client-certificate=yes
    </code></pre>
    <h2>openvpn-install.sh</h2>
    <hr>
    <pre><code>sudo chmod +x openvpn-install.sh
sudo bash openvpn-install.sh
sudo systemctl cat openvpn-iptables.service
sudo more /etc/openvpn/server/server.conf
sudo systemctl stop openvpn-server@server.service
sudo systemctl start openvpn-server@server.service
sudo systemctl restart openvpn-server@server.service
sudo systemctl status openvpn-server@server.service
sudo find / -type f -name "iphone.ovpn"
sudo find / -type f -name "*.ovpn" -ls
    </code></pre>
</body>
</html>
