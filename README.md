# Waypoint Error Triage: Group Build Data Pack

Sample data for the **Claude Code for developers** workshop group build (AdVenture Media, built for Waypoint Analytical's development team).

Production errors arrive as emailed stack traces. Customers report the same problems in TeamSupport, SQL Server shouts about deadlocks in Slack, and nobody has one view that says *which bug to fix first*. In this build we make that view: an internal error-triage dashboard in C# / .NET, built live with Claude Code.

## Fully synthetic

**Everything in this repo is made up.** The apps, servers, releases, accounts, growers, retailers, people, email addresses, phone numbers and stack traces are invented to look like a lab's production support data. They do not come from Waypoint Analytical's systems.

- Email domains end in `.example` (reserved, never routable). Phone numbers use the reserved 555-01xx range. IP addresses come from documentation ranges (192.0.2.x, 198.51.100.x, 203.0.113.x).
- No real customer, co-op, retailer or staff names. Any resemblance is coincidence.

## How to download

**Option A: download everything (recommended).** Click the green **Code** button at the top of this page, then **Download ZIP**. Unzip it and open the folder in the Claude desktop app (Code tab) or run `claude` in it.

**Option B: clone it.**
```
git clone https://github.com/isaacjordan1022/waypoint-error-triage.git
cd waypoint-error-triage
```

You need the **.NET 10 SDK** (`dotnet --version` should print 10.x) and Claude Code.

## What's inside

```
data/
  error-emails/              723 .eml files, production error emails, Oct 1 to Nov 15 2026
  teamsupport-tickets.csv    2,500 TeamSupport tickets, Jan to Nov 2026
  slack-sql-alerts.json      285 messages from the #sql-alerts Slack channel, Oct 1 to Nov 15 2026
  releases.csv               120 production deployments across 5 apps, Jan to Nov 2026
  accounts.csv               400 customer accounts (growers, co-ops, retailers, consultants, homeowners)
CLAUDE.md                    project notes for Claude
```

The scenario: Waypoint's web team runs four production apps.

| App | What it is | Error emails come from |
|---|---|---|
| `Waypoint.Portal` | Blazor Server client portal: results, invoices, sample tracking, sample submission, retailer admin | WPA-WEB01 to WPA-WEB04 |
| `Waypoint.Mobile.Api` | API behind the iOS and Android app | wpa-mobile-api containers |
| `Waypoint.Reports` | PDF report builder (soil, tissue, water; standard and retailer white-label templates) | WPA-RPT01, WPA-RPT02 |
| `Waypoint.Integrations.Ups` | UPS label and tracking integration | WPA-INT01 |

A fifth app, `Waypoint.LimsSync`, is the nightly job that copies results from the desktop LIMS into the portal database. It appears in `releases.csv` and in the Slack alerts.

## Column dictionary

### data/error-emails/*.eml
One file per email as exported from the GroupWise gateway (RFC 822 / MIME, CRLF line endings). The subject is `Production | Error: <ExceptionType> in <App>`. The body is written by the in-house ErrorMailer: a block of `Label : value` lines (Environment, Server, Time, Release, request and user details), then `Exception:` and the .NET exception text with its stack trace. `Release` is `<version>+<short commit sha>` and matches `releases.csv`. File names are GroupWise export ids and carry no meaning.

### teamsupport-tickets.csv
| Column | Meaning |
|---|---|
| `ticket_number` | TeamSupport ticket number (41873 and up) |
| `opened_at` | When the ticket was opened, `YYYY-MM-DD HH:MM`, US Central (the export user's time zone) |
| `closed_at` | When it was closed; blank if still open |
| `account_number` | Customer account (`accounts.csv`); blank for callers without an account |
| `product` | Portal, Mobile, LIMS or Reports |
| `severity` | Low, Normal, High, Urgent |
| `status` | New, Open, In Progress, Waiting on Customer, Closed |
| `subject` | Ticket subject as the customer or support agent wrote it |
| `description` | First message of the ticket, free text as pasted in |
| `assignee_team` | Web, Mobile, LIMS, Reports, Customer Service or Billing |

### slack-sql-alerts.json
A Slack channel export: a JSON array of message objects. Bot alerts from "SQL Monitor" have `subtype: "bot_message"`, a `text` headline and `attachments[0].fields` (title/value pairs such as Database, Victim, Other process, Objects, Event time). People's replies have `user`, `user_profile` and `thread_ts` pointing at the alert they answer. `ts` is the Slack timestamp (Unix epoch seconds) and also the message id.

### releases.csv
| Column | Meaning |
|---|---|
| `app` | Waypoint.Portal, Waypoint.Mobile.Api, Waypoint.Reports, Waypoint.Integrations.Ups or Waypoint.LimsSync |
| `version` | Semantic version |
| `deployed_at` | Production deploy time, ISO 8601 UTC |
| `commit_sha` | Full git commit sha (the first 7 characters appear in error emails) |
| `notes` | Release notes, one line |

### accounts.csv
| Column | Meaning |
|---|---|
| `account_number` | 5-digit customer account number |
| `account_name` | Business or homeowner name |
| `account_type` | grower, co_op, retailer, consultant, enterprise, lawn_garden |
| `parent_account_number` | For growers added by a retailer (retailers resell tests to their growers), the retailer's account; blank otherwise |
| `state` | US state |
| `home_lab` | Lab code: MTN Memphis, ACA Anaheim, WNC Wilson, CIL Champaign, AIA Atlantic, LPA Leola, RVA Richmond, GNE Grand Island |
| `portal_enabled` | true if the account uses the client portal |
| `price_tier` | Standard, Volume, Retail Partner, Enterprise |
| `created_date` | Account created, YYYY-MM-DD |

## The prompt

Open this folder in the Claude desktop app's Code tab (or run `claude` here), then paste:

```
My goal: build an internal error-triage dashboard for Waypoint's web team, so that every morning we can see which production bugs to fix first, which release introduced each one, and which customers it is hurting.

The inputs are in data/: production error emails (.eml), TeamSupport tickets, Slack SQL alerts, releases and accounts. CLAUDE.md and the README column dictionary describe them. Treat data/ as read-only.

Target stack: .NET 10 (LTS), C#, ASP.NET Core, SQLite, a simple web dashboard served by the same app, and xUnit tests. One command should import the data into SQLite, and one should run the dashboard.

The dashboard should have:
- a list of error groups, one row per distinct bug, with count, first seen, last seen, app, the release it first appeared in, and the number of customer accounts affected
- errors per day over time, with release markers
- an hour-of-day by weekday heatmap of errors and SQL alerts
- for each error group, the TeamSupport tickets that look related
- a detail page for one error group with a sample email and its stack trace

Before you write any code: read CLAUDE.md and the README, look at a handful of the data files, then interview me briefly (3 to 5 questions). Then propose a plan: the file tree, projects, language and frameworks, NuGet libraries, how you will parse the emails and group the errors, and which tests you will write. Wait for my OK before building. When you build, run the tests and start the app so I can see it.
```

## After it builds

Review it the way you would review a teammate's pull request:

1. `/code-review high` on your changes. Walk through the findings, fix the real ones, then run it again.
2. `/security-review` (it needs an `origin` remote: create your own empty GitHub repo and push to it first).
3. A second opinion from a fresh session. Start a new session in this folder and paste:

```
My goal: an independent review of this error-triage app before the web team relies on it. Read the code, the tests and a sample of the files in data/, then list the most serious problems you find, worst first, with the file and line for each and how you would prove it. Don't change any code yet.
```


---

*Fictional data for training purposes. Made for the Claude Code workshop at Waypoint Analytical · AdVenture Media.*
