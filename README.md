# Pingstack MCP server

<img src="assets/icon-512.png" alt="Pingstack" width="96" align="right">

[Pingstack](https://pingstack.io) monitors cron jobs and scheduled tasks with
heartbeat checks, and checks websites, TCP ports, DNS records, SSL
certificates and domain expiry. Its MCP server lets your coding agent set up
that monitoring and read the results for you.

Ask your agent to add a heartbeat check for the backup job it just wrote, with
the right cron schedule, time zone and grace period. Ask which checks had
incidents last week, why, and how long they were down. Ask it to build a
dashboard for a client, publish it as a status page and set up who gets
alerted.

This repository holds the setup files and directory listings for the hosted
server. The server itself runs at Pingstack, so there's nothing to install or
run locally.

| | |
| --- | --- |
| Endpoint | `https://app.pingstack.io/mcp` |
| Transport | Streamable HTTP (stateless, JSON responses) |
| Sign-in | OAuth in your browser, or an API key |
| Price | Free for 15 checks. MCP is included on every plan |
| Docs | [docs.pingstack.io/mcp](https://docs.pingstack.io/mcp/overview/) |

## Connect

You need a Pingstack account. [Sign up free](https://app.pingstack.io/signup),
then add the server to your client. Leave out any Authorization header: the
client opens your browser, an Owner or Admin picks the Pingstack account, and
you approve the connection. Clients register themselves, so there's no client
ID or secret to copy.

**Claude Code**

```sh
claude mcp add --transport http pingstack https://app.pingstack.io/mcp
```

Then run `/mcp`, choose `pingstack` and sign in. Add `--scope user` to use it
in every project. You can also install it as a plugin, which adds the
[skills](#skills) below:

```sh
/plugin marketplace add atomicvan/pingstack-mcp
/plugin install pingstack@pingstack
```

**Claude (claude.ai and Claude Desktop)**

Go to Settings → Connectors, choose **Add custom connector**, name it
Pingstack and enter `https://app.pingstack.io/mcp`.

**GitHub Copilot CLI**

```sh
copilot mcp add --transport http pingstack https://app.pingstack.io/mcp
```

If `/mcp` shows `needs-auth`, run `/mcp auth pingstack`.

**VS Code with Copilot**

Run **MCP: Add Server** from the Command Palette, choose **HTTP** and enter
the URL. Or add it to `.vscode/mcp.json`:

```json
{
  "servers": {
    "pingstack": {
      "type": "http",
      "url": "https://app.pingstack.io/mcp"
    }
  }
}
```

**Cursor**

Add this to `~/.cursor/mcp.json`, then sign in from Cursor's MCP settings:

```json
{
  "mcpServers": {
    "pingstack": {
      "url": "https://app.pingstack.io/mcp"
    }
  }
}
```

**OpenAI Codex**

```sh
codex mcp add pingstack --url https://app.pingstack.io/mcp
codex mcp login pingstack
```

**Google Antigravity**

Install the plugin in the `antigravity` folder with the Antigravity CLI. The
app picks it up too. You sign in in the browser the first time it connects:

```sh
git clone https://github.com/atomicvan/pingstack-mcp
agy plugin install ./pingstack-mcp/antigravity
```

Or add this to `~/.gemini/config/mcp_config.json`, then click **Authenticate**
next to `pingstack` under Agent settings → Customizations:

```json
{
  "mcpServers": {
    "pingstack": {
      "serverUrl": "https://app.pingstack.io/mcp"
    }
  }
}
```

**Gemini CLI**

Install it as an extension:

```sh
gemini extensions install https://github.com/atomicvan/pingstack-mcp
```

Then run `/mcp auth pingstack`.

The [client setup guide](https://docs.pingstack.io/mcp/clients/) covers
each client in more detail, plus how to use an
[API key](https://docs.pingstack.io/mcp/api-key/) when your client can't sign
in through a browser.

## Tools

The server has 35 tools. Your client only sees the ones the connection has
permission for.

**Checks**

| Tool | What it does |
| --- | --- |
| `get_account_capabilities` | Account, plan limits, usage, a count of checks by status, time zone and the fields each check type accepts |
| `list_checks` | List and filter checks, by one status or several |
| `get_check` | One check's settings, status, why it's down, last run and expiry date |
| `create_check` | Create a heartbeat, website, TCP, DNS, SSL certificate or domain check |
| `update_check` | Change a check's settings |
| `get_check_history` | Daily runs, latency and incidents for up to a year |
| `list_check_runs` | Individual runs for up to 90 days |
| `get_check_metrics` | Latency percentiles, by probe region or DNS resolver |
| `get_check_uptime` | Uptime over 7 days to a year |

**Incidents**

| Tool | What it does |
| --- | --- |
| `list_incidents` | Incidents with their cause, newest first, filtered by check and state |
| `get_incident` | One incident |
| `create_incident_update` | Post a status update on an incident |

**Notification groups**

| Tool | What it does |
| --- | --- |
| `list_notification_groups` | Alert groups, with their members and rosters |
| `list_account_members` | People on the account |
| `list_rosters` | On-call rosters |
| `create_notification_group` | Create a group from people and rosters already on the account |
| `update_notification_group` | Rename a group or change who it alerts |
| `assign_notification_group` | Add a group to many checks, or to every check on a dashboard |

**Dashboards and status pages**

| Tool | What it does |
| --- | --- |
| `list_dashboards` | Dashboards and status pages |
| `get_dashboard` | One dashboard, its sections and its checks |
| `create_dashboard` | Create a private dashboard, with checks already on it if you like |
| `update_dashboard` | Change a dashboard's name and display settings |
| `attach_check_to_dashboard` | Add one check or several to a dashboard |
| `detach_check_from_dashboard` | Remove a check from a dashboard |
| `reorder_dashboard_checks` | Reorder a dashboard's checks |
| `create_dashboard_section` | Add a section heading to a dashboard |
| `update_dashboard_section` | Rename a section or set which checks it holds |
| `publish_dashboard` | Make a dashboard a public status page (account Admins only) |

**Off unless you turn them on**

These permissions start unticked when you approve a connection, each with a
**Sensitive** badge. Tick one only if your agent needs it.

| Tool | What it does |
| --- | --- |
| `get_check_ping_urls` | A heartbeat check's ping URLs, which work as credentials |
| `pause_check`, `resume_check` | Pause a check and resume it. A paused check sends no alerts |
| `delete_check` | Delete a check and its history for good |
| `delete_dashboard` | Delete a private dashboard. Its checks are kept |
| `delete_notification_group` | Delete an alert group |
| `unpublish_dashboard` | Take a status page offline |

A connection can delete 10 items a minute. A few minutes after it deletes
anything, the account owner gets an email listing what was deleted.

History goes back as far as your plan keeps it. See the
[tool reference](https://docs.pingstack.io/mcp/tools/) for every argument.

## Skills

The Claude Code and Cursor plugins come with three skills. Each one walks your
agent through a whole job, so you don't have to prompt every step.

| Skill | What it does |
| --- | --- |
| `monitor-cron-job` | Reads a job's schedule, creates a matching heartbeat check and adds the ping to the job |
| `client-dashboard` | Builds a dashboard for a client or team, organizes it into sections and sets up who gets alerted |
| `uptime-report` | Writes up uptime, incidents and their causes for a dashboard or the whole account |

In Claude Code, run one with `/pingstack:monitor-cron-job`, or describe what
you want and Claude picks the skill. The skills are plain Markdown in the
[`skills`](skills) folder.

## What it won't do

The server can't invite people, add notification channels such as email, Slack
or SMS, edit rosters, change status page branding or domains, change a check's
type or buy anything. Those stay in the Pingstack app. It never sends a test
alert or runs a check on demand.

Every connection is limited to one account, to the permissions approved when
it was connected, and to the approving person's current role. At a plan limit
it tells your agent the limit and returns an upgrade link for a person to open.

## Example prompts

- Add a daily heartbeat for the backup job with a 15 minute grace period and put it on the Infrastructure dashboard.
- Which checks had incidents in the last 7 days, what caused them, and how long were they down?
- Create a dashboard for the Northstar client with their checks in two sections, then a notification group with Alex and me in it, and alert that group for every check on the dashboard.
- Pause the staging checks while we migrate the database, and resume them when I say so.
- Create website and SSL certificate checks for every domain in this README.
- What's the p95 latency for the API check over the last 30 days?
- How many checks do I have left on my plan?

## Support

- Docs: [docs.pingstack.io](https://docs.pingstack.io/mcp/overview/)
- Email: [support@pingstack.io](mailto:support@pingstack.io)
- Privacy policy: [pingstack.io/privacy-policy](https://pingstack.io/privacy-policy/)
- Terms: [pingstack.io/terms-conditions](https://pingstack.io/terms-conditions/)

Please report security issues to support@pingstack.io rather than opening a
public issue.

## License

The files in this repository are under the [MIT License](LICENSE). The
Pingstack service is covered by its [terms](https://pingstack.io/terms-conditions/).
