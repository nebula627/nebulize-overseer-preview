> Public documentation preview. No installer or runtime is provided here; setup commands apply to an already installed private build.

# Website access: step-by-step tutorial

This tutorial connects the included Overseer manager to your own HTTPS website and Discord login. It uses Supabase for authentication and an existing Nginx HTTPS site on the bot VM. For other hosting layouts or a custom login system, see [WEBSITE.md](WEBSITE.md). Prepare the accounts using [BEFORE-INSTALL.md](BEFORE-INSTALL.md), then install the bot using the invite-only installation instructions supplied to approved testers.

The manager is included in the integration package; you do not need to build its pages. Adding a link from an existing website does not automatically connect that website's login session. Reusing an existing custom login requires the adapter described below.

## 1. Write down your URLs

Replace these examples throughout this guide:

| Setting | Example | Where it is used |
| --- | --- | --- |
| Website origin | `https://community.example.com` | Setup wizard; scheme and hostname only |
| Manager URL | `https://community.example.com/overseer/manager/index.html` | Browser link and Supabase Redirect URLs |
| Supabase project URL | `https://YOUR-PROJECT.supabase.co` | Setup wizard |
| Discord OAuth callback | Copy from your Supabase Discord provider form | Discord application Redirects |

Keep the bot token and OAuth client secret separate. Only the public Supabase publishable/anon key goes into website setup. Do not use a service-role key.

## 2. Prepare Discord sign-in

1. Open your Supabase project → Authentication → Sign In / Providers → Discord. Copy its callback URL.
2. Open your own [Discord application](https://discord.com/developers/applications) → OAuth2. Add that callback to Redirects and save.
3. Enter the application's client ID and OAuth client secret in Supabase's Discord provider form, enable it and save. The bot token does not belong here.
4. In Supabase Authentication → URL Configuration, add your exact manager URL to Redirect URLs. Set Site URL to your website's HTTPS origin.
5. To link Discord to existing Supabase accounts, enable manual identity linking in the project's Auth settings. Use the same Supabase project as those accounts.

See the [official Supabase Discord instructions](https://supabase.com/docs/guides/auth/social-login/auth-discord) if a dashboard label differs. No secret needs to be pasted into GitHub or browser JavaScript.

## 3. Enable the manager on the VM

After installing the integration package, run:

```bash
sudo bash /opt/nebulize-overseer/setup.sh
```

Choose website access, enter your website origin, choose `supabase`, and enter your project URL and public publishable/anon key. Optional Kuma and DDNS fields can be skipped if not needed. The wizard saves the configuration and restarts the bot with the website service bound to `127.0.0.1:8093`.

Check the service locally on the bot VM:

```bash
curl --fail --silent --show-error http://127.0.0.1:8093/client-config
```

Expected result: JSON describing the configured public website/auth connection. This endpoint deliberately contains public connection details, not the bot token or OAuth secret. Connection refused means the service has not started; inspect logs:

```bash
sudo docker compose -f /opt/nebulize-overseer/compose.yaml -f /opt/nebulize-overseer/compose.website.yaml logs --tail 50 overseer
```

## 4. Route your HTTPS website to the manager

For this route, Nginx must already serve your domain over HTTPS on the same VM. A static website hosted somewhere else cannot reach this VM's localhost simply by adding a link.

1. Find the active Nginx configuration for your domain, commonly under `/etc/nginx/sites-available/`. Make a backup before editing it.
2. Inside its existing HTTPS `server { ... }` block, add:

```nginx
location /overseer/ {
    proxy_pass http://127.0.0.1:8093/;
    proxy_set_header Host $host;
    proxy_set_header Authorization $http_authorization;
    proxy_set_header Origin $http_origin;
    proxy_read_timeout 30s;
    proxy_buffering off;
    proxy_cache off;
}
```

3. Keep the trailing slash on `proxy_pass`: it removes the `/overseer/` prefix before forwarding to the bot. Do not cache authenticated API responses. This is a location block, not a complete replacement for your website configuration. [Nginx proxy reference](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass).
4. Validate and reload only if validation succeeds:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

5. Open your full manager URL. The manager page should load; its controls still require sign-in and permission checks.

### If your proxy or Cloudflare Tunnel runs on another host

`127.0.0.1` always means that host itself. Use a private network route to the bot VM, with the Compose port bind/firewall restricted to the proxy, or run the tunnel connector locally on the bot VM. A tunnel that forwards straight to port 8093 serves `/manager/index.html` at its hostname; the `/overseer/manager/index.html` example requires the Nginx prefix-removal route above. Update both the wizard origin and Supabase Redirect URL to match the URL you actually serve.

Do not forward your router's public port 8093 to the bot. Domain ownership, certificates and tunnel provisioning depend on your hosting provider and are not created by the wizard. [WEBSITE.md](WEBSITE.md) explains the service connection requirements.

## Coolify: connect an existing website

In the setup wizard, answer **yes** to “Is your existing website hosted with Coolify?” It records your hosting choice and prints the connection checklist. It does not request a Coolify API token or modify a running website.

### Short checklist

Keep your website in Coolify and your bot on its installed VM. You only need to connect them:

1. **Run the wizard.** Enter your website address, choose Supabase or your existing login, and answer yes to Coolify.
2. **Connect the servers privately.** Same Docker host: use a shared Docker network. Separate hosts: use a private LAN/VPN connection. Your website server must be able to reach the bot on port 8093. See the expandable instructions below for this one-time networking step.
3. **Add the website route.** If your website uses Nginx, copy the route in step 4 above and replace `127.0.0.1` with the bot's reachable private address or network name. Save it in your website configuration, validate it, and redeploy through Coolify. Other website servers need an equivalent proxy route.
4. **Open and sign in.** Visit `https://YOUR-WEBSITE/overseer/manager/index.html`, sign in as the Discord server owner, choose role permissions, and add the Manage Discord link to your account menu.

**Quick check:** open `https://YOUR-WEBSITE/overseer/client-config`. JSON means the connection works. A 404 usually means the route is missing; a 502 usually means the website cannot reach the bot.

Your existing website stays in place. The wizard prepares the bot settings; the website route and private connection still need to be configured. Adding the route does not automatically reuse a custom website login—see section 7 for that adapter.

<details>
<summary>Advanced: network connection, proxy details and other hosting layouts</summary>

### Keep the bot on its installed VM

1. Complete website authentication in steps 1–3 above. Use your existing website's HTTPS origin in the wizard.
2. Open your existing website resource in Coolify. Identify how its server serves pages: Nginx, another reverse proxy, or a framework server. A static frontend alone cannot proxy API requests; configure its serving web server or add a server-side gateway.
3. Arrange a private connection from that website container to Overseer. On separate machines, use the bot VM's private LAN/VPN address and restrict its listening interface/firewall to the proxy host. The default localhost port mapping must be deliberately changed for this layout. Do not replace it with an unrestricted public mapping.
4. On the same Docker host, a verified shared Docker network can provide container-to-container access at the bot's internal port 8093. Record the actual bot network alias; do not assume `overseer` resolves across isolated networks. If Coolify manages a Compose resource, its predefined-network option can connect it to the selected destination; this does not join a bot on another machine.
5. In the website's Nginx configuration, use the location block in step 4, replacing `http://127.0.0.1:8093/` with the verified private upstream, for example `http://BOT_PRIVATE_IP:8093/`. For a framework server, implement the equivalent server-side `/overseer/` proxy. Strip that prefix, forward Authorization/Origin, and do not cache authenticated responses.
6. Save the configuration in the website's deployment source or persistent configuration so redeploying will retain it. Validate before deploying the website change. Do not import this documentation-only repository as a bot source build or start a second bot with the same token.
7. Open `https://YOUR-WEBSITE/overseer/client-config`. You should receive the public connection JSON. Then open `https://YOUR-WEBSITE/overseer/manager/index.html` and continue with owner sign-in below.

Coolify's public domain route and the private bot connection are separate. Keep the existing website's domain attached to its website resource; the route above is handled by that website server. A new subdomain is optional, but would need its own origin, redirect URL and manager route configuration.

The downloadable image is currently loaded by the VM installer. Moving the entire bot into a Coolify-managed resource additionally requires making that image available on its destination and retaining configuration, secrets and data mounts; the wizard does not perform that migration.

Official references: [Coolify networking](https://coolify.io/docs/core/networking-in-coolify), [Coolify Docker Compose](https://coolify.io/docs/applications/builds/docker-compose).

</details>

## 5. Sign in as owner and grant access

1. Open the manager and click **Sign in**. Authorize Discord using the account that owns the configured Discord server.
2. Confirm your server information and channel choices load. The server owner is recognized through the verified Discord identity.
3. Open **Discord role permissions**, choose an existing ordinary server role, open its permission picker, select the tools it should access, and save.
4. Grant module viewing separately from module management where offered. Give moderation actions and full configuration only to trusted roles. The bot's native Discord permissions and role position also need to permit each action.
5. Assign the role to the intended member in Discord. Have that member sign in with their own Discord account. Current membership and roles are checked by the server on requests.

The package does not create a new Discord role automatically. You cannot delegate @everyone or integration-managed roles. Never give access by accepting a Discord ID typed into a public form.

## 6. Add the button to your website

For the Nginx layout above, the link is:

```html
<a href="/overseer/manager/index.html">Manage Discord</a>
```

Put it in your account/admin menu. Your site's developer should show it only when the signed-in user's effective access allows the manager, using the authenticated `/overseer/api/access` response. Hiding the link alone is not authorization: the bot validates requests even when someone opens the manager URL directly.

For a separately hosted manager, use its full HTTPS URL instead of a relative link. Different origins can have separate browser login sessions, so members may need to sign in on the manager too.

## 7. Reuse an existing custom website login (developer route)

If you do not use Supabase, choose `custom` in the wizard. Your developer must provide:

- An HTTPS auth endpoint that verifies the supplied bearer session token and returns the server-derived user/verified Discord identity contract in [WEBSITE.md](WEBSITE.md).
- Your website's sign-in URL.
- `/opt/nebulize-overseer/custom-auth.js`, defining `window.OVERSEER_SESSION_PROVIDER` to return your user's access token from the real website auth client. The integration package mounts this adapter into the included manager.

This is an adapter requirement, not a universal automatic connector to every website platform. Do not supply a shared admin token or bot token as a user's session. Existing website roles must be supplied by the trusted backend, not editable profile fields.

## 8. Verify before inviting members

| Test | Expected result |
| --- | --- |
| Server owner signs in | Server details load and owner controls are available |
| Member with limited role signs in | Only delegated tools can be used |
| Member without grants requests a management action | Access denied, including direct API requests |
| Remove a granted Discord role | Grant no longer applies on the next request |
| Save a harmless setting, then refresh | Saved value persists |
| Leave the dashboard open | Data refreshes every minute without submitting unfinished forms |

Use a disposable poll or test channel to check message delivery; avoid bans or disruptive changes as connectivity tests.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Page is 404 | Manager path, Nginx prefix stripping, and whether your installed release includes the manager |
| Proxy returns 502 | Bot container logs, local curl check, and whether the proxy is on the same host |
| Discord says redirect is invalid | Discord application's callback must exactly match Supabase's callback |
| Login returns to the wrong page | Exact manager Redirect URL and Site URL in Supabase |
| Account linking is disabled | Manual identity linking setting and an existing signed-in session |
| Signed in but forbidden | Verified Discord identity, server membership, owner account or delegated role grants |
| Origin rejected | Actual manager origin must match the wizard's configured origin |
| No channels or members | Bot is in the correct server, intents are enabled, and channel/role permissions allow access |

For recovery, rerun the setup wizard with corrected values. It saves a config backup; avoid sharing logs containing credentials. This tutorial configures the included manager, not a new general-purpose website or an automatic import of another site's users and roles.
