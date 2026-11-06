# Pingstack

The `pingstack` MCP server connects to one Pingstack account. Pingstack monitors cron jobs and scheduled tasks (heartbeat checks), websites, SSL certificates, domain renewals, DNS records and TCP ports, and alerts notification groups when something fails or recovers.

- Call `get_account_capabilities` first. It returns the account, plan limits and usage, the fields each check kind accepts and the user's time zone.
- Send alerts to existing notification groups: call `list_notification_groups` and pass their ids as `notification_group_ids`.
- `create_check` and `create_dashboard` need an `idempotency_key`. Reuse it when retrying the same request.
- Heartbeat ping URLs from `get_check_ping_urls` are secrets. Keep them out of logs, commits and chat.
- Pass `time_zone` to the history, runs, metrics and uptime tools, and stay within the plan's `history_days` and `run_history_days`.
- On `quota_exceeded`, say the change didn't happen and give the limit. Share `billing.upgrade_url` only when it isn't null.
- Show the user a proposed check or dashboard before creating it.

The first time you use the server, run `/mcp auth pingstack` to sign in.
