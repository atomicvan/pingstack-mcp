---
name: uptime-report
description: Write an uptime report from Pingstack for a dashboard or a whole account. Covers uptime per check, incidents with their causes and durations, and response times. Use when the user asks for a weekly or monthly uptime report, an incident summary, or how their monitoring did over a period.
---

# Write an uptime report

This skill reads monitoring data and writes it up. It only reads; it changes
nothing. It uses the `pingstack` MCP server's tools.

## 1. Settle the scope

- **Which checks.** One dashboard, or the whole account. For a dashboard, find it with `list_dashboards` and read its checks with `get_dashboard`. For the account, use `list_checks`. Follow `next_cursor` until it's null in both cases.
- **Which period.** Uptime and history accept 7, 14, 30, 90 or 365 days. Incidents accept 7 or 30. Default to 7 days for a weekly report and 30 for a monthly one.

## 2. Read the account's settings

Call `get_account_capabilities` and note:

- `time_zone`: pass it as `time_zone` to every history call, so days line up with the user's
- `limits.history_days` and `limits.run_history_days`: don't ask for a longer period than the plan keeps. Asking for more returns `quota_exceeded`.
- `usage.checks_by_status`: the current count of checks up, down, late, degraded, paused and pending

## 3. Gather the data

For the period chosen:

- `list_incidents` with `days`, and `check_id` when reporting on a subset. Each incident has the check's name, its `cause`, when it started and resolved, and `duration_seconds`. An open incident has `resolved_at: null`, and its duration counts up to now.
- `get_check_uptime` for each check, with `days` and `time_zone`. Read `uptime_percentage` and `downtime_seconds`. A null `uptime_percentage` means the check has no runs or incidents yet.
- `get_check_metrics` for website, DNS and TCP checks, for `p50_duration_ms` and `p95_duration_ms`. Heartbeat, domain and SSL checks have no response times, and the call returns `invalid_kind` for them.
- `get_check` for domain and SSL checks, to read `expires_on`.

On a large account, read uptime for every check, and metrics only for the
checks the user cares about or the ones that had incidents. The server allows
60 calls a minute per connection.

Don't call `get_check_ping_urls`. A report never needs ping URLs.

## 4. Write the report

Lead with the answer, then the detail:

1. **Summary.** One or two sentences: how many checks, overall uptime, how many incidents and the total downtime. Say plainly if everything was up.
2. **Incidents.** Newest first: the check, what caused it, when it started, how long it lasted, and whether it's still open. Group repeats of the same cause on the same check.
3. **Uptime by check.** A table of check, uptime percentage and downtime. Sort the lowest uptime first.
4. **Response times.** For website, DNS and TCP checks: median and 95th percentile.
5. **Coming up.** Domain or SSL checks that expire in the next 30 days.

Give times in the account's time zone and name it once. Use the check names as
they are in Pingstack.

## 5. Be straight about the limits of the data

- Uptime here is incident-free time. Time a check spent paused or pending counts as up, so say so if any check was paused during the period.
- A check created partway through the period only has data from its creation.
- If the plan keeps less history than the period asked for, say which period the report covers.
