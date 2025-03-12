<h2>OpenVPN Server and certificate on MikroTik 7x&nbsp;</h2>
<pre><code class="language-plaintext">
<span lang="shell" dir="ltr">:global CN [/system identity get name]</span>
<span lang="shell" dir="ltr">:global PORT 1194</span>

<span lang="shell" dir="ltr">:global DAYSVALIDCA 9450</span>
<span lang="shell" dir="ltr">## it does not make sense for me that the SERVER is bigger/longer than CA which is used to sign the server cert</span>
<span lang="shell" dir="ltr">:global DAYSVALIDSERVER 9450</span>
<span lang="shell" dir="ltr">:global DAYSVALIDCLIENT 3650</span>

<span lang="shell" dir="ltr">:global OVPNCLIENTCERTIFICATETEMPLATENAME "ovpn-ClientTemplate"</span>
<span lang="shell" dir="ltr">:global OVPNCLIENTCERTIFICATETEMPLATECOMMONNAME "ovpnClientTemplate"</span>
<span lang="shell" dir="ltr">:global OVPNPROFILENAME "ovpn-profile"</span>
<span lang="shell" dir="ltr">:global OVPNNETWORKMASK "24"</span>
<span lang="shell" dir="ltr">:global OVPNNETWORK "10.0.6.0/24"</span>
<span lang="shell" dir="ltr">:global OVPNIPADDRESS "10.0.6.254"</span>
<span lang="shell" dir="ltr">:global OVPNCLIENTIPADDRESS "10.0.6.253"</span>

<span lang="shell" dir="ltr">:global OVPNDHCPPOOLNAME "ovpn-dhcpPool"</span>
<span lang="shell" dir="ltr">:global OVPNDHCPPOOLRANGE "10.0.6.249-10.0.6.253"</span>

<span lang="shell" dir="ltr">:global USERNAME "ovpn-Client1"</span>
<span lang="shell" dir="ltr">:global PASSWORDUSERLOGIN "ovpn-Client1-Password"</span>

<span lang="shell" dir="ltr">## 2022.04.21 - it it really limited to 8characters?</span>
<span lang="shell" dir="ltr">## it is being used to secure the private key</span>
<span lang="shell" dir="ltr">:global PASSWORDCERTPASSPHRASE "12345678"</span>


<span lang="shell" dir="ltr">## generate a CA certificate</span>
<span lang="shell" dir="ltr">/certificate</span>
<span lang="shell" dir="ltr">add name=ca-template common-name="$CN" days-valid="$DAYSVALIDCA" \</span>
<span lang="shell" dir="ltr">  key-usage=crl-sign,key-cert-sign</span>
<span lang="shell" dir="ltr">sign ca-template ca-crl-host=127.0.0.1 name="$CN"</span>
<span lang="shell" dir="ltr">:delay 10</span>

<span lang="shell" dir="ltr">## generate a server certificate</span>
<span lang="shell" dir="ltr">/certificate</span>
<span lang="shell" dir="ltr">add name=server-template common-name="ovpn-server@$CN" days-valid="$DAYSVALIDSERVER" \</span>
<span lang="shell" dir="ltr">  key-usage=digital-signature,key-encipherment,tls-server</span>
<span lang="shell" dir="ltr">sign server-template ca="$CN" name="ovpn-server@$CN"</span>
<span lang="shell" dir="ltr">:delay 10</span>

<span lang="shell" dir="ltr">## create a client template</span>
<span lang="shell" dir="ltr">/certificate</span>
<span lang="shell" dir="ltr">add name="$OVPNCLIENTCERTIFICATETEMPLATENAME" common-name="$OVPNCLIENTCERTIFICATETEMPLATECOMMONNAME" \ days-valid="$DAYSVALIDCLIENT" key-usage=tls-client</span>
<span lang="shell" dir="ltr">:delay 5</span>

<span lang="shell" dir="ltr">## create IP pool</span>
<span lang="shell" dir="ltr">/ip pool</span>
<span lang="shell" dir="ltr">add name="$OVPNDHCPPOOLNAME" ranges="$OVPNDHCPPOOLRANGE"</span>

<span lang="shell" dir="ltr">## add VPN profile</span>
<span lang="shell" dir="ltr">/ppp profile</span>
<span lang="shell" dir="ltr">add local-address="$OVPNIPADDRESS" name="$OVPNPROFILENAME" \</span>
<span lang="shell" dir="ltr">  remote-address="$OVPNDHCPPOOLNAME" use-ipv6=no use-upnp=no use-compression=no use-mpls=no use-encryption=yes only-one=yes</span>

<span lang="shell" dir="ltr">## setup OpenVPN server</span>
<span lang="shell" dir="ltr">/interface ovpn-server server</span>
<span lang="shell" dir="ltr">add certificate="ovpn-server@$CN" cipher=blowfish128,aes128-cbc,aes192-cbc,aes256-cbc \</span>
<span lang="shell" dir="ltr"> comment=openvpn default-profile="$OVPNPROFILENAME" disabled=no  name=OpenVpn-Server</span>


<span lang="shell" dir="ltr">## setup OpeVPN interface binding</span>
<span lang="shell" dir="ltr">/interface ovpn-server</span>
<span lang="shell" dir="ltr">add name="$USERNAME" user="$USERNAME"</span>

<span lang="shell" dir="ltr">## add a firewall rule</span>
<span lang="shell" dir="ltr">/ip firewall filter</span>
<span lang="shell" dir="ltr">add chain=input action=accept dst-port="$PORT" protocol=tcp \</span>
<span lang="shell" dir="ltr">  comment="OpenVPN - Allow" place-before=0</span>
<span lang="shell" dir="ltr">add chain=input action=accept dst-port=53 protocol=udp src-address="$OVPNNETWORK"  \</span>
<span lang="shell" dir="ltr">  comment="OpenVPN - Accept DNS requests from clients" place-before=1</span>


<span lang="shell" dir="ltr">## add a user</span>
<span lang="shell" dir="ltr">/ppp secret</span>
<span lang="shell" dir="ltr">add local-address="$OVPNIPADDRESS" name="$USERNAME" \</span>
<span lang="shell" dir="ltr">  password="$PASSWORDUSERLOGIN" profile="$OVPNPROFILENAME" \ remote-address="$OVPNCLIENTIPADDRESS" service=ovpn</span>

<span lang="shell" dir="ltr">## generate a client certificate</span>
<span lang="shell" dir="ltr">/certificate</span>
<span lang="shell" dir="ltr">add name=client-template-to-issue copy-from="$OVPNCLIENTCERTIFICATETEMPLATENAME" \</span>
<span lang="shell" dir="ltr">  common-name="$USERNAME@$CN"</span>
<span lang="shell" dir="ltr">sign client-template-to-issue ca="$CN" name="$USERNAME@$CN"</span>
<span lang="shell" dir="ltr">:delay 10</span>

<span lang="shell" dir="ltr">/certificate</span>
<span lang="shell" dir="ltr">set "ovpn-server@$CN" trusted=yes</span>

<span lang="shell" dir="ltr">## export the CA, client certificate, and private key</span>
<span lang="shell" dir="ltr">/certificate</span>
<span lang="shell" dir="ltr">export-certificate "$CN" export-passphrase=""</span>
<span lang="shell" dir="ltr">export-certificate "$USERNAME@$CN" export-passphrase="$PASSWORDCERTPASSPHRASE"</span>

<span lang="shell" dir="ltr">## clear the console history to get rid of sensitive information</span>
<span lang="shell" dir="ltr">/console clear-history</span>
</code></pre>
<h2>Copy</h2>
<pre><code class="language-plaintext"><span lang="shell" dir="ltr">sftp admin@1.2.3.4:cert_export_\*</span></code></pre>
<h1>config</h1>
<pre><code class="language-plaintext">
<span lang="shell" dir="ltr">client</span>
<span lang="shell" dir="ltr">dev tun</span>
<span lang="shell" dir="ltr">proto tcp-client</span>
<span lang="shell" dir="ltr">remote 1.2.3.4 1194</span>
<span lang="shell" dir="ltr">nobind</span>
<span lang="shell" dir="ltr">persist-tun</span>
<span lang="shell" dir="ltr">cipher AES-256-CBC</span>
<span lang="shell" dir="ltr">verb 5</span>
<span lang="shell" dir="ltr">mute 3</span>
<span lang="shell" dir="ltr">redirect-gateway def1</span>


<span lang="shell" dir="ltr"># Provide user authentication details via an external file (user.auth)</span>
<span lang="shell" dir="ltr">auth-user-pass user.auth</span>

<span lang="shell" dir="ltr"># Specify certificates and keys</span>
<span lang="shell" dir="ltr">ca cert_export_MikroTik.crt</span>
<span lang="shell" dir="ltr">cert cert_export_ovpn-Client1@MikroTik.crt</span>
<span lang="shell" dir="ltr">key cert_export_ovpn-Client1@MikroTik.key</span>

<span lang="shell" dir="ltr"># Uncomment to route all traffic through VPN</span>
<span lang="shell" dir="ltr">#redirect-gateway def1</span>

<span lang="shell" dir="ltr"># Add routes to networks behind the MikroTik router</span>
<span lang="shell" dir="ltr">#route 192.168.252.0 255.255.255.0  # Adjust if needed</span></code></pre>
