# Configure DuckDNS with Wireguard

## Configure DuckDNS

1. Go to https://www.duckdns.org/, log in with one of available methods and create a new domain.

2. Then create a folder where it will sit duckDNS script to update the domain to your current IP every 5mins.

> nano ~/duckdns/duck.sh

with

> echo url="https://www.duckdns.org/update?domains=YOU_DOMAIN&token=YOUR_TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -

YOUR_DOMAIN needs to be just the domain, not the full url YOUR_DOMAIN.duckdns.org

3. Make it executable:

> chmod +x ~/duckdns/duck.sh

4. Add cron job to update every 5 minutes:

> crontab -e

5. Add this line:

> */5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1


6. Test it:

> ~/duckdns/duck.sh


You should see OK inside ~/duckdns/duck.log.


## Configure Wireguard (using Docker)

1. Create a docker-compose file with

```
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SERVERURL=myrasppi.duckdns.org     # your DuckDNS domain goes HERE
      - SERVERPORT=51820
      - PEERS=peer_name1,peer_name2                            # how many client configs to generate
      - PEERDNS=1.1.1.1
      - INTERNAL_SUBNET=10.8.0.0
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
    restart: unless-stopped
```

2. Start the container

> docker compose up -d

3. Get the QR Codes for each peer by getting the logs

> docker compose logs

4. Port Forwarding

On your router, forward:

> UDP 51820 → Raspberry Pi LAN IP

If you use Vodafone, here's how

<details>

<summary>General Steps to Do Port Forwarding on a Vodafone Router</summary>

Find Your Router’s IP Address
Usually it’s something like 192.168.1.1.

Log Into the Router Web Interface

Open a browser and navigate to that IP (e.g., http://192.168.1.1).

Log in with your username/password. (Often the default is on a sticker on the router or it was given by Vodafone.)

Enable “Expert Mode” (if available)
Some Vodafone routers hide advanced options like port forwarding unless you enable expert or advanced mode. 
forum.vodafone.de
+1

Navigate to Port Forwarding / Port Mapping / “Redirecionamento de Portas”
According to Vodafone’s support docs:

Go to Internet → Port Mapping (ou “Mapeamento de Portas IPv4”) 
Apoio a Clientes Vodafone

Or in newer Wi-Fi-6 routers: “Internet” → “IPv4 Port Mapping” 
Apoio a Clientes Vodafone

Create a New Port Forwarding Rule
When adding a rule:

Service name: something like “WireGuard” or “WG”

Protocol: UDP (WireGuard uses UDP)

Device / LAN IP: the local IP of your Raspberry Pi (e.g., 192.168.1.100)

External (Public) Port: 51820 (or whatever port WireGuard listens on)

Internal (LAN) Port: same 51820 (unless you’ve configured it differently)

Source / Origin IP: often you leave this blank to accept traffic from anywhere. 
forum.vodafone.pt

Save / Apply the Rule
After adding the rule, click Save and/or Apply so the router makes the change.

Test the Port

Use a tool like canyouseeme.org or portchecker.co to check if the port is open.

Make sure your Raspberry Pi is running WireGuard while testing.
</details>

5. Download the Wireguard app

6. Add a tunnel, scan your QR code and you are all setup with it.