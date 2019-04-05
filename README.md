# wireguard
Wireguard configuration examples

## Installation Ubuntu
```
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt update
sudo apt install wireguard qrencode
```
## Installation Raspberry Pi (Raspbian)
```
sudo apt install libmnl-dev build-essential git qrencode
git clone https://git.zx2c4.com/WireGuard
cd WireGuard/src
make
sudo make install
```
## VPN-Server einrichten
```
sudo mkdir /etc/wireguard
(umask 077 && printf "[Interface]\nPrivateKey = " | sudo tee /etc/wireguard/wg0.conf > /dev/null)
wg genkey | sudo tee -a /etc/wireguard/wg0.conf | wg pubkey | sudo tee /etc/wireguard/publickey
```
```
sudo gedit /etc/wireguard/wg0.conf &
```
```
[Interface]
Address = 100.64.0.1/10
ListenPort = 51820
PrivateKey = [der schon vorhandene private Schlüssel des Servers]
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s31f6 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s31f6 -j MASQUERADE
[Peer]
PublicKey = 
AllowedIPs = 100.64.0.2/32
```
