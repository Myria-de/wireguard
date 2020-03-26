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
**Nur für IPv4**
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
**Für IPv4 und IPv6 mit zwei Clients**
```
[Interface]
Address = 10.66.66.1/24,fd42:42:42::1/64
ListenPort = 51820
PrivateKey = MC1GuT/RBjcXEn66fDtoML+/QG86dsD9pSm1g1ar6ns=
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE
[Peer]
PublicKey = 
AllowedIPs = 10.66.66.2/32,fd42:42:42::2/128
PresharedKey = 

[Peer]
PublicKey = 
AllowedIPs = 10.66.66.3/32,fd42:42:42::2/128
PresharedKey = 
```
Alle IP-Adressen sind Beispiele. Die IP-Adresse ist frei wählbar, sollte aber – wie in unserem Beispiel – aus einem privaten Adressbereich nach RFC 1918 stammen. Allerdings besteht die Gefahr von Adresskonflikten, wenn derselbe Bereich zufällig auch im gerade genutzten WLAN zum Einsatz kommt. Eine Alternative ist eine IP-Adresse gemäß RFC 6596, beispielsweise „100.64.0.1/10“ (siehe Beispiel für IPv4), die ebenfalls nicht über das Internet geroutet wird und damit als sicher gelten kann.

Name der Netzwerkschnittstelle ermitteln: 
```
ip addr
```
## VPN-Client einrichten
**Client-Konfiguration erzeugen**
```
sudo mkdir /etc/wireguard
(umask 077 && printf "[Interface]\nPrivateKey = " | sudo tee /etc/wireguard/wg0.conf > /dev/null)
wg genkey | sudo tee -a /etc/wireguard/wg0.conf | wg pubkey | sudo tee /etc/wireguard/publickey
```
**Client-Konfiguration**
```
[Interface]
Address = 100.64.0.2/32
PrivateKey = [der schon vorhandene private Schlüssel des Cliens]

[Peer]
PublicKey = yIDlao0TEyGYuv7AWiRzn4VW6Obi1y3nSLhJrOZlc3s=
AllowedIPs = 0.0.0.0/0 #kompletter Netzwerkzugriff
#Zugriff nur auf Server und Heimnetzwerk
#AllowedIPs = 100.64.0.1/32, 192.168.178.0/24
Endpoint = domain.meinserver.de:51820
```
## Wireguard starten
Auf dem Server und Client:
```
sudo wg-quick up wg0
```
Infos zur Verbindung:
```
wg show
```
Wireguard beenden:
```
sudo wg-quick down wg0
```
Wenn Wireguard automatisch starten soll, beenden Sie den Server zuerst mit „wg-quick down wg0“ und führen dann diese Befehlszeile aus:
```
sudo systemctl enable --now wg-quick@wg0
```
## QR-Code für die Smartphone-Konfiguration erzeugen
Auf einem Client-PC starten:
```
sudo qrencode -o conf.png < /etc/wireguard/wg0.conf
```
## Links
Wireguard: https://www.wireguard.com/

Open-VPN-Konfiguration: https://www.pcwelt.de/40450

CIDR to IPv4 Conversion: https://www.ipaddressguide.com/cidr

Öffentliche IP-Adresse ermitteln: https://www.whatsmyip.org

VPN-CLient-Software: https://tunsafe.com


