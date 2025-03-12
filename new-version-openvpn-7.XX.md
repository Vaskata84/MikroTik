<h2><span style="color: #99cc00;">OpenVPN Server and certificate on MikroTik 7x&nbsp;</span></h2>
<pre><code class="language-plaintext">
<span dir="ltr" lang="shell">:global CN [/system identity get name]</span>
<span dir="ltr" lang="shell">:global PORT 1194</span>

<span dir="ltr" lang="shell">:global DAYSVALIDCA 9450</span>
<span dir="ltr" lang="shell" style="color: #99cc00;">## it does not make sense for me that the SERVER is bigger/longer than CA which is used to sign the server cert</span>
<span dir="ltr" lang="shell">:global DAYSVALIDSERVER 9450</span>
<span dir="ltr" lang="shell">:global DAYSVALIDCLIENT 3650</span>

<span dir="ltr" lang="shell">:global OVPNCLIENTCERTIFICATETEMPLATENAME "ovpn-ClientTemplate"</span>
<span dir="ltr" lang="shell">:global OVPNCLIENTCERTIFICATETEMPLATECOMMONNAME "ovpnClientTemplate"</span>
<span dir="ltr" lang="shell">:global OVPNPROFILENAME "ovpn-profile"</span>
<span dir="ltr" lang="shell">:global OVPNNETWORKMASK "24"</span>
<span dir="ltr" lang="shell">:global OVPNNETWORK "10.0.6.0/24"</span>
<span dir="ltr" lang="shell">:global OVPNIPADDRESS "10.0.6.254"</span>
<span dir="ltr" lang="shell">:global OVPNCLIENTIPADDRESS "10.0.6.253"</span>

<span dir="ltr" lang="shell">:global OVPNDHCPPOOLNAME "ovpn-dhcpPool"</span>
<span dir="ltr" lang="shell">:global OVPNDHCPPOOLRANGE "10.0.6.249-10.0.6.253"</span>

<span dir="ltr" lang="shell">:global USERNAME "ovpn-Client1"</span>
<span dir="ltr" lang="shell">:global PASSWORDUSERLOGIN "ovpn-Client1-Password"</span>

<span style="color: #99cc00;"><span dir="ltr" lang="shell">## 2022.04.21 - it it really limited to 8characters?</span>
<span dir="ltr" lang="shell">## it is being used to secure the private key</span></span>
<span dir="ltr" lang="shell">:global PASSWORDCERTPASSPHRASE "12345678"</span>


<span dir="ltr" lang="shell" style="color: #99cc00;">## generate a CA certificate</span>
<span dir="ltr" lang="shell">/certificate</span>
<span dir="ltr" lang="shell">add name=ca-template common-name="$CN" days-valid="$DAYSVALIDCA" \</span>
<span dir="ltr" lang="shell">  key-usage=crl-sign,key-cert-sign</span>
<span dir="ltr" lang="shell">sign ca-template ca-crl-host=127.0.0.1 name="$CN"</span>
<span dir="ltr" lang="shell">:delay 10</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## generate a server certificate</span>
<span dir="ltr" lang="shell">/certificate</span>
<span dir="ltr" lang="shell">add name=server-template common-name="ovpn-server@$CN" days-valid="$DAYSVALIDSERVER" \</span>
<span dir="ltr" lang="shell">  key-usage=digital-signature,key-encipherment,tls-server</span>
<span dir="ltr" lang="shell">sign server-template ca="$CN" name="ovpn-server@$CN"</span>
<span dir="ltr" lang="shell">:delay 10</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## create a client template</span>
<span dir="ltr" lang="shell">/certificate</span>
<span dir="ltr" lang="shell">add name="$OVPNCLIENTCERTIFICATETEMPLATENAME" common-name="$OVPNCLIENTCERTIFICATETEMPLATECOMMONNAME" \ days-valid="$DAYSVALIDCLIENT" key-usage=tls-client</span>
<span dir="ltr" lang="shell">:delay 5</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## create IP pool</span>
<span dir="ltr" lang="shell">/ip pool</span>
<span dir="ltr" lang="shell">add name="$OVPNDHCPPOOLNAME" ranges="$OVPNDHCPPOOLRANGE"</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## add VPN profile</span>
<span dir="ltr" lang="shell">/ppp profile</span>
<span dir="ltr" lang="shell">add local-address="$OVPNIPADDRESS" name="$OVPNPROFILENAME" \</span>
<span dir="ltr" lang="shell">  remote-address="$OVPNDHCPPOOLNAME" use-ipv6=no use-upnp=no use-compression=no use-mpls=no use-encryption=yes only-one=yes</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## setup OpenVPN server</span>
<span dir="ltr" lang="shell">/interface ovpn-server server</span>
<span dir="ltr" lang="shell">add certificate="ovpn-server@$CN" cipher=blowfish128,aes128-cbc,aes192-cbc,aes256-cbc \</span>
<span dir="ltr" lang="shell"> comment=openvpn default-profile="$OVPNPROFILENAME" disabled=no  name=OpenVpn-Server</span>


<span dir="ltr" lang="shell" style="color: #99cc00;">## setup OpeVPN interface binding</span>
<span dir="ltr" lang="shell">/interface ovpn-server</span>
<span dir="ltr" lang="shell">add name="$USERNAME" user="$USERNAME"</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## add a firewall rule</span>
<span dir="ltr" lang="shell">/ip firewall filter</span>
<span dir="ltr" lang="shell">add chain=input action=accept dst-port="$PORT" protocol=tcp \</span>
<span dir="ltr" lang="shell">  comment="OpenVPN - Allow" place-before=0</span>
<span dir="ltr" lang="shell">add chain=input action=accept dst-port=53 protocol=udp src-address="$OVPNNETWORK"  \</span>
<span dir="ltr" lang="shell">  comment="OpenVPN - Accept DNS requests from clients" place-before=1</span>


<span dir="ltr" lang="shell" style="color: #99cc00;">## add a user</span>
<span dir="ltr" lang="shell">/ppp secret</span>
<span dir="ltr" lang="shell">add local-address="$OVPNIPADDRESS" name="$USERNAME" \</span>
<span dir="ltr" lang="shell">  password="$PASSWORDUSERLOGIN" profile="$OVPNPROFILENAME" \ remote-address="$OVPNCLIENTIPADDRESS" service=ovpn</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## generate a client certificate</span>
<span dir="ltr" lang="shell">/certificate</span>
<span dir="ltr" lang="shell">add name=client-template-to-issue copy-from="$OVPNCLIENTCERTIFICATETEMPLATENAME" \</span>
<span dir="ltr" lang="shell">  common-name="$USERNAME@$CN"</span>
<span dir="ltr" lang="shell">sign client-template-to-issue ca="$CN" name="$USERNAME@$CN"</span>
<span dir="ltr" lang="shell">:delay 10</span>

<span dir="ltr" lang="shell">/certificate</span>
<span dir="ltr" lang="shell">set "ovpn-server@$CN" trusted=yes</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## export the CA, client certificate, and private key</span>
<span dir="ltr" lang="shell">/certificate</span>
<span dir="ltr" lang="shell">export-certificate "$CN" export-passphrase=""</span>
<span dir="ltr" lang="shell">export-certificate "$USERNAME@$CN" export-passphrase="$PASSWORDCERTPASSPHRASE"</span>

<span dir="ltr" lang="shell" style="color: #99cc00;">## clear the console history to get rid of sensitive information</span>
<span dir="ltr" lang="shell">/console clear-history</span>
</code></pre>
<h2><span style="color: #99cc00;">Copy</span></h2>
<pre><code class="language-plaintext"><span dir="ltr" lang="shell">sftp admin@1.2.3.4:cert_export_\*</span></code></pre>
<h1><span style="color: #99cc00;">config</span></h1>
<pre><code class="language-plaintext">
<span dir="ltr" lang="shell">client</span>
<span dir="ltr" lang="shell">dev tun</span>
<span dir="ltr" lang="shell">proto tcp-client</span>
<span dir="ltr" lang="shell">remote 1.2.3.4 1194</span>
<span dir="ltr" lang="shell">nobind</span>
<span dir="ltr" lang="shell">persist-tun</span>
<span dir="ltr" lang="shell">cipher AES-256-CBC</span>
<span dir="ltr" lang="shell">verb 5</span>
<span dir="ltr" lang="shell">mute 3</span>
<span dir="ltr" lang="shell">redirect-gateway def1</span>


<span dir="ltr" lang="shell" style="color: #99cc00;"># Provide user authentication details via an external file (user.auth)</span>
<span dir="ltr" lang="shell">auth-user-pass user.auth</span>

<span dir="ltr" lang="shell" style="color: #99cc00;"># Specify certificates and keys</span>
<span dir="ltr" lang="shell">ca cert_export_MikroTik.crt</span>
<span dir="ltr" lang="shell">cert cert_export_ovpn-Client1@MikroTik.crt</span>
<span dir="ltr" lang="shell">key cert_export_ovpn-Client1@MikroTik.key</span>

<span dir="ltr" lang="shell" style="color: #99cc00;"># Uncomment to route all traffic through VPN</span>
<span dir="ltr" lang="shell">#redirect-gateway def1</span>

<span dir="ltr" lang="shell" style="color: #99cc00;"># Add routes to networks behind the MikroTik router</span>
<span dir="ltr" lang="shell">#route 192.168.252.0 255.255.255.0  # Adjust if needed</span></code></pre>
