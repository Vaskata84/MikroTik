<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OpenVPN Server on MikroTik 7x</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212;
            color: #ffffff;
            text-align: center;
            padding: 20px;
        }
        .container {
            max-width: 900px;
            margin: auto;
            background: #1e1e1e;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
        }
        pre {
            background: #2d2d2d;
            padding: 10px;
            border-radius: 5px;
            text-align: left;
            overflow-x: auto;
        }
        img {
            width: 100%;
            border-radius: 5px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>OpenVPN Server and Certificate on MikroTik 7x</h1>
        <p>Below is the configuration script for setting up an OpenVPN server on MikroTik RouterOS 7x.</p>
        <img src="https://upload.wikimedia.org/wikipedia/commons/3/3b/MikroTik_Logo.png" alt="MikroTik Logo">
        <pre>
:global CN [/system identity get name]
:global PORT 1194
:global DAYSVALIDCA 9450
:global DAYSVALIDSERVER 9450
:global DAYSVALIDCLIENT 3650
:global OVPNCLIENTCERTIFICATETEMPLATENAME "ovpn-ClientTemplate"
:global OVPNCLIENTCERTIFICATETEMPLATECOMMONNAME "ovpnClientTemplate"
:global OVPNPROFILENAME "ovpn-profile"
:global OVPNNETWORKMASK "24"
:global OVPNNETWORK "10.0.6.0/24"
:global OVPNIPADDRESS "10.0.6.254"
:global OVPNCLIENTIPADDRESS "10.0.6.253"
:global OVPNDHCPPOOLNAME "ovpn-dhcpPool"
:global OVPNDHCPPOOLRANGE "10.0.6.249-10.0.6.253"
:global USERNAME "ovpn-Client1"
:global PASSWORDUSERLOGIN "ovpn-Client1-Password"
:global PASSWORDCERTPASSPHRASE "12345678"

/certificate
add name=ca-template common-name="$CN" days-valid="$DAYSVALIDCA" key-usage=crl-sign,key-cert-sign
sign ca-template ca-crl-host=127.0.0.1 name="$CN"
:delay 10

/certificate
add name=server-template common-name="ovpn-server@$CN" days-valid="$DAYSVALIDSERVER" key-usage=digital-signature,key-encipherment,tls-server
sign server-template ca="$CN" name="ovpn-server@$CN"
:delay 10

/certificate
add name="$OVPNCLIENTCERTIFICATETEMPLATENAME" common-name="$OVPNCLIENTCERTIFICATETEMPLATECOMMONNAME" days-valid="$DAYSVALIDCLIENT" key-usage=tls-client
:delay 5

/ip pool
add name="$OVPNDHCPPOOLNAME" ranges="$OVPNDHCPPOOLRANGE"

/ppp profile
add local-address="$OVPNIPADDRESS" name="$OVPNPROFILENAME" remote-address="$OVPNDHCPPOOLNAME" use-ipv6=no use-upnp=no use-compression=no use-mpls=no use-encryption=yes only-one=yes

/interface ovpn-server server
add certificate="ovpn-server@$CN" cipher=blowfish128,aes128-cbc,aes192-cbc,aes256-cbc comment=openvpn default-profile="$OVPNPROFILENAME" disabled=no name=OpenVpn-Server

/interface ovpn-server
add name="$USERNAME" user="$USERNAME"

/ip firewall filter
add chain=input action=accept dst-port="$PORT" protocol=tcp comment="OpenVPN - Allow" place-before=0
add chain=input action=accept dst-port=53 protocol=udp src-address="$OVPNNETWORK" comment="OpenVPN - Accept DNS requests"
        </pre>
        <img src="https://www.networkstraining.com/wp-content/uploads/2020/09/mikrotik-openvpn.jpg" alt="MikroTik OpenVPN Setup">
    </div>
</body>
</html>
