---
name: monitor-cron-job
description: Add Pingstack heartbeat monitoring to a cron job or scheduled task. Creates a check that matches the job's schedule, then wires the ping into the job. Use when the user wants to know if a cron job, backup, scheduled task or worker stops running or fails.
---

# Monitor a cron job with Pingstack

A heartbeat check expects the job to call a ping URL each time it finishes. If
the ping doesn't arrive on schedule, Pingstack opens an incident and alerts.
This skill creates the check and adds the ping to the job. It uses the
`pingstack` MCP server's tools.

## 1. Find the job and its schedule

Read the job's definition before asking the user anything. Look for:

- a crontab line or a file in `/etc/cron.d`
- `on.schedule` in a GitHub Actions workflow
- a Kubernetes `CronJob` spec
- a systemd timer
- a scheduler config, such as `config/schedule.rb`, `config/recurring.yml`, Celery beat or `vercel.json` crons

Work out three things:

- **The schedule.** A five-field cron expression, or a plain interval such as every 15 minutes.
- **The time zone the schedule runs in.** Server crontabs use the machine's zone. GitHub Actions and most hosted schedulers use UTC. If you can't tell, ask.
- **How long a run takes.** Ask if you can't tell from the code.

## 2. Check the account

Call `get_account_capabilities`. Note:

- `limits.check_limit` and `usage.checks`, to confirm there's room for one more check
- `effective_min_interval_seconds`, the shortest interval the plan allows
- `scopes`, to see what this connection may do. Creating needs `checks:create`.

If `checks:create` is missing, tell the user the connection needs that
permission and stop.

## 3. Propose the check

Build the check and show it to the user before creating anything:

- `name`: what the job does, such as "Nightly database backup"
- `kind`: `ping`
- For a cron schedule: `schedule_type: "cron"`, `cron_expression`, and `timezone` as an IANA name such as `America/New_York`
- For a plain interval: `schedule_type: "interval"` and `interval_seconds`
- `grace_period_seconds`: how late a ping may be before the check is marked late. Use the job's usual run time plus a margin. Ten minutes is a sensible start for a job that takes a few minutes.

Call `list_notification_groups` and ask which group should be alerted. Pass its
id in `notification_group_ids`. With no group, nobody is told when the job
stops.

Wait for the user to agree to the settings.

## 4. Create the check

Call `create_check` with the `check` object and an `idempotency_key`, such as a
new UUID. If the call fails partway and you retry, reuse the same key so you
don't create a duplicate.

If it returns `quota_exceeded`, nothing was created. Tell the user the limit,
and share `billing.upgrade_url` only if it isn't null.

## 5. Get the ping URL without exposing it

The ping URL works as a credential. Anyone who has it can report the job as
done. Never print it in chat, write it into a file that gets committed, or put
it in a log.

- If `get_check_ping_urls` is in your tool list, call it with the check's id. Store the URL straight into a secret, such as an environment variable named `PINGSTACK_PING_URL`, a GitHub Actions secret or a Kubernetes Secret.
- If that tool isn't listed, the connection doesn't have the permission. Tell the user to copy the URL from the check's page in Pingstack into the secret themselves.

## 6. Add the ping to the job

Ping after the work succeeds, so a failed run doesn't count as healthy:

```sh
your-job && curl --fail --silent --show-error --max-time 10 --retry 3 "$PINGSTACK_PING_URL"
```

To alert right away when the job fails, and to record how long each run took:

```sh
curl --fail --silent --show-error --max-time 10 "$PINGSTACK_PING_URL/start"
if your-job; then
  curl --fail --silent --show-error --max-time 10 --retry 3 "$PINGSTACK_PING_URL"
else
  curl --fail --silent --show-error --max-time 10 "$PINGSTACK_PING_URL/fail?exit_code=$?"
fi
```

Fit this to how the job is defined: a final step in a GitHub Actions job, the
last command of a container, an `ensure` block in application code. Reference
the secret by name. Don't inline the URL.

## 7. Confirm it works

Tell the user the check stays `pending` until the first ping arrives. After the
job's next run, call `get_check`. A status of `up` with a recent
`last_activity_at` means the ping is wired correctly. If it's still `pending`
or has gone `late`, check that the secret is set where the job runs and that
the job reached the ping line.
