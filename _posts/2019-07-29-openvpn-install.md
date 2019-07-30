---
layout: post
title:  "Ubuntu 18.04 安装 Openvpn"
categories: how-to openvpn
---
### Ubuntu 18.04 安装使用openvpn

1. 安装openvpn程序，修改配置文件。ca.crt, server.crt, server.key, dh.pem, ta.key这个五个文件将在使用EasyRSA时生成
```bash
# cat /etc/issue
Ubuntu 18.04.2 LTS  \n \l
# apt-get update && apt-get -y install openvpn
# cd ~
# wget -P https://raw.githubusercontent.com/OpenVPN/openvpn/2b8aec62d5db2c17d5d4052991bc18272748bf29/sample/sample-config-files/server.conf
# mv server.conf /etc/openvpn/
# sed -i -e 's!port 1194!port 11094!g' /etc/openvpn/server.conf
# sed -i -e 's!ca ca.crt!ca /etc/openvpn/server/ca.crt!g' /etc/openvpn/server.conf
# sed -i -e 's!cert server.crt!cert /etc/openvpn/server/server.crt!g' /etc/openvpn/server.conf
# sed -i -e 's!key server.key!key /etc/openvpn/server/server.key!g' /etc/openvpn/server.conf
# sed -i -e 's!dh2048.pem!/etc/openvpn/server/dh.pem!g' /etc/openvpn/server.conf
# sed -i -e '/redirect-gateway/s!\;push!push!g' /etc/openvpn/server.conf
# sed -i -e '/dhcp-option/s/\;push/push/g' /etc/openvpn/server.conf
# sed -i -e '/tls-auth/s!ta.key!/etc/openvpn/server/ta.key!g' /etc/openvpn/server.conf
# sed -i -e '/user nobody/s/\;user/user/g' /etc/openvpn/server.conf
# sed -i -e '/group nobody/s/\;group nobody/group nogroup/g' /etc/openvpn/server.conf
# sed -i -e '/net.ipv4.ip_forward/s/#net/net/g' /etc/sysctl.conf && sysctl -p
# iptables -t nat -A POSTROUTING -s 10.8.0.0/16 -o eth0 -j MASQUERADE #eth0 needed to change depend your system
```

配置文件后对应的项应为：
>port 1194
>ca /etc/openvpn/server/ca.crt
>cert /etc/openvpn/server/server.crt
>key /etc/openvpn/server/server.key
>dh /etc/openvpn/server/dh.pem
>push "redirect-gateway def1 bypass-dhcp"
>push "dhcp-option DNS 208.67.220.220"
>push "dhcp-option DNS 208.67.222.222"
>tls-auth /etc/openvpn/server/ta.key 0
>user nobody
>group nogroup


2. 安装EasyRSA，并生成相关的pki文件

```bash
# cd ~
# wget -P https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.6/EasyRSA-unix-v3.0.6.tgz
# tar xvf EasyRSA-unix-v3.0.6.tgz
# cd EasyRSA-v3.0.6/
# cp vars.example vars
# sed -i -e '/EASYRSA_REQ_COUNTRY/s/#set_var/set_var/g' -i -e '/EASYRSA_REQ_COUNTRY/s/US/CN/g' vars
# sed -i -e '/EASYRSA_REQ_PROVINCE/s/#set_var/set_var/g' -i -e '/EASYRSA_REQ_PROVINCE/s/California/Beijin/g' vars
# sed -i -e '/EASYRSA_REQ_CITY/s/#set_var/set_var/g' -i -e '/EASYRSA_REQ_CITY/s/San Francisco/Beijin/g' vars
# sed -i -e '/EASYRSA_REQ_ORG/s/#set_var/set_var/g' -i -e '/EASYRSA_REQ_ORG/s/Copyleft Certificate Co/My Co/g' vars
# sed -i -e '/EASYRSA_REQ_EMAIL/s/#set_var/set_var/g' -i -e '/EASYRSA_REQ_EMAIL/s/me@example.net/admin@example.com/g' vars
# sed -i -e '/EASYRSA_REQ_OU/s/#set_var/set_var/g' -i -e '/EASYRSA_REQ_OU/s/My Organizational Unit/IT/g' vars
```

修改配置vars文件后，相关项应该为：
>set_var EASYRSA_REQ_COUNTRY     "CN"
>set_var EASYRSA_REQ_PROVINCE    "Beijin"
>set_var EASYRSA_REQ_CITY        "Beijin"
>set_var EASYRSA_REQ_ORG "My Co"
>set_var EASYRSA_REQ_EMAIL       "admin@example.com"
>set_var EASYRSA_REQ_OU          "IT"

使用EasyRSA生成pki认证，并对服务器进行签名认证
```bash
# cd ~/EasyRSA-v3.0.6/
# ./easyrsa init-pki
# ./easyrsa build-ca nopass # if you need encry your ca, remove nopass
# ./easyrsa gen-req server nopass
# ./easyrsa sign-req server server
# ./easyrsa gen-dh
# openvpn --genkey --secret ta.key
# cp ~/EasyRSA-v3.0.6/pki/dh.pem ~/EasyRSA-v3.0.6/pki/ca.crt ~/EasyRSA-v3.0.6/pki/issued/server.crt ~/EasyRSA-v3.0.6/pki/private/server.key ~/EasyRSA-v3.0.6/ta.key  /etc/openvpn/server # copy 5 files to config dir
```

3. 启动openvpn
- 前台运行，方便排错
```bash
# openvpn --config /etc/openvpn/server.conf
```

- 后台服务运行
```bash
# sed -i -e 's/#AUTOSTART="all"/AUTOSTART="server"/g' /etc/default/openvpn
# systemctl daemon-reload
# service openvpn start
```

4. 生成客户端配置

```bash
# cd ~/EasyRSA-v3.0.6/
# ./easyrsa gen-req my-client nopass
# ./easyrsa sign-req client my-client
# mkdir -p ~/vpn-client-config/my-client
# cp ~/EasyRSA-v3.0.6/pki/issued/my-client.crt ~/EasyRSA-v3.0.6/pki/private/my-client.key ~/EasyRSA-v3.0.6/pki/ca.crt ~/EasyRSA-v3.0.6/ta.key ~/vpn-client-config/my-client
# wget -P https://raw.githubusercontent.com/OpenVPN/openvpn/2b8aec62d5db2c17d5d4052991bc18272748bf29/sample/sample-config-files/client.conf
# cd ~/vpn-client-config/my-client
# sed -i -e 's/remote my-server-1 1194/remote ip 1194/g' client.conf #your ownip or servername and port
# sed -i -e 's/cert client.crt/cert my-client.crt/g' client.conf
# sed -i -e 's/key client.key/key my-client/g' client.conf
# cd ~
# tar -cvf config.tar ~/vpn-client-config/my-client/*
```

### 参考资料
https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04
https://github.com/OpenVPN/easy-rsa/blob/master/doc/EasyRSA-Readme.md
https://dzone.com/articles/enabling-openvpn-configuration-autostart-on-ubuntu
https://yq.aliyun.com/articles/517339
