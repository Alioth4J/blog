---
title: OpenVPN Quick Set
description: Follow the white rabbit
date: 2025-05-15
slug: openvpn
image: 
categories:
    - Network
---

## Steps
### Server Configuration
#### Install Dependencies
```bash
yum upgrade 
yum install epel-release
yum install openvpn easy-rsa
```

#### Generating Certificates and Keys
```bash
cd /etc/openvpn
mkdir easy-rsa
cp -r /usr/share/easy-rsa/3/* easy-rsa/
cd easy-rsa
vi vars
# Example content of the file
# set_var EASYRSA_REQ_COUNTRY    "CT"
# set_var EASYRSA_REQ_PROVINCE   "Province"
# set_var EASYRSA_REQ_CITY       "City"
# set_var EASYRSA_REQ_ORG        "ExampleOrg"
# set_var EASYRSA_REQ_EMAIL      "root@example.com"
# set_var EASYRSA_REQ_OU         "ExampleOU"
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh
openvpn --genkey --secret ta.key
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

#### OpenVPN Configuration
```bash
vi /etc/openvpn/server.conf
```

Note that no matter TCP or UDP and what port is used, it's unable to use OpenVPN directly. The reason is clear.  

Despite this, please do not use default port as someone may actively probes server ports. **TCP** and **443 Port** or **UDP** and **53 Port** are appropriate.  

In this config file:  
```txt
port 53
proto udp
dev tun

ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem

tls-auth /etc/openvpn/easy-rsa/ta.key 0

server 10.8.0.0 255.255.255.0

push "redirect-gateway def1 bypass-dhcp"

keepalive 10 120

client-to-client

persist-key
persist-tun
log /dev/null
status /dev/null
verb 0
```

Record no log.  

#### Ip Forward
```bash
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```

#### Iptables
```bash
iptables -t nat -A POSTROUTING -o eth0 -s 10.8.0.0/24 -j MASQUERADE
```

#### Firewall
I do not use firewall...  

#### Starting the Service
```bash
systemctl start openvpn@server
systemctl enable openvpn@server
```

To check the status of the service:  
```bash
systemctl status openvpn@server
```

### Client Configuration
#### Getting Certificates and Keys
Use `scp` command to get them from the server.  
They are:  
- /etc/openvpn/easy-rsa/pki/ca.crt
- /etc/openvpn/easy-rsa/pki/issued/client1.crt
- /etc/openvpn/easy-rsa/pki/private/client1.key
- /etc/openvpn/easy-rsa/ta.key

#### SELinux
OS: Silverblue  

```bash
sudo rpm-ostree kargs --append enforcing=0
```

Another way:  
```bash
sudo rpm-ostree kargs --editor
```
and then add `enforcing=0`.  

#### Simply Using GUI to Connect
GNOME Gui:  
`Settings -> Network -> VPN -> Add VPN -> OpenVPN`  

Most configurations are in `Identity`.  
```
Gateway: server ip
Authentication Type: Certificates; and select the corresponding files in the following lines
Advanced -> General -> Use custom gateway port 443; Use a TCP connection
Advanced -> TLS Authentication -> Additional TLS authentication or encryption -> Mode: TLS auth; Key File: Select ta.key; Key Direction: 1
```

Now a connnection should be successful.  

## Toubleshooting
### IPv6
If the server does not have an ipv6 address, simply disable ipv6 to prevent ipv6 leak.  

### Command Line to Connect.
So it's clear to see the logs.  

client.conf:  
```txt
client
proto <tcp|udp>
dev tun
remote <ip> <port> 
nobind

ca ca.crt
cert client1.crt
key client1.key

tls-auth ta.key 1

keepalive 10 120

redirect-gateway def1

persist-key
persist-tun

verb 4
```

then  
```
sudo openvpn --config client.conf
```
### Checking ip route
```bash
ip route
```

