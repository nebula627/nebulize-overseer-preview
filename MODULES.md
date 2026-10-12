# Functions and modules

This catalog describes the current private build's implemented controls. It is a preview, not a guarantee that every external provider is configured or a claim of full parity with Dyno, YAGPDB or MEE6. One Discord server per installation; public multi-tenant hosting and AI services are not included.

| Category / module | Functions |
| --- | --- |
| Overview | Server identity, connection state, member count, recorded messages, voice activity, latency, quick actions and DDNS status |
| Community statistics | Message and voice activity, period selection, daily chart, member rankings; tracking starts when connected |
| Activity logs | Recorded community/moderation events; optional message edit/deletion logging |
| Member moderation | Member search, warnings and warning history, timeouts/removal, kick, ban/unban, ordinary role changes, purge and slowmode |
| Moderation notifications | Embedded member DMs for supported actions and tracked expiry/removal notices; delivery depends on Discord DM permissions |
| AutoMod | Spam and blocked-word filtering, invites/links, excessive mentions/capitalization, message length and attachment filters; edited-message checks |
| Escalation and join protection | Violation thresholds/windows and configured warn/timeout/kick/ban responses; minimum account age protection |
| Channel cleanup | Selected auto-delete channels and scheduled removal of old messages |
| Messages | Channel-selectable announcements and Discord formatting; mention notifications suppressed by default |
| Polls | Single-choice button voting, current totals, message refresh after votes, results and closing |
| Giveaways | Persistent entry buttons, prize, duration, winner count and completion results |
| Schedules and reminders | Timed messages, repeating jobs, reminders and cancellation |
| Tags / replies | Community tags, exact/contains/starts-with auto responses, configured custom replies and custom slash commands |
| Starboard | Selected destination and star threshold for highlighted messages |
| AFK | Member AFK notice with reason, cleared on return |
| Roles and welcomes | Join/leave templates and channels, autorole, extra join roles/delay and self-service role buttons |
| Invite tracking | Observed invite attribution and counts; vanity, expired or ambiguous paths may remain unknown |
| Leveling | Configurable message XP, cooldowns, level requirements and Discord reward roles; rank and XP leaderboard |
| Birthdays and achievements | Opt-in month/day birthdays, opt-out, milestone tracking and configurable birthday/level-up/achievement channels |
| Channel automation | Existing channel counters and templates, join-to-create voice rooms, empty-room cleanup and owner room renaming |
| Voice/text links | Configured voice-channel membership linked to text access |
| Economy and role shop | Private coins, daily rewards, balances, transfers, community points and configured role purchases |
| Utility commands | Random choice, dice rolls, bot status/help and member information |
| Social notifications | Platform/page configuration, destination channels and feed delivery state; provider-specific requirements apply |
| Webhooks | Configurable webhook tools; requires native Manage Webhooks permission |
| Uptime Kuma | Public status-page connection, monitor selection and status-change notifications; one page at a time |
| DDNS notifications | Public DNS checks, persistent IP status message, refresh button, baseline/change history and optional announcements |
| Optional Cloudflare DDNS worker | Existing DNS-only IPv4 A-record updates, scoped secret, two-source IP agreement, validation, dry-run checks and retry backoff |
| Discord forms | Button-launched forms, up to five questions, private response channel and response viewing |
| Support tickets | Ticket status/channel configuration and private website integration for ticket conversations/logging; email/DM delivery needs its providers and linked identities |
| Discord linking | Supabase OAuth or custom verified identity adapter, owner recognition and current Discord membership/roles |
| Role permissions | Per-module View/Manage grants, individual moderation/command grants, searchable five-column permission picker and native-permission reference |
| Website access | Basic/full controls, authenticated API, configured origins, website menu integration, Supabase or custom login adapter |
| Dashboard usability | Role/channel/monitor multi-selects, member search, one-minute refresh and refresh after interactions |
| Themes | Nebula, Midnight, Ocean and Asgard-inspired dashboard themes |
| Configuration | Private wizard for own website/auth, Coolify guidance, Kuma and DDNS settings; persistent storage and separate updater secret |

## Provider and integration limits

Social platform choices include YouTube, Reddit, Twitch, TikTok, X/Twitter, Instagram, Kick, RSS and podcast feeds. A selectable option does not imply a universal credential-free connector: check the configured provider's requirements and delivery status. Ticket website pages and email depend on the host website/backend and mail provider; the bot package does not automatically recreate an arbitrary website's support system. Discord-native permissions and role hierarchy still apply to every action.

See [all registered slash commands](COMMANDS.md), [website setup](WEBSITE-SETUP.md), [Kuma](UPTIME-KUMA.md) and [DDNS](DDNS.md).

Provider status: X/Twitter, Instagram and Kick are configuration placeholders whose API/provider connectors are not implemented in this build. Twitch requires private API credentials; TikTok requires authorized Display API access. YouTube, Reddit and allowed RSS/podcast feeds use their supported feed routes.
