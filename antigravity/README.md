# Pingstack for Antigravity

Pingstack monitors cron jobs and scheduled tasks with heartbeat checks, and
checks websites, TCP ports, DNS records, SSL certificates and domain expiry.
This plugin connects Antigravity to Pingstack's remote MCP server at
`https://app.pingstack.io/mcp`.

## Sign-in

The first time the server connects, Antigravity opens your browser to sign in
to Pingstack. An Owner or Admin approves the connection and picks its
permissions. Heartbeat URL access is off by default.

## What your agent can do

- Create and update heartbeat, website, TCP, DNS, SSL certificate and domain checks
- Create dashboards and attach, detach and reorder checks on them
- List incidents and read uptime, daily history, run history and latency
- Read your plan's limits and usage

It can't delete anything, publish a status page, pause checks or buy anything.

## Example prompts

- Add a daily heartbeat for the backup job with a 15 minute grace period and attach it to the Infrastructure dashboard
- Which checks had incidents in the last 7 days, and how long were they down?
- What's the p95 latency for the API check over the last 30 days?

## Links

- Docs: https://docs.pingstack.io/mcp/overview/
- Tool reference: https://docs.pingstack.io/mcp/tools/
- Pricing: https://pingstack.io/pricing/ (free for 15 checks, with MCP on every plan)
- Privacy: https://pingstack.io/privacy-policy/
- Support: support@pingstack.io
