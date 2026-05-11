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

## Configure Port Forwarding

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

Device / LAN IP: the local IP of your Raspberry Pi (e.g., 192.168.1.100). One thing that is worth is to give a static IP to your PI so you can use it here

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


## Configure Wireguard (using Docker)

1. Create a docker-compose file with

```
volumes:
  etc_wireguard:

services:
  wg-easy:
    environment:
      - PORT=51821
      - INSECURE=true
      - WG_HOST=YOURDOMAIN.duckdns.org
      - WG_DEFAULT_DNS=1.1.1.1
    
    privileged: true

    image: ghcr.io/wg-easy/wg-easy:15
    container_name: wg-easy
    volumes:
      - ./etc_wireguard:/etc/wireguard
      - /lib/modules:/lib/modules:ro
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      net.ipv4.ip_forward: "1"
      net.ipv4.conf.all.src_valid_mark: "1"

```

2. Start the container

> docker compose up -d

3. Log in, configure an account with your duckdns url

4. Download the Wireguard app on your mobile

5. Add a client by scanning your QR code and you are all setup with it.
