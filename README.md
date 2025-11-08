# WireGuard Server Setup (VPS)

## 1. Install WireGuard
```bash
apt update
apt install wireguard
```

## 2. Generate Server Keys
```bash
cd /etc/wireguard
wg genkey | tee server_private.key | wg pubkey | tee server_public.key
```

## 3. Enable IP Forwarding
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

## 4. Create Config File
```bash
nano /etc/wireguard/wg0.conf
```
```ini
[Interface]
PrivateKey = [CONTENT OF server_private.key]
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Add [Peer] sections below as you add clients
```

## 5. Open Firewall Port
```bash
ufw allow 51820/udp
ufw allow 22/tcp
ufw enable
```

## 6. Start WireGuard
```bash
wg-quick up wg0
systemctl enable wg-quick@wg0
```

## 7. Check Status
```bash
wg show
```

## Adding Clients

For each client, add to `/etc/wireguard/wg0.conf`:
```ini
[Peer]
PublicKey = [CLIENT'S PUBLIC KEY]
AllowedIPs = 10.0.0.X/32  # X = 2,3,4 etc for each client
```

Then reload:
```bash
wg-quick down wg0 && wg-quick up wg0
```

## Server Public Key for Clients
```bash
cat /etc/wireguard/server_public.key
```
Give this to clients for their configs.
