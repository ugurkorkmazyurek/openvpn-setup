#!/bin/bash

# Bu script Ubuntu 20.04 sunucusuna OpenVPN kurulumu ve konfigürasyonu yapar.

# OpenVPN ve Easy-RSA kurulumunu yapıyoruz
apt update
apt install -y openvpn easy-rsa

# Easy-RSA varsayılan dizinine geçiyoruz
make-cadir ~/openvpn-ca
cd ~/openvpn-ca

# CA (Certificate Authority) oluşturmak için gerekli dosyaları yapılandırıyoruz
cat > vars << EOF
set_var EASYRSA_REQ_COUNTRY    "US"
set_var EASYRSA_REQ_PROVINCE   "California"
set_var EASYRSA_REQ_CITY       "San Francisco"
set_var EASYRSA_REQ_ORG        "Copyleft Certificate Co"
set_var EASYRSA_REQ_EMAIL      "me@example.net"
set_var EASYRSA_REQ_OU         "MyOrganizationalUnit"
EOF

# CA oluşturuyoruz
./easyrsa init-pki
./easyrsa build-ca nopass

# Sunucu sertifikası ve anahtarı oluşturuyoruz
./easyrsa gen-req server nopass
./easyrsa sign-req server server

# Diffie-Hellman parametreleri ve TLS-Auth anahtarı oluşturuyoruz
./easyrsa gen-dh
openvpn --genkey --secret ta.key

# İstemci sertifikası ve anahtarı oluşturuyoruz
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1

# Sunucu konfigürasyon dosyasını oluşturuyoruz
cat > /etc/openvpn/server.conf << EOF
port 1194
proto udp
dev tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh.pem
auth SHA256
tls-auth /etc/openvpn/ta.key 0
topology subnet
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
log-append /var/log/openvpn.log
verb 3
EOF

# Sertifikaları, anahtarları ve gerekli dosyaları uygun dizinlere taşıyoruz
cp pki/ca.crt pki/private/server.key pki/issued/server.crt /etc/openvpn
cp pki/dh.pem ta.key /etc/openvpn

# OpenVPN sunucusunu başlatıyoruz ve sistem başlatma sırasında çalışmasını sağlıyoruz
systemctl start openvpn@server
systemctl enable openvpn@server

# İstemci konfigürasyon dosyasını oluşturuyoruz
cat > ~/client1.ovpn << EOF
client
dev tun
proto udp
remote 128.140.91.164 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
auth SHA256
cipher AES-256-CBC
key-direction 1
<ca>
$(cat pki/ca.crt)
</ca>
<cert>
$(cat pki/issued/client1.crt)
</cert>
<key>
$(cat pki/private/client1.key)
</key>
<tls-auth>
$(cat ta.key)
</tls-auth>
EOF

echo "OpenVPN kurulumu tamamlandı. İstemci dosyası ~/client1.ovpn olarak kaydedildi."
