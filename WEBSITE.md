> Public documentation preview. No installer or runtime is provided here; setup commands apply to an already installed private build.

# Connect your own website

The private preview supports one Discord server per installation. Each tester uses their own bot, website, authentication service and data volume. It does not use the Nebulizer Galaxy Supabase project.

## Guided configuration

For an existing Coolify website, answer yes to the wizard's Coolify question and follow [the Coolify walkthrough](WEBSITE-SETUP.md#coolify-connect-an-existing-website). It guides private connectivity and website routing; it does not automatically modify your Coolify resources.

For a guided walkthrough, follow [Website access: step-by-step tutorial](WEBSITE-SETUP.md).

After installation, run `sudo bash /opt/nebulize-overseer/setup.sh`. Enter your HTTPS website origin, choose `supabase` or `custom`, supply the requested public connection details, and optionally enter your Uptime Kuma page and DDNS hostname. A backup of config.json is saved. The bot restarts with port 8093 bound to localhost only. The wizard does not change DNS, enable a provider, or create Discord channels.

## HTTPS routing

Use an existing HTTPS website reverse proxy on the bot VM. Add this Nginx location inside its existing HTTPS server block:

```nginx
location /overseer/ {
    proxy_pass http://127.0.0.1:8093/;
    proxy_set_header Host $host;
    proxy_set_header Authorization $http_authorization;
    proxy_set_header Origin $http_origin;
    proxy_read_timeout 30s;
    proxy_buffering off;
}
```

Run `sudo nginx -t` before reloading your existing Nginx service. Open `https://YOUR-WEBSITE/overseer/manager/index.html`. Link this URL from your website's account menu. All dashboard actions require server-verified sign-in; hiding a menu link is not authorization.

If your reverse proxy runs on another machine, localhost will not be reachable there. Use a private network route or tunnel to the VM and restrict access to the proxy host. Change the Compose bind deliberately; do not publish an unauthenticated HTTP port to the Internet. Existing Cloudflare Tunnel installations can route an HTTPS hostname to the local service, but the hostname serving the manager must also be in `website_origins`.

## Supabase: easiest sign-in option

1. Use your own Supabase project. In the wizard enter its HTTPS project URL and **public publishable/anon key**. Never enter a service-role/secret key.
2. In your own Discord Developer Portal application, open OAuth2. Add `https://YOUR-PROJECT.supabase.co/auth/v1/callback` as a redirect URI.
3. In Supabase Authentication → Sign In / Providers → Discord, enable Discord and enter your Discord OAuth2 client ID and client secret there. The OAuth secret is different from the bot token. Keep it in Supabase, not the website JavaScript or bot config.
4. In Supabase Authentication → URL Configuration, add the exact manager URL `https://YOUR-WEBSITE/overseer/manager/index.html` to Redirect URLs. Set your Site URL appropriately.
5. Open the manager and click Sign in. Complete Discord authorization using the server owner's Discord account. Supabase validates the token; the bot reads the verified Discord identity. The Discord server owner gets owner control for this installation. To link Discord to an already signed-in Supabase account, enable manual identity linking in your project's Auth settings; the manager uses linkIdentity for an existing session and normal Discord sign-in when signed out.
6. Open **Discord role permissions**, select an ordinary role, click its permission picker, choose permissions and save. Members sign in through the same Discord provider and must be current members of this bot's server. Their current Discord roles are fetched for each request.

Separate `view` and `manage` grants exist for each module, plus warn/timeout/kick/ban actions and individual Discord commands. Grant only the intended tools. Full control and permission delegation are reserved for the owner unless your trusted authentication service explicitly grants broader website permissions. @everyone and managed integration roles cannot be delegated. Revocation applies on the next API request.

Optional website roles can be maintained by your trusted server in Supabase `app_metadata.roleIds` and `app_metadata.permissions`; supported website permission names are `manageDiscordBot`, `controlDiscordBot`, `manageSupportTickets`, and owner role ID `owner`. Never put authorization in `user_metadata`, which account holders can edit. The bot ignores it for permissions.

Official provider steps: https://supabase.com/docs/guides/auth/social-login/auth-discord

## Connect an existing authentication system

Choose `custom` and set `website_auth_url` to your own HTTPS token-verification endpoint. The bot calls it with the browser's `Authorization: Bearer <website-session-token>` on every request, with redirects disabled. The endpoint must validate the token and return server-derived identity and permissions:

```json
{"user":{"id":"stable-account-id","displayName":"Member","discordId":"VERIFIED_DISCORD_ID","roleIds":[],"permissions":[]}}
```

Return 401 for invalid/expired tokens. `discordId` must come from a verified OAuth link, never a client-submitted form or editable profile field. For the installation owner return roleIds `["owner"]`; basic administrators may receive `["manageDiscordBot"]` in permissions, while full configuration uses `controlDiscordBot`. Linked role grants work with empty website permissions too. This verification endpoint must not call the bot to enrich its response, which would create an authentication loop.

For your existing frontend, call `/overseer/api/access` with that token to obtain effective permissions before showing links. Call `/overseer/api/overview` to load the server and channels. Other resources use `/overseer/api/RESOURCE`, GET for reads and POST with JSON for actions.

To reuse the included manager with custom authentication, edit `/opt/nebulize-overseer/custom-auth.js` to define `window.OVERSEER_SESSION_PROVIDER = async () => YOUR_EXISTING_AUTH_CLIENT.getAccessToken()`. It is mounted read-only into the manager and loaded before integration-client.js. Supply/import your actual session client; no shared bot token or administrator API key belongs in the browser. Set website_login_url to your existing sign-in page. Custom sign-out is handled by that website. This adapter requires your website developer to connect its existing login client.

## Test and recover

Test as owner, then a member with one limited role, then a member without any grant. A member without access must get 403 for management actions even when requesting the URL directly. Remove the limited role and verify access is revoked. Refreshes occur every minute and after actions; Discord interaction changes also trigger the dashboard's change watcher.

If sign-in fails, check Discord's Supabase callback URI and Supabase's manager Redirect URL. If access fails, check your linked Discord identity, current guild membership, configured origin, and role grants. Inspect logs with `sudo docker compose -f /opt/nebulize-overseer/compose.yaml logs --tail 50` without sharing tokens. Restore config.json.bak if needed and restart with both Compose files.

The server application is packaged without raw Python source. HTML/CSS/JavaScript delivered to a browser is necessarily visible; the downloadable manager contains no bot token or OAuth client secret. Ticket email delivery and third-party API modules still require their own provider configuration.
