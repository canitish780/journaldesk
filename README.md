# JournalDesk — personal accounting webapp backed by Google Sheets

A double-entry journal + account-wise ledger app, installable as a PWA
and usable offline for both data and document uploads. Backed entirely
by a Google Sheet (data) and Google Drive (documents) — no separate
database or server.

| File | Where it goes |
|---|---|
| `index.html` | GitHub repo root — the whole webpage |
| `manifest.json` | GitHub repo root — PWA install metadata |
| `service-worker.js` | GitHub repo root — offline app-shell caching |
| `icon-192.png`, `icon-512.png` | GitHub repo root — PWA icons |
| `Code.gs` | Pasted into your Google Sheet's Apps Script editor |

## Your setup

- **Webpage:** https://canitish780.github.io/journaldesk/
- **Apps Script Web App URL:** wired into `index.html`
- **Passphrase:** `trusttheprocess` (set inside `Code.gs`, never
  published to GitHub)
- **Google Drive folder:** documents upload under
  https://drive.google.com/drive/u/0/folders/1sKBik3-ZmiVLG2k_NfxgcIhI6dmv2YQi,
  in a subfolder named exactly after each account

## Redeploying this update

`Code.gs` changed (two new sheets — `AccountDetails`, `AccountDocuments`
— plus Drive integration):

1. Paste the updated `Code.gs` into Extensions > Apps Script, replacing
   the old code. Save.
2. Deploy > Manage deployments > pencil icon > Version: **New version**
   > Deploy. Same URL, no need to update `index.html`'s config.
3. The first time you upload a document, Apps Script will ask you to
   re-authorize (it now needs Drive access too) — approve it once.
4. Push `index.html`, `manifest.json`, `service-worker.js`,
   `icon-192.png`, and `icon-512.png` to your repo root.
5. If you'd already opened the app on a device before, do one hard
   refresh there so the new service worker takes over.

Existing accounts, entries, and categories are untouched.

## Features

**Journal entries** — classic accountant Dr / To layout. Add multiple
debit and credit lines (compound entries); they must balance to post.
Account fields are type-to-search, grouped by category.

**Ledgers** — search and pick an account to see a details panel (head,
category, account number, interest info, custom details, and clickable
document chips) followed by the full transaction history with running
balance. Defaults to the current financial year (1 April) through
today; both dates are editable.

**Accounts & categories** — group accounts under each head (e.g. under
Assets: Savings Accounts, Fixed Deposits, Gold, Land). Categorizing is
optional. When adding an account:
- **Additional details** — click **+ Add detail** repeatedly for
  free-form rows: CIF, IFSC, address, contact person, anything.
- **Documents** — click **+ Add document**, name what it is (e.g.
  "Passbook"), choose a file (max 8MB). It uploads to a Drive folder
  named after the account, and the file itself is named after what you
  typed.
- **Interest-bearing assets** — mark an Assets account as
  interest-bearing to get rate, accrual frequency (Monthly/Yearly/On
  maturity), one date field (its meaning adapts to the frequency), and
  which Income account the interest credits to.

**🔔 Reminders** — badge shows when interest is due within 7 days (or
overdue) on any interest-bearing account. Click an item to post a
pre-filled entry in two clicks.

**Dashboard** — totals by head and your most recent entries.

**🌙/☀️ Dark mode** — toggle next to the brand, remembered per device.

**Same layout everywhere** — one header (brand line + four nav buttons
filling the width) on mobile and desktop alike; content fills the
available screen width rather than being boxed into a fixed column.

**Installable, works offline** — Add to Home Screen / desktop like a
native app. The page itself is cached so it opens with no signal.

## How data loss is avoided

- Every journal entry, category, account, custom detail, and document
  is written to **this device's storage the instant you submit it**,
  before any network call happens. Documents specifically use
  IndexedDB (not `localStorage`) since file data needs more headroom.
- A background sync loop retries every ~20 seconds, and immediately
  when the tab regains focus or the device comes back online.
- Everything carries a unique client-generated ID, so a retried sync
  never creates a duplicate row — or a duplicate Drive upload — even
  if an earlier attempt actually succeeded but its response was lost.
- Custom details and documents wait for their parent account to be
  confirmed saved before syncing themselves, so nothing ever attaches
  to the wrong (or a not-yet-existing) account.
- The sync stamp (top right) shows amber while anything is
  queued/syncing, green with a timestamp once everything is confirmed
  saved to your Sheet and Drive.

## Security note on the passphrase

The passphrase only exists inside `Code.gs`, on Google's servers — never
in the public `index.html`. Once you unlock a device, the webpage stores
a private token (not the passphrase) in that browser's storage, and the
token is what's checked on every write after that. Clearing that
browser's storage, or using a new device, means entering the passphrase
again.

## Notes & limits

- Built for personal / single-person use across your own devices.
- Apps Script free-tier quota (~20,000 executions/day) is far beyond
  normal personal bookkeeping use.
- Interest estimates are simple (principal × rate ÷ periods) — always
  verify against your actual statement before posting.
- Documents are capped at 8MB per file to stay within Apps Script's
  request-size limits.
- Responsive down to small phone screens; identical behavior on mobile
  and desktop.
