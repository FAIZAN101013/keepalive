# keepalive

One public repo that keeps all my servers and databases awake, so the
project repos themselves can be private without burning the 2,000
free private-repo Actions minutes. Public repos get unlimited minutes.

## What runs here

| Workflow | What it does | Schedule |
|---|---|---|
| `keep-servers-awake.yml` | curl-pings the Noted API (Render sleeps after 15 idle min; hours budgeted — see file header) | every 10 min, 05:30–00:29 IST |
| `keep-resume-optimizer-alive.yml` | Supabase read + **write** + self-healing restore for resume-optimizer | daily 03:23 UTC |
| `wake-shopngo.yml` | manual wake button for the ShopNGo Render API (run before a demo) | manual |
| `heartbeat.yml` | commits a timestamp monthly so GitHub never auto-disables the schedules (it disables cron in repos with 60 days of no activity) | monthly |

## How to ADD a simple server ping (Render, Railway, Flyâ€¦)

Open `.github/workflows/keep-servers-awake.yml` and add one line in the
`run:` block, copying the existing pattern:

```
curl -fsS -m 60 https://YOUR-SERVER.onrender.com/health || echo "ping failed"
```

Commit. Done â€” no secrets needed for public health endpoints.

## How to REMOVE a server ping

Delete its `curl` line from the same file and commit. To stop a whole
workflow without deleting it: Actions tab â†’ pick the workflow â†’ "â€¦" menu â†’
**Disable workflow**.

## How to ADD a Supabase project

1. Copy `keep-resume-optimizer-alive.yml` to a new file, e.g.
   `keep-myproject-alive.yml`.
2. In the copy, change the `name:`, pick a different cron minute, and
   replace every `RESUME_OPT_` with your prefix, e.g. `MYPROJECT_`.
3. Add the secrets in THIS repo: Settings â†’ Secrets and variables â†’
   Actions â†’ New repository secret:
   - `MYPROJECT_SUPABASE_URL` â€” https://xxxx.supabase.co
   - `MYPROJECT_SUPABASE_ANON_KEY` â€” dashboard â†’ Settings â†’ API
   - `MYPROJECT_SUPABASE_SERVICE_ROLE_KEY` â€” same page (keep it ONLY here)
   - `MYPROJECT_SUPABASE_ACCESS_TOKEN` â€” optional, from
     supabase.com/dashboard/account/tokens; enables auto-restore if paused
4. The project's database needs the `keepalive` table (the
   `0003_keepalive.sql` migration from resume-optimizer) â€” the write step
   fails loudly without it, which is correct.

Note: secret VALUES cannot be copied from the old repo (GitHub never shows
them again) â€” take them fresh from the Supabase dashboard.

## How to REMOVE a Supabase project

Delete its workflow file (and optionally its secrets) and commit.

## After moving a workflow here

Disable the old copy in the project repo: that repo â†’ Actions â†’ the
workflow â†’ "â€¦" â†’ Disable workflow (or delete the file). Otherwise it keeps
eating private-repo minutes.
