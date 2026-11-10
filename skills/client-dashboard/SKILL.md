---
name: client-dashboard
description: Build a Pingstack dashboard for a client, project or team. Puts their checks on it, organizes them into sections, sets up who gets alerted, and can publish it as a status page. Use when the user wants a dashboard or status page for a group of checks, or wants a group of people alerted for them.
---

# Set up a dashboard for a client or team

This skill builds a dashboard, fills and organizes it, and sets up alerts for
the people responsible. It uses the `pingstack` MCP server's tools. Plan first,
show the plan, then make the changes.

## 1. Learn what the user wants

You need:

- the dashboard's name, usually the client or project
- which checks belong on it
- whether the checks should be grouped under headings, such as "Websites" and "Scheduled jobs"
- who should be alerted when one of those checks fails
- whether the dashboard should be public as a status page

Ask only for what the request didn't already say.

## 2. Look at the account

- `get_account_capabilities`: check `limits.dashboard_limit` against `usage.dashboards`, and read `scopes` to see what this connection may do.
- `list_checks` with `q` to find checks by name. Follow `next_cursor` until it's null so you don't miss any.
- `list_dashboards`: make sure a dashboard with that name doesn't already exist.
- `list_account_members`: find the people to alert by name. Only people already on the account can be added. If someone isn't listed, tell the user to invite them in Pingstack first.
- `list_notification_groups`: see whether a suitable group already exists.

## 3. Show the plan

Before changing anything, list:

- the dashboard name and the checks going on it, by name
- the sections and which checks go in each
- the notification group's name and its members
- whether it will be published

Wait for the user to agree.

## 4. Build the dashboard

1. `create_dashboard` with `dashboard: {name}`, `check_ids` for up to 100 checks, and an `idempotency_key`. Reuse the key if you retry. For more than 100 checks, add the rest with `attach_check_to_dashboard` and `check_ids`.
2. For each heading, `create_dashboard_section` with the dashboard's id and a `name`.
3. For each section, `update_dashboard_section` with `check_ids` listing its checks in the order they should appear.

New dashboards are private.

## 5. Set up alerts

1. `create_notification_group` with `group: {name, user_ids}`. Use the ids from `list_account_members`. To use an existing group, skip this.
2. `assign_notification_group` with the group's `id` and the `dashboard_id`. Every check on the dashboard now alerts that group, and keeps any groups it already had.

Tell the user that people in the group are alerted through their own
notification settings, and that channels such as a shared email address or a
Slack channel are added to the group in Pingstack. This server can't add
channels.

## 6. Publish, only if asked

Publishing makes the dashboard a status page that anyone with the link can
open. It shows the names and statuses of its checks, and its incident history
if that setting is on.

- Confirm with the user that the listed checks are fine to show publicly.
- Call `publish_dashboard`. It needs an account Admin; an Editor's connection gets `forbidden`.
- Share `domain_url` if it's set, otherwise `public_url`.

Changes to a public dashboard show on its status page right away.

## 7. Report back

Summarize what was created: the dashboard and its link in Pingstack if you
have one, the sections, the group and who is in it, and the public link if it
was published. Mention anything you couldn't do, such as a person who isn't on
the account yet.
