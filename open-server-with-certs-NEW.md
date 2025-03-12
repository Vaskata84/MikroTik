# OpenVPN Server and certificate management on MikroTik

## Contents

- [Setup OpenVPN server and generate certificates](#setup-openvpn-server-and-generate-certificates)
- [Add a new user](#add-a-new-user)
- [Setup OpenVPN client](#setup-openvpn-client)
- [Decrypt private key to avoid password asking (optional)](#decrypt-private-key-to-avoid-password-asking-optional)
- [Delete a user and revoke his certificate](#delete-a-user-and-revoke-his-certificate)
- [Revert OpenVPN server configuration on MikroTik](#revert-openvpn-server-configuration-on-mikrotik)

## Setup OpenVPN server and generate certificates

```sh
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
set auth=sha1 certificate="server@$CN" cipher=aes128,aes192,aes256 \
  default-profile=VPN-PROFILE mode=ip netmask=24 port="$PORT" \
  enabled=yes require-client-certificate=yes

## add a firewall rule
/ip firewall filter
add chain=input action=accept dst-port="$PORT" protocol=tcp \
  comment="Allow OpenVPN"
add chain=input action=accept dst-port=53 protocol=udp \
  src-address=192.168.252.0/24 \
  comment="Accept DNS requests from VPN clients"
move [find comment="Allow OpenVPN"] 0
move [find comment="Accept DNS requests from VPN clients"] 1

## Setup completed. Do not forget to create a user.

```

**NOTE:** To allow clients to surf the Internet, make sure that there are permissive rules, such as:

```sh
/ip firewall filter
add chain=forward action=accept src-address=192.168.252.0/24 \
  out-interface-list=WAN place-before=0
add chain=forward action=accept in-interface-list=WAN \
  dst-address=192.168.252.0/24 place-before=1
/ip firewall nat
add chain=srcnat src-address=192.168.252.0/24 out-interface-list=WAN \
  action=masquerade
```

## Add a new user

```sh
# Add a new user and generate/export certs
#
# Change variables below if needed then copy the whole script
# and paste into MikroTik terminal window.
#

:global CN [/system identity get name]
:global USERNAME "user"
:global PASSWORD "password"

## add a user
/ppp secret
add name=$USERNAME password=$PASSWORD profile=VPN-PROFILE service=ovpn

## generate a client certificate
/certificate
add name=client-template-to-issue copy-from=client-template \
  common-name="$USERNAME@$CN"
sign client-template-to-issue ca="$CN" name="$USERNAME@$CN"
:delay 10

## export the CA, client certificate, and private key
/certificate
export-certificate "$CN" export-passphrase=""
export-certificate "$USERNAME@$CN" export-passphrase="$PASSWORD"

## Done. You will find the created certificates in Files.

```

## Setup OpenVPN client

1. Copy the exported certificates from the MikroTik

    ```sh
    sftp admin@MikroTik_IP:cert_export_\*
    ```

    Also, you can download the certificates from the web interface or Winbox.
    Open Winbox/WebFig → <kbd>Files</kbd> for this.


2. Create `user.auth` file

    The file auth.auth holds your username/password combination. On the first
    line must be the username and on the second line your password.

    ```
    user
    password
    ```

3. Create OpenVPN config that named like `USERNAME.ovpn`:

    ```sh
    client
    dev tun
    proto tcp-client
    remote MikroTik_IP 1194
    nobind
    persist-key
    persist-tun
    cipher AES-128-CBC
    auth SHA1
    pull
    verb 2
    mute 3

    # Create a file 'user.auth' with a username and a password
    #
    # cat << EOF > user.auth
    # user
    # password
    # EOF
    auth-user-pass user.auth

    # Copy the certificates from MikroTik and change
    # the filenames below if needed
    ca cert_export_MikroTik.crt
    cert cert_export_user@MikroTik.crt
    key cert_export_user@MikroTik.key

    # Uncomment the following line if Internet access is needed
    #redirect-gateway def1

    # Add routes to networks behind MikroTik
    #route 192.168.88.0 255.255.255.0
    ```

4. Try to connect

    ```
    sudo openvpn USERNAME.ovpn
    ```

## Decrypt private key to avoid password asking (optional)

```
openssl rsa -passin pass:password -in cert_export_user@MikroTik.key -out cert_export_user@MikroTik.key
```

## Delete a user and revoke his certificate

```sh
# Delete a user and revoke his certificate
#
# Change variables below and paste the script
# into MikroTik terminal window.
#

:global CN [/system identity get name]
:global USERNAME "user"

## delete a user
/ppp secret
remove [find name=$USERNAME profile=VPN-PROFILE]

## revoke a client certificate
/certificate
issued-revoke [find name="$USERNAME@$CN"]

## Done.

```

## Revert OpenVPN server configuration on MikroTik

```sh
## Revert OpenVPN configuration
/interface ovpn-server server
set enabled=no default-profile=default port=1194

/ip pool
remove [find name=VPN-POOL]

/ppp secret
remove [find profile=VPN-PROFILE]

/ppp profile
remove [find name=VPN-PROFILE]

/ip firewall filter
remove [find comment="Allow OpenVPN"]
remove [find comment="Accept DNS requests from VPN clients"]

/certificate
## delete the certificates manually
```
## OpenVPN Server Setup for MikroTik 7.X 


:global CN [/system identity get name]
:global PORT 1194

:global DAYSVALIDCA 9450
## it does not make sense for me that the SERVER is bigger/longer than CA which is used to sign the server cert
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

## 2022.04.21 - it it really limited to 8characters?
## it is being used to secure the private key
:global PASSWORDCERTPASSPHRASE "12345678"


## generate a CA certificate
/certificate
add name=ca-template common-name="$CN" days-valid="$DAYSVALIDCA" \
  key-usage=crl-sign,key-cert-sign
sign ca-template ca-crl-host=127.0.0.1 name="$CN"
:delay 10

## generate a server certificate
/certificate
add name=server-template common-name="ovpn-server@$CN" days-valid="$DAYSVALIDSERVER" \
  key-usage=digital-signature,key-encipherment,tls-server
sign server-template ca="$CN" name="ovpn-server@$CN"
:delay 10

## create a client template
/certificate
add name="$OVPNCLIENTCERTIFICATETEMPLATENAME" common-name="$OVPNCLIENTCERTIFICATETEMPLATECOMMONNAME" \ days-valid="$DAYSVALIDCLIENT" key-usage=tls-client
:delay 5

## create IP pool
/ip pool
add name="$OVPNDHCPPOOLNAME" ranges="$OVPNDHCPPOOLRANGE"

## add VPN profile
/ppp profile
add local-address="$OVPNIPADDRESS" name="$OVPNPROFILENAME" \
  remote-address="$OVPNDHCPPOOLNAME" use-ipv6=no use-upnp=no use-compression=no use-mpls=no use-encryption=yes only-one=yes

## setup OpenVPN server
/interface ovpn-server server
add certificate="ovpn-server@$CN" cipher=blowfish128,aes128-cbc,aes192-cbc,aes256-cbc \
 comment=openvpn default-profile="$OVPNPROFILENAME" disabled=no  name=OpenVpn-Server


## setup OpeVPN interface binding
/interface ovpn-server
add name="$USERNAME" user="$USERNAME"

## add a firewall rule
/ip firewall filter
add chain=input action=accept dst-port="$PORT" protocol=tcp \
  comment="OpenVPN - Allow" place-before=0
add chain=input action=accept dst-port=53 protocol=udp src-address="$OVPNNETWORK"  \
  comment="OpenVPN - Accept DNS requests from clients" place-before=1


## add a user
/ppp secret
add local-address="$OVPNIPADDRESS" name="$USERNAME" \
  password="$PASSWORDUSERLOGIN" profile="$OVPNPROFILENAME" \ remote-address="$OVPNCLIENTIPADDRESS" service=ovpn

## generate a client certificate
/certificate
add name=client-template-to-issue copy-from="$OVPNCLIENTCERTIFICATETEMPLATENAME" \
  common-name="$USERNAME@$CN"
sign client-template-to-issue ca="$CN" name="$USERNAME@$CN"
:delay 10

/certificate
set "ovpn-server@$CN" trusted=yes

## export the CA, client certificate, and private key
/certificate
export-certificate "$CN" export-passphrase=""
export-certificate "$USERNAME@$CN" export-passphrase="$PASSWORDCERTPASSPHRASE"

## clear the console history to get rid of sensitive information
/console clear-history

Copy
sftp admin@1.2.3.4:cert_export_\*


Config

client
dev tun
proto tcp-client
remote 1.2.3.4 1194
nobind
persist-tun
cipher AES-256-CBC
verb 5
mute 3
redirect-gateway def1


# Provide user authentication details via an external file (user.auth)
auth-user-pass user.auth

# Specify certificates and keys
ca cert_export_MikroTik.crt
cert cert_export_ovpn-Client1@MikroTik.crt
key cert_export_ovpn-Client1@MikroTik.key

# Uncomment to route all traffic through VPN
#redirect-gateway def1

# Add routes to networks behind the MikroTik router
#route 192.168.252.0 255.255.255.0  # Adjust if needed


