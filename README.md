# hermes-updates

Static JSON snapshots of GitHub release feeds for the Hermes ecosystem.

Consumed by the public changelog page at <https://yuai-r.cn/hermes/changelog/>.

## What's here

- `data/portable.json` — releases from [`yuluyangguang1/hermes-portable`](https://github.com/yuluyangguang1/hermes-portable)
- `data/standard.json` — releases from [`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent)

Both files are arrays of release objects with the fields the changelog
page uses: `tag_name`, `name`, `body`, `published_at`, `created_at`,
`html_url`, `prerelease`, `draft`.

## How it stays fresh

The [`sync` workflow](.github/workflows/sync.yml) runs every 30 minutes,
authenticates with the default `GITHUB_TOKEN` (5000 req/h instead of the
anonymous 60 req/h), pulls the two upstream feeds, and commits any
diff. yuai-site has its own scheduled job that mirrors these files into
`/hermes/changelog/data/` so the page never has to call the GitHub API
from the browser.

## Why this indirection?

Calling `api.github.com` directly from the changelog page tripped the
anonymous rate limit on shared NATs. Two static files served from this
repo (and mirrored to yuai-site) are cheap, cacheable, and never
return a 403.

## Manual run

Trigger from the Actions tab → "Sync Hermes release feeds" → Run
workflow. The job is idempotent; if neither feed has changed it will
exit cleanly without committing.

## First-time setup

After pushing this repo to GitHub:

1. **Settings → Actions → General → Workflow permissions**
   choose **Read and write permissions** (so the scheduled job can
   `git push` updated JSON snapshots back).
2. **Actions tab → "Sync Hermes release feeds" → Run workflow**
   to seed `data/portable.json` and `data/standard.json` immediately
   instead of waiting for the next half-hour cron tick.
3. yuai-site already has a paired sync job (`.github/workflows/sync-manuals.yml`)
   that mirrors `data/*.json` from this repo into
   `/hermes/changelog/data/`. Once it runs, the page loads from the
   same origin instead of falling back to `raw.githubusercontent.com`.
