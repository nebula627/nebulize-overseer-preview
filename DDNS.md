> Public documentation preview. No installer or runtime is provided here; setup commands apply to an already installed private build.

# Connect DDNS notifications

1. Choose either the bundled updater or your existing DDNS updater. Run it on the network whose public IP should be published. Never run two updaters for the same record.
2. Set the game hostname to a DNS-only A record. HTTP proxying is not a generic game-server proxy. Check that the hostname resolves to one public IPv4 address.
3. Run `sudo bash /opt/nebulize-overseer/setup.sh` and enter the hostname, game port and status title. To use the bundled updater, answer yes to its prompt and supply your Cloudflare zone ID, existing A-record ID and API token with DNS Edit permission restricted to that zone. The token is entered privately and mounted only into the updater container. Leave it disabled to keep your existing updater.
4. In the website manager open **DDNS notifications**, choose the existing status-message channel and optional IP-change announcement channel, enable notifications and save. The bot needs View Channel, Send Messages, Embed Links and Read Message History in the status channel. No channels are created by setup.
5. On the first check, Overseer posts its own status message and records a baseline. It checks every minute and updates when its Refresh status button is used. Later public-IP changes update the same message and optionally announce the change. Last address change stays unrecorded until an actual change is observed.

The bundled updater checks every minute using two independent public IPv4 services. If they disagree or fail, it leaves DNS unchanged. It verifies the exact hostname, record ID, A type and DNS-only setting before updating only the record content; TTL and other settings stay intact. Errors back off automatically. It updates an existing record and does not create records, change proxy settings, configure router ports, or create Cloudflare tunnels. IPv6 updating is not included.

Find the zone ID on your domain's Cloudflare overview. Get the A-record ID from Cloudflare's DNS records API using your own token, or your existing updater's configuration. Never paste that token into chat, website JavaScript, or a shell command containing literal credentials.

To inspect the proposed update without changing DNS, run after setup:

```bash
sudo docker compose -f /opt/nebulize-overseer/compose.yaml --profile ddns run --rm ddns-updater --ddns-check
```

Use this before starting the updater if you need to verify the selected record. setup.sh starts it when you enable it, so leave it disabled until your old updater is stopped and you are ready. Updater logs: `sudo docker compose -f /opt/nebulize-overseer/compose.yaml --profile ddns logs --tail 30 ddns-updater`.

Discord notifications read public DNS independently and need no Cloudflare key. CNAME chains are supported when they return one final A address; multiple A addresses are rejected. DNS caches/TTL may delay announcement detection. A public-IP change must actually occur for a change announcement; the first check establishes a baseline.

Never reuse Nebulizer Galaxy's zone, API token or production bot token. A cloud VM will publish its own egress IP, so use a VM on the game server's Internet connection for home-server DDNS.

Cloudflare dynamic DNS: https://developers.cloudflare.com/dns/manage-dns-records/how-to/managing-dynamic-ip-addresses/
