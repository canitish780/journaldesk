# Ledger — personal accounting webpage backed by Google Sheets

A double-entry journal + account-wise ledger app. Just **2 files**:

| File | Where it goes |
|---|---|
| `index.html` | Your GitHub repo (this is the whole webpage — single file, no build step) |
| `Code.gs` | Pasted into your Google Sheet's Apps Script editor |

Your Google Sheet is the database. `index.html` is a single-page app: nav
clicks switch views instantly, no page reloads.

## Your setup (already done)

- **Webpage:** https://canitish780.github.io/journaldesk/
- **Apps Script Web App URL:** already wired into `index.html` (the
  `APPS_SCRIPT_URL` constant near the top of the `<script>` block)
- **Passphrase:** `trusttheprocess` (set in `Code.gs` — change any time
  by editing `PASSPHRASE` there and redeploying a new version)

## To go live

1. Paste `Code.gs` into your Sheet's Extensions > Apps Script (if you
   haven't already) and deploy it as a Web App (Execute as: Me, Access:
   Anyone) — you've already done this since you sent me the URL.
2. Push `index.html` to the root of your `journaldesk` GitHub repo.
3. Confirm GitHub Pages is serving from that branch/root in repo
   Settings > Pages.
4. Open https://canitish780.github.io/journaldesk/ — you'll be asked
   for your passphrase once per device.

**Whenever you edit Code.gs later:** Deploy > Manage deployments > edit
(pencil icon) > Version: **New version** > Deploy. The URL stays the
same, but changes only go live after a new version.

## Using it

1. **Accounts** — add accounts under Assets, Liabilities, Income, or
   Expense, with opening balances (Debit or Credit side).
2. **New entry** — post double-entry journal entries. Add as many
   debit/credit lines as you need; they must balance before posting.
3. **Ledgers** — pick any account to see its full history with a
   running balance, filterable by date range.
4. **Dashboard** — totals by head and your most recent entries.

## How data loss is avoided

- Every journal entry and new account is written to **browser storage
  the instant you submit it** — before any network call happens.
- A background sync loop retries every ~20 seconds, and immediately
  when the tab regains focus or the device comes back online.
- Every entry carries a unique client-generated ID, so a retried sync
  never creates a duplicate row even if an earlier attempt actually
  succeeded but its response was lost.
- The sync stamp (top right) shows amber while entries are
  queued/syncing, green with a timestamp once everything is confirmed
  saved to your Sheet.

## Security note on the passphrase

The passphrase only exists inside `Code.gs`, which lives on Google's
servers — it's never part of the public `index.html` on GitHub. Once you
unlock a device, the webpage stores a private token (not the passphrase)
in that browser's local storage, and the token is what gets checked on
every write from then on. Clearing that browser's storage, or using a
new device, means entering the passphrase again.

## Notes & limits

- Built for personal / single-person use across your own devices.
- Apps Script free-tier quota is ~20,000 executions/day on a personal
  Gmail account — far beyond normal personal bookkeeping use.
- Multiple people editing at the same time isn't specifically handled.
