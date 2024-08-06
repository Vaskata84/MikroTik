# Setup OpenVPN Server and generate certs
#
# Change variables below if needed then copy the whole script
# and paste into MikroTik terminal window.
#

:global CN [/system identity get name]
:global PORT 1194

## generate a CA certificate
/certificate
add name=ca-template common-name="$CN" days-valid=3650 \
  key-usage=crl-sign,key-cert-sign
sign ca-template ca-crl-host=127.0.0.1 name="$CN"
:delay 10

## generate a server certificate
/certificate
add name=server-template common-name="server@$CN" days-valid=3650 \
  key-usage=digital-signature,key-encipherment,tls-server
sign server-template ca="$CN" name="server@$CN"
:delay 10

## create a client template
/certificate
add name=client-template common-name="client" days-valid=3650 \
  key-usage=tls-client

## create IP pool
/ip pool
add name=VPN-POOL ranges=192.168.252.128-192.168.252.224

## add VPN profile
/ppp profile
add dns-server=192.168.252.1 local-address=192.168.252.1 name=VPN-PROFILE \
  remote-address=VPN-POOL use-encryption=yes

## setup OpenVPN server
/interface ovpn-server server
set auth=sha1 certificate="server@$CN" cipher=aes128-cbc,aes192-cbc,aes256-cbc \
  default-profile=VPN-PROFILE mode=ip netmask=24 port="$PORT" \
  enabled=yes require-client-certificate=yes

## add firewall rules with specific positions
/ip firewall filter
add chain=input action=accept dst-port="$PORT" protocol=tcp \
  comment="Allow OpenVPN" place-before=0
add chain=input action=accept dst-port=53 protocol=udp \
  src-address=192.168.252.0/24 \
  comment="Accept DNS requests from VPN clients" place-before=1

## Setup completed. Do not forget to create a user.

## Configure complete
test.ovpn
#####client
dev tun
proto tcp
remote YOUR_SERVER_IP 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca /home/test/Downloads/openvpn/cert_export_supportpc.org.crt
cert /home/test/Downloads/openvpn/cert_export_user@supportpc.org.crt
key /home/test/Downloads/openvpn/cert_export_user@supportpc.org.key.nopass
remote-cert-tls server
data-ciphers AES-256-GCM:AES-128-GCM:AES-128-CBC
cipher AES-128-CBC
verb 3
auth-user-pass /home/vasil/Downloads/openvpn/user.auth
####

chmod 600 /home/test/Downloads/openvpn/cert_export_user@supportpc.org.key.nopass
chmod 600 /home/test/Downloads/openvpn/user.auth

test connections
openvpn --config /home/vasil/Downloads/openvpn/test.ovpn

