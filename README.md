# keepalive

One public repo that keeps all my servers and databases awake, so the
project repos themselves can be private without burning the 2,000
free private-repo Actions minutes. Public repos get unlimited minutes.

## What runs here

| Workflow | What it does | Schedule |
|---|---|---|
| `keep-servers-awake.yml` | curl-pings web servers (Render free tier sleeps after 15 idle minutes) | every 10 min |
| `keep-resume-optimizer-alive.yml` | Supabase read + **write** + self-healing restore for resume-optimizer | daily 03:23 UTC |
| `keep-shop-and-go-alive.yml` | same, for shop-and-go | daily 03:41 UTC |
| `heartbeat.yml` | commits a timestamp monthly so GitHub never auto-disables the schedules (it disables cron in repos with 60 days of no activity) | monthly |

## How to ADD a simple server ping (Render, Railway, Fly…)

Open `.github/workflows/keep-servers-awake.yml` and add one line in the
`run:` block, copying the existing pattern:

```
curl -fsS -m 60 https://YOUR-SERVER.onrender.com/health || echo "ping failed"
```

Commit. Done — no secrets needed for public health endpoints.

## How to REMOVE a server ping

Delete its `curl` line from the same file and commit. To stop a whole
workflow without deleting it: Actions tab → pick the workflow → "…" menu →
**Disable workflow**.

## How to ADD a Supabase project

1. Copy `keep-resume-optimizer-alive.yml` to a new file, e.g.
   `keep-myproject-alive.yml`.
2. In the copy, change the `name:`, pick a different cron minute, and
   replace every `RESUME_OPT_` with your prefix, e.g. `MYPROJECT_`.
3. Add the secrets in THIS repo: Settings → Secrets and variables →
   Actions → New repository secret:
   - `MYPROJECT_SUPABASE_URL` — https://xxxx.supabase.co
   - `MYPROJECT_SUPABASE_ANON_KEY` — dashboard → Settings → API
   - `MYPROJECT_SUPABASE_SERVICE_ROLE_KEY` — same page (keep it ONLY here)
   - `MYPROJECT_SUPABASE_ACCESS_TOKEN` — optional, from
     supabase.com/dashboard/account/tokens; enables auto-restore if paused
4. The project's database needs the `keepalive` table (the
   `0003_keepalive.sql` migration from resume-optimizer) — the write step
   fails loudly without it, which is correct.

Note: secret VALUES cannot be copied from the old repo (GitHub never shows
them again) — take them fresh from the Supabase dashboard.

## How to REMOVE a Supabase project

Delete its workflow file (and optionally its secrets) and commit.

## After moving a workflow here

Disable the old copy in the project repo: that repo → Actions → the
workflow → "…" → Disable workflow (or delete the file). Otherwise it keeps
eating private-repo minutes.
