> Public documentation preview. No installer or runtime is provided here; setup commands apply to an already installed private build.

# Connect Uptime Kuma

1. In your own Uptime Kuma instance, create a public **Status Page** and add the monitors you want to show. Use its page URL, such as `https://status.example.com/status/servers`, not the login/dashboard URL.
2. Run `sudo bash /opt/nebulize-overseer/setup.sh` and enter that URL. The wizard records its exact HTTPS origin in `uptime_allowed_origins`. This allowlist prevents dashboard users from selecting unrelated network destinations.
3. Finish website sign-in using WEBSITE.md. Open **Server status** in the manager, enter the same public page URL, and refresh. Published monitors should appear.
4. Choose monitors from the multi-select and the Discord update channel. Choose none to include all published monitors. Enable notifications and save. The bot needs View Channel and Send Messages there. The setup does not create channels.
5. Change a disposable test monitor's status and wait for its next heartbeat and the bot's next minute check. Only state changes after the initial baseline are announced. API errors or stale heartbeats show unknown availability rather than falsely declaring an outage.

Overseer reads the public status page and heartbeat JSON endpoints. It does not need an Uptime Kuma administrator password or API key. Monitors absent from the public page cannot be shown. Cloudflare Access or another login wall must not block these public JSON routes. An instance serving a different API format may require an adapter.

To allow another Kuma instance, add its exact HTTPS origin to config.json's uptime_allowed_origins and restart the bot, then choose its /status/slug page in the manager. One page is selected at a time; multiple monitors on that page are supported. Existing saved page settings take priority over uptime_default_url, so save the new URL in the manager after changing an existing installation.

Uptime Kuma project: https://github.com/louislam/uptime-kuma
