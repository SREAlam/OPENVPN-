# OPENVPN-
This is a step to step guid on how to create a VPN 
# we will be using Openvpn -  It will be installed on linux Ubuntu #

# this will install packages that you need to install the vpn # 
sudo apt install openvpn easy-rsa -y

# config the vpn server - create the dictory and move you into it#
make-cadir ~/openvpn-ca
cd ~/openvpn-ca

# nano is a text editor - easy to use# 

nano vars

scrool down 
edit below that suits you
set_var EASYRSA_REQ_COUNTRY    "US"
set_var EASYRSA_REQ_PROVINCE   "California"
set_var EASYRSA_REQ_CITY       "San Francisco"
set_var EASYRSA_REQ_ORG        "MyOrg"
set_var EASYRSA_REQ_EMAIL      "email@example.com"
set_var EASYRSA_REQ_OU         "MyOrgUnit"

save and exit -> 

# now you need to create the certificate Authority (CA)

./easyrsa init-pki
./easyrsa build-ca

# now generate server certificate #

 ./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh
openvpn --genkey --secret ta.key

# copy the gnerated file to the openvpn dictory  using the cp command #

sudo cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem ta.key /etc/openvpn

# generate the openvpn server config fil - add the following date first command will open the file nano - second is the details you need to add #

sudo nano /etc/openvpn/server.conf

port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
auth SHA256
tls-auth ta.key 0
cipher AES-256-CBC
persist-key
persist-tun
user nobody
group nogroup
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
comp-lzo
max-clients 100
status openvpn-status.log
log-append /var/log/openvpn.log
verb 3

# start the server #

sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server

#  config the vpn client - gnerate the certificate and keys

cd ~/openvpn-ca
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1

# create a client config file #

nano client1.ovpn

client
dev tun
proto udp
remote your_server_ip 1194
resolv-retry infinite
nobind
user nobody
group nogroup
persist-key
persist-tun
remote-cert-tls server
auth SHA256
cipher AES-256-CBC
comp-lzo
verb 3
<ca>
-----BEGIN CERTIFICATE-----
# Paste the contents of ca.crt here
-----END CERTIFICATE-----
</ca>
<cert>
-----BEGIN CERTIFICATE-----
# Paste the contents of client1.crt here
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
# Paste the contents of client1.key here
-----END PRIVATE KEY-----
</key>
<tls-auth>
-----BEGIN OpenVPN Static key V1-----
# Paste the contents of ta.key here
-----END OpenVPN Static key V1-----
</tls-auth>






