# syshub: vision

## In one line
syshub is the home server's front page. It's one LAN address that tells you
what this Mac offers, whether it's healthy, and what its jobs have been doing.

## What it shows
1. **Services directory (ready reckoner).** What runs here and how to reach it:
   addresses, ports, ready-to-copy commands, short how-tos.
   *e.g.* Open WebUI at `:8080`, `mysql -h <host> -u root -p`, `ssh <user>@<host>`,
   the Screen Sharing and SMB links.
2. **Service health.** Whether each service is up right now, checked live.
   *e.g.* Ollama `/api/version`, Open WebUI `/health`, Jupyter `/login`. Later:
   a TCP check on MariaDB `:3306`, or checking that a process is running.
3. **Job status.** What each scheduled job did last time, whether it's
   overdue, and its run history.
   *e.g.* nightly DB backups (result, sizes, snapshots, next run) and the
   hourly sconsole screenshot backup.
4. **System health (later).** The machine itself.
   *e.g.* free disk space on the backup volume, uptime, load, pending macOS
   updates, certificate or key expiry.

## Principles
- **Jobs report, syshub reads.** Each job writes its own status file, and
  syshub only reads them. A job never depends on the web server, and syshub
  never needs to know how a job works.
- **One status format for all jobs.** Every job writes the same top-level
  fields, plus any detail of its own:
  ```json
  { "job": "db-backup", "overall": "ok|running|failed|stale",
    "headline": "Backups OK", "last_run": {...}, "next_scheduled": "...",
    "detail": { ...anything job-specific... } }
  ```
  A new job shows up on the page just by writing its file to the status
  folder.
- **Driven by config, not code.** Services, health checks and links are listed
  in a config file. Adding a service means adding an entry, not writing a page.
- **The public repo holds no machine details.** The code is public on GitHub.
  Hosts, ports, service lists and DB names go in a config file that git
  ignores, with an example copy committed.
- **Simple stack.** Caddy serves the static files and passes PHP to PHP-FPM.
  Plain PHP, with no framework and no build step. Everything runs as
  LaunchAgents under the user, without sudo.
- **Read-only and LAN-only to start.** It shows information. Actions such as
  "run backup now" or "restart a service" come later, if ever, and
  deliberately.
- **Follows the system-utils conventions.** Same folder layout, the same
  build/deploy/undeploy scripts and script standard, and `target/` is never
  committed.

## How it grows
| To add…                  | You…                                 | Example                                       |
|--------------------------|--------------------------------------|-----------------------------------------------|
| a service                | add a config entry                   | a new Home Assistant card with its health URL |
| a job                    | have it write `status/<job>.json`    | the screenshot backup reports files moved     |
| a type of health check   | add one PHP checker                  | `tcp`, `process`, `disk` next to `http`       |
| a detail page            | add a PHP page per area              | `/backups/` today; `/jobs/<name>` later       |
| machine-readable access  | expose a JSON endpoint               | `/api/status`, usable by scripts or alerts    |

## Rough phases
1. **Move the current setup.** Run Caddy and PHP-FPM from the repo. Bring the
   landing page and backups page across as they are, and move the health
   checks from the Caddyfile into PHP and config.
2. **Standardize job status.** Agree on the status folder and format. Point
   `backup-status` at it, and add the screenshot backup.
3. **Generic job views.** One page design that renders any job's status, plus
   overdue detection for every job.
4. **System health and extras.** Disk, uptime and similar checks, a JSON API,
   and maybe notifications.

## Open questions
- Should Caddy serve the site straight from the repo, or from a copy that
  deploy puts in `~/softwares/workspace/syshub/`?
- Where do status files live, and does each job own its file, or does syshub
  collect them?
- Do `db-backup` and `backup-status` become projects in `system-utils`?
- How much does phase 1 cover: a straight move, or a redesign now?
