# Waypoint error triage

An internal error-triage dashboard for a fictional version of Waypoint Analytical's web team (soil and plant testing lab). The data is synthetic production-support data for their client portal, mobile API, PDF reports and UPS integration.

## Data (inputs, read-only)
- `data/error-emails/` production error emails (.eml) exported from the GroupWise gateway
- `data/teamsupport-tickets.csv` TeamSupport ticket export
- `data/slack-sql-alerts.json` Slack export of the #sql-alerts channel
- `data/releases.csv` production deployments
- `data/accounts.csv` customer accounts

Column meanings are in README.md under "Column dictionary". Never edit files in `data/`.

## Stack
- .NET 10 (LTS), C#, ASP.NET Core, SQLite, xUnit
- A simple web dashboard served by the same app

## Commands
Add the build, test, import and run commands here once the solution exists.

