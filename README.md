# wireguard
Wireguard configuration examples

**Ausführliche Anleitung:** https://www.pcwelt.de/article/1181301/vpn_mit_wireguard.html

Alternativ können Sie das Script wireguard-install.sh verwenden (Quelle: https://github.com/angristan/wireguard-install). Es Installiert die nötige Software und erstellt die Konfigurationsdateien.

## Installation Ubuntu

**Update:** WireGuard ist ab Linux-Kernel Version 5.6 im Kernel enthalten und wurde auch in die Kernel der Ubuntu-LTS-Versionen Focal Fossa 20.04 (Kernel 5.4) und Bionic Beaver 18.04 (Kernel 4.15) zurück portiert.
```
sudo apt install wireguard
```

Installation bei älteren Ubuntu-Versionen:
```
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt update
sudo apt install wireguard qrencode
```
## Installation Raspberry Pi (Raspbian)
**Update:** Im aktuellen Raspberry Pi OS mit Kerneln ab Version 5.10 lässt sich Wireguard aus den Standard-Repositorien installieren:
```
sudo apt install wireguard
```
Altere Raspian-Versionen:
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
Der Server muss als Router IPv4- und IPv6-Pakete weiterleiten. Um die Funktion zu aktivieren, führen Sie diese Befehlszeile aus:
```
sudo gedit /etc/sysctl.conf
```
Entfernen Sie die Kommentarzeichen ("#") vor "net.ipv4.ip_forward=1" und "net.ipv6.conf.all.forwarding=1". Mit 
```
sysctl -p
```
wenden Sie die Änderungen ohne Neustart an.

**Server-Konfiguration nur für IPv4 mit einem Client**
```
[Interface]
Address = 10.66.66.1/24
ListenPort = 51820
PrivateKey = [der schon vorhandene private Schlüssel des Servers]
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s31f6 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s31f6 -j MASQUERADE

[Peer]
PublicKey = 
AllowedIPs = 10.66.66.2/32
PresharedKey =
```
**Server-Konfiguration für IPv4 und IPv6 mit zwei Clients**
```
[Interface]
Address = 10.66.66.1/24,fd42:42:42::1/64
ListenPort = 51820
PrivateKey = [der schon vorhandene private Schlüssel des Servers]
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
printf "[Interface]\nPrivateKey = " | tee ~/wg0-client1.conf > /dev/null
wg genkey | tee -a ~/wg0-client1.conf | wg pubkey | tee ~/client1_publickey
printf "[Peer]\nPresharedKey = " | tee -a ~/wg0-client1.conf > /dev/null
wg genpsk | tee -a ~/wg0-client1.conf
```
**Client-Konfiguration (IPv4 und IPv6)**
```
[Interface]
Address = 10.66.66.2/24, fd42:42:42::2/64
PrivateKey = [der schon vorhandene private Schlüssel des Clients]
DNS = 2001:4860:4860::8888, 2001:4860:4860::8844, 8.8.8.8, 8.8.4.4
[Peer]
PublicKey = [Schlüssel aus der Datei "/etc/wireguard/publickey" des Wireguard-Servers]
AllowedIPs = 0.0.0.0/0,::/0 #kompletter Netzwerkzugriff ipv4 und ipv6
#Zugriff nur auf Server und Heimnetzwerk
#AllowedIPs = 10.66.66.1/32, 192.168.178.0/24
Endpoint = domain.meinserver.de:51820
```
Kopieren Sie die Client-Konfigurationsdatei "wg0-client1.conf" in den Ordner "/etc/wireguard".

**Server-Konfiguration ergänzen**
Ergänzen Sie auf dem Server in der Datei "/etc/wireguard/wg0.conf" unter "[Peer]" hinter "PublicKey=" den öffentlichen Schlüssel des Clients aus der Datei 
```
~/client1_publickey
```
sowie den "PresharedKey" aus der Datei
```
~/wg0-client1.conf.
```
Bei mehreren Clients vervielfältigen Sie den Abschnitt "[Peer]" und tragen den öffentlichen Schlüssel des jeweiligen Clients ein. Hinter "AllowedIPs=" passen Sie die IP-Adresse mit "10.66.66.3/32", "10.66.66.3/32" und so weiter an. 

## Wireguard starten
Auf dem Server:
```
sudo wg-quick up wg0
```
Auf dem Client-PC:
```
sudo wg-quick up wg0-client1
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
qrencode -t ansiutf8 -l L < ~/wg0-client1.conf
```
In der Smartphone-App scannen Sie den QR-Code ein und aktiveren danach die VPN-Verbindung.

## Alternative Open VPN

```
wget -O pivpn https://install.pivpn.io
bash pivpn
```
Konfigurationsdatei erstellen:
```
pivpn add
```
Auf dem Client-PC:
```
sudo openvpn --config [ovpn-Datei]
```

## Links
Wireguard: https://www.wireguard.com/

Open-VPN-Konfiguration: https://www.pcwelt.de/article/1149871/openvpn__sicher_unterwegs_in_unsicheren_netzen-verschluesselt_surfen.html

CIDR to IPv4 Conversion: https://www.ipaddressguide.com/cidr

Öffentliche IP-Adresse ermitteln: https://www.whatsmyip.org

VPN-CLient-Software: https://tunsafe.com


