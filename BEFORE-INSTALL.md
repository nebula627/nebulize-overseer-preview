> Public documentation preview. No installer or runtime is provided here; setup commands apply to an already installed private build.

# Before you install: first-time setup

Follow this guide before running the invite-only installation instructions supplied to approved testers. You need your own accounts and credentials; the package does not include Nebulizer Galaxy's accounts or settings. Labels in service dashboards can change; official instructions are linked below.

## 1. Choose what you need

| Feature | Prepare before installation | Required? |
| --- | --- | --- |
| Discord bot | Discord application, bot token, server ID, bot invitation | Yes |
| Private download | Accepted GitHub repository invitation | Yes |
| Host | Dedicated Ubuntu/Debian x86-64 VM, Internet connection, sudo access | Yes |
| Website manager | HTTPS domain, reverse proxy, Supabase or an existing auth backend | Only for website controls |
| Discord website linking | OAuth client ID/secret and configured authentication provider | Only for website linking |
| Server status | Uptime Kuma instance and public status page | Optional |
| DDNS notifications | Existing game hostname and game port | Optional |
| Cloudflare DDNS updater | Cloudflare domain, existing DNS-only A record, scoped API token, zone/record IDs | Optional |

The installer first starts the bot. You then run `sudo bash /opt/nebulize-overseer/setup.sh` to configure selected integrations. The wizard saves settings and starts the selected containers; external accounts, OAuth provider configuration, HTTPS routing, DNS records and Discord channels remain your responsibility. You can skip optional features and run the wizard again later.

## 2. Prepare your VM (approved testers)

1. Runtime distribution is invite-only and is not included in this public preview.
2. Create a dedicated x86-64 VM and install Ubuntu 22.04/24.04 or Debian 12/13. This preview is not an ARM package.
3. Log into its terminal with an account allowed to use `sudo`. Run `uname -m` and confirm it reports `x86_64`; run `cat /etc/os-release` to check the distribution.
4. Confirm the VM has working Internet access. Give it a stable LAN address if a reverse proxy or tunnel will connect to it.
5. The private installation instructions installs GitHub CLI and Docker if Docker is absent. If you already use Docker, check `sudo docker compose version` works. Do not use the fresh-install command to upgrade an existing Overseer installation.

The bot alone needs outbound connectivity, not an inbound public bot port. For home-server DDNS, put the updater VM on the same Internet connection as the server whose address it publishes. A remote cloud VM would detect the cloud VM's address instead.

## 3. Create your Discord bot

1. Sign into the [Discord Developer Portal](https://discord.com/developers/applications), select **New Application**, give it a name and create it. Save its application/client ID from General Information.
2. Open **Bot**. Generate/reset its token when ready and keep it in your password manager. The installer asks for this token using hidden input. Resetting it invalidates the previous token.
3. Enable **Server Members Intent** and **Message Content Intent** under privileged gateway intents, then save. These support member lookup and message-based modules. Larger/verified applications may need Discord approval for these intents.
4. In Discord, enable **Developer Mode** under User Settings → Advanced. Right-click your server and choose **Copy Server ID**. This is the number the installer asks for; it is not your application ID or a channel ID.
5. In the application’s **OAuth2 URL Generator**, select `bot` and `applications.commands`. Choose permissions for the modules you plan to use. Open the generated URL, select your server, and authorize the bot using an account allowed to add apps.
6. In Discord Server Settings → Roles, move the bot's role above the roles and members it will manage. Discord's role hierarchy still applies even when a dashboard permission is granted.

Choose permissions according to use:

| Tools | Discord permissions to consider |
| --- | --- |
| Messages, polls, status and ticket logs | View Channels, Send Messages, Embed Links, Read Message History; Attach Files for file/transcript delivery |
| Message cleanup and moderation | Manage Messages |
| Timeouts | Moderate Members |
| Kicks / bans | Kick Members / Ban Members |
| Level rewards and role tools | Manage Roles |
| Webhook tools | Manage Webhooks |
| Invite-source tracking | Manage Server for invite inspection; attribution can be uncertain for some join paths |
| Modules that create or change channels | Manage Channels, only when those modules are used |

Channel permission overrides can deny access despite the server-level grant. You do not need to select Administrator as a shortcut. Configure existing channels through the manager after installation; setup does not create them.

Official reference: [Discord bot setup](https://docs.discord.com/developers/quick-start/getting-started).

## 4. Prepare website login with Supabase

Skip this section if you do not need the website manager. Supabase is the guided sign-in route; an existing login system needs the developer adapter described in [WEBSITE.md](WEBSITE.md).

1. Create your own project at [Supabase](https://supabase.com/dashboard). Save the project URL and its **public publishable/anon key** from the project connection/API settings. Never substitute a secret or service-role key.
2. In Authentication → Sign In / Providers → Discord, copy the displayed callback URL.
3. Return to your Discord application → OAuth2. Add that callback under Redirects and save. Copy the application's client ID and client secret into Supabase's Discord provider form, enable the provider and save.
4. Decide your manager URL, for example `https://community.example.com/overseer/manager/index.html`. In Supabase Authentication → URL Configuration, add that exact URL to Redirect URLs and set the appropriate Site URL.
5. If members will link Discord to an existing account, enable manual identity linking in Supabase Auth settings. Use your existing website's Supabase project to link existing users; a new project has separate accounts.
6. In the wizard choose `supabase`, enter your project URL/public key and website origin, for example `https://community.example.com`. The origin has no `/overseer/` path.

Keep these three credentials distinct:

| Credential | Where it goes |
| --- | --- |
| Discord bot token | Installer's hidden bot-token prompt |
| Discord OAuth client secret | Supabase Discord provider or your trusted authentication backend |
| Supabase public publishable/anon key | Wizard's website configuration |

Official reference: [Supabase Discord provider](https://supabase.com/docs/guides/auth/social-login/auth-discord). First sign into the manager as the Discord server owner. After installation, use **Discord role permissions** to delegate specific tools to other roles. Website permission grants are distinct from Discord's native permissions; both must allow the operation.

## 5. Prepare your HTTPS website address

For a guided walkthrough, follow [Website access: step-by-step tutorial](WEBSITE-SETUP.md).

You can prepare the domain and proxy before installation, but the manager itself will respond only after installation and website setup.

1. Choose a domain/subdomain you control. Arrange a valid HTTPS certificate and a working reverse proxy, or use an existing Cloudflare Tunnel.
2. For the documented Nginx route, use an existing HTTPS Nginx server on the bot VM. Add the `/overseer/` location from [WEBSITE.md](WEBSITE.md); it forwards to `http://127.0.0.1:8093/`. Validate with `sudo nginx -t` before reloading.
3. If the proxy or tunnel runs elsewhere, its localhost is a different machine. Arrange a private route to the bot VM and restrict the API bind/firewall to that proxy. Follow WEBSITE.md rather than exposing port 8093 directly to the Internet.
4. Use the same origin in the wizard and the same manager URL in Supabase's redirect list. A dedicated tunnel hostname must likewise be configured as an allowed origin and OAuth redirect destination.

If you have no domain/proxy yet, install the bot with website access disabled and finish this section later. Automatic provisioning of domains, certificates, tunnels and arbitrary existing website logins is not included.

## 6. Prepare Uptime Kuma (optional)

If you do not already run Kuma, follow its [official installation instructions](https://github.com/louislam/uptime-kuma#-how-to-install) on a suitable host before continuing.

1. Open Kuma and add monitors for the services you want to track. Confirm they report the expected state.
2. Create a **Status Page**, choose a slug such as `servers`, add the desired monitors to its groups and save/publish it.
3. Put that page behind a working HTTPS hostname. Open `https://status.example.com/status/servers` in a signed-out browser and confirm the chosen monitors appear. Do not supply the Kuma admin dashboard URL.
4. Save that public page URL for the wizard. Overseer needs access to the public page and its JSON/heartbeat routes; a login wall on those routes prevents monitor discovery.
5. After installation select the monitors and notification channel under **Server status**. No Kuma admin password is needed.

See [UPTIME-KUMA.md](UPTIME-KUMA.md) for verification and [Kuma's status-page guide](https://github.com/louislam/uptime-kuma/wiki/Status-Page) for page setup.

## 7. Prepare Cloudflare DDNS (optional)

For notification-only use, keep your existing updater and prepare its hostname/game port. Enable the bundled updater only if you want this installation to update your record.

1. Add your domain to Cloudflare and complete its domain setup if it is not already active there.
2. In DNS → Records, create or identify an **A** record for your game hostname, such as `game.example.com`. Set it to **DNS only**. The bundled updater requires an existing IPv4 A record; it does not create one or update AAAA records.
3. Copy the domain's **Zone ID** from its overview page.
4. Open your Cloudflare profile → API Tokens → Create Token. Use the DNS-edit template or a custom token with **Zone → DNS → Edit**, restricted to this domain's zone. Store the token privately. See [Cloudflare's token guide](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/).
5. Obtain the record ID from your existing updater's configuration, or list the zone's records with the [Cloudflare DNS records API](https://developers.cloudflare.com/api/resources/dns/subresources/records/methods/list/). Match both the exact hostname and type A. The record ID is different from the zone ID.

If you need to list the ID from a Linux terminal, this example reads the token privately and passes it to curl through standard input, rather than putting a literal token in the command. Replace the non-secret placeholders first:

```bash
read -r -p 'Zone ID: ' cf_zone
read -r -p 'Exact game hostname: ' cf_name
read -r -s -p 'Cloudflare API token (hidden): ' cf_token; printf '\n'
printf 'header = "Authorization: Bearer %s"\n' "$cf_token" | curl --fail --silent --show-error --config - --get "https://api.cloudflare.com/client/v4/zones/$cf_zone/dns_records" --data-urlencode "type=A" --data-urlencode "name=$cf_name"
unset cf_token
```

Find the matching object in `result` and save its `id`. A failed request or empty result means you should check the token's zone restriction, zone ID and hostname before proceeding.

6. Stop any old updater for this same record before answering yes to the wizard's updater prompt: setup starts the updater when enabled. Enter the zone ID, record ID and hidden token there.
7. Configure the Discord status/announcement channels after installation using [DDNS.md](DDNS.md). For notification-only use, no Cloudflare token is needed.

DDNS updates DNS only. Router port forwarding, game-server firewall rules, CGNAT limitations and game hosting are separate. Confirm you can reach the game server using its address before assuming a successful DNS update makes the server reachable.

## 8. Ready-to-install checklist

- [ ] Accepted the GitHub invitation and can see Releases.
- [ ] Have a supported dedicated VM and sudo access.
- [ ] Created/invited my own Discord bot and saved its server ID/token privately.
- [ ] Enabled the required intents and positioned the bot role appropriately.
- [ ] For website access: prepared HTTPS routing, auth provider, callback and redirect URLs.
- [ ] For Kuma: prepared a publicly readable HTTPS `/status/slug` page.
- [ ] For the bundled updater: prepared the correct network, record/zone IDs and scoped token, and stopped the previous updater.

Approved testers can now follow their private installation instructions. After setup, check bot logs, sign in as owner, configure channels and test with a limited-access account. The wizard does not require optional integration information when that feature is skipped.
