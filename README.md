Configure WireGuard VPN on Debian 12
Preparation
Install necessary packages
Generate cryptographic keys
Enable IP Forwarding
Install packages
shell
sudo apt update
sudo apt install wireguard procps iptables -y
Use code with caution.
Generate Keys
shell
cd /etc/guard/
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
Use code with caution.
Repeat this step on both Server and Client. Keep privatekey secret.
Server Configuration
Enable IPv4 Forwarding
shell
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
Use code with caution.
Create server config
shell
sudo nano /etc/wireguard/wg0.conf
Use code with caution.
ini
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address = 10.8.0.1/24
ListenPort = 51820

# NAT Routing (Replace ens33 with your interface name from 'ip route')
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE

[Peer]
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.8.0.2/32
Use code with caution.
Client Configuration
Create client config
shell
sudo nano /etc/wireguard/wg0.conf
Use code with caution.
ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.8.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_EXTERNAL_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
Use code with caution.
Service Management
Start and Enable
shell
# Turn on the tunnel
sudo wg-quick up wg0

# Enable autostart on boot
sudo systemctl enable wg-quick@wg0
Use code with caution.
Check Status
shell
# Detailed WireGuard status
sudo wg show

# Check if NAT is working (should show Server IP)
curl ifconfig.me
