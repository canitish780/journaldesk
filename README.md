# JournalDesk — personal accounting webapp backed by Google Sheets

A double-entry journal + ledger app, installable as a PWA and usable
offline for both data and document uploads. Backed entirely by a
Google Sheet (data) and Google Drive (documents) — no separate
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
- **Passphrase:** `trusttheprocess` (inside `Code.gs`, never on GitHub)
- **Google Drive:** account documents go in per-account subfolders;
  supporting documents from journal entries go in a shared
  "Supportings" subfolder; automatic weekly backups go in a "Backup"
  subfolder — all under
  https://drive.google.com/drive/u/0/folders/1sKBik3-ZmiVLG2k_NfxgcIhI6dmv2YQi

## Redeploying this update

`Code.gs` changed (new `CustomReminders` sheet, plus backup functions):

1. Paste the updated `Code.gs` into Extensions > Apps Script, replacing
   the old code. Save.
2. Deploy > Manage deployments > pencil icon > Version: **New version**
   > Deploy.
3. **One-time step — turn on weekly backups:** in the Apps Script
   editor, use the function dropdown at the top (next to Run/Debug) to
   select `setupWeeklyBackupTrigger_`, then click **Run**. Approve any
   permission prompts (it now needs to manage triggers). This installs
   a trigger that copies your whole Sheet into a dated file inside the
   "Backup" folder every Monday at 3am, and also takes one backup
   immediately so you can confirm it worked. You only need to do this
   once — re-running it later is safe (it won't create duplicate
   triggers), for example if you ever want to change the schedule.
4. Push the updated `index.html` to your repo root (other files
   unchanged this round, but fine to re-push all of them).

Existing accounts, entries, categories, and documents are untouched.

## What's new this round

**Dashboard is now the home screen**, opening to seven tiles instead of
stats:
- **New Entry** — jumps to the journal form
- **New Category** — jumps to Accounts and focuses the category field
- **New Account** — jumps to Accounts and focuses the account name field
- **Entries in F.Y.** — a searchable day book of every entry this
  financial year
- **All Ledgers** — search or browse every account; tap one to open its
  ledger for the current financial year
- **Profit & Loss** — income vs. expense for any date range, grouped by
  category, with net profit/loss called out
- **Balance Sheet** — assets vs. liabilities as of any date, grouped by
  category, with net worth called out

Recent-entry activity and account-head totals still show below the
tiles, same as before.

**Custom reminders.** The 🔔 dropdown now has a "+ Set custom reminder"
button — give it a label (e.g. "Pay rent", "Renew insurance") and a
date, and it shows up alongside interest reminders whenever it's due
within 7 days or overdue. Mark it done with one tap to remove it.

**Faster, lighter on your Apps Script quota.** Accounts, categories,
and entries are now cached per request for 60 seconds — switching
between Dashboard, Ledgers, and Accounts repeatedly no longer re-hits
your Sheet every single time, while still refreshing automatically the
moment you make a change. The background sync loop also now runs every
45 seconds instead of 20 — this doesn't affect how fast your own
entries save (that's still instant, to your device), it just means the
periodic "check for anything missed" pass happens less often.

**Smoother page transitions.** Switching views now has a quick,
deliberate fade-and-lift instead of an abrupt swap.

**Automatic weekly backups.** Every Monday at 3am, a complete dated
copy of your Sheet is saved into a "Backup" folder in Drive — extra
insurance independent of the sync system, in case you ever need to
roll back to a known-good state. Backups older than 60 days are
automatically cleaned up so the folder doesn't grow forever. See the
one-time setup step above.

## Full feature list

**Journal entries** — classic accountant Dr / To layout, auto-numbered
vouchers, optional supporting document capture (camera or file),
type-to-search account fields grouped by category.

**Ledgers** — search or browse every account (grouped by head and
category); tap one to see an editable details panel (head, category,
account number, interest info, custom details, document chips) above
the full transaction history with running balance. Defaults to the
current financial year (1 April) through today.

**Accounts & categories** — group accounts under each head. When adding
an account: unlimited custom details (CIF, IFSC, address, anything),
document uploads (Drive, per-account folder), and optional
interest-bearing setup (rate, accrual frequency, linked income account).

**Reports** — Profit & Loss and Balance Sheet, both date-range/as-of
selectable and grouped by category.

**🔔 Reminders** — interest due within 7 days (or overdue) on
interest-bearing accounts, plus any custom reminders you've set; post a
pre-filled interest entry or mark a custom reminder done in one tap.

**🌙/☀️ Dark mode**, remembered per device.

**Same layout everywhere** — one header (brand + bell + sync status +
theme toggle, then nav buttons) on mobile and desktop alike.

**Installable, works offline** — Add to Home Screen / desktop. The app
shell is cached so it opens with no signal; journal entries, accounts,
details, and document uploads are all saved to your device the instant
you submit them and sync automatically once you're online.

## How data loss is avoided

- Every journal entry, category, account, custom detail, document, and
  supporting document is written to **this device's storage the
  instant you submit it**, before any network call happens. Entries
  and accounts (including any attached files) use IndexedDB, since
  file data needs more headroom than `localStorage` offers.
- A background sync loop retries every ~45 seconds, and immediately
  when the tab regains focus or the device comes back online.
- Everything carries a unique client-generated ID, so a retried sync
  never creates a duplicate row — or a duplicate Drive upload — even
  if an earlier attempt actually succeeded but its response was lost.
- Custom details, documents, and supporting documents wait for their
  parent record (account or entry) to be confirmed saved before syncing
  themselves, so nothing ever attaches to the wrong (or a
  not-yet-existing) record.
- The 60-second request cache never delays your own writes — pending
  items you've just added always show immediately from local storage,
  regardless of cache state. The cache only affects how often *other*
  already-synced data is re-fetched, and is cleared automatically the
  moment something you changed actually finishes syncing.
- **Automatic weekly backups** (see above) give you an independent,
  point-in-time copy of the whole Sheet, separate from the live sync
  system entirely.
- The sync stamp shows amber while queued/syncing, **red if something
  has been stuck for a while despite being online**, and green with a
  timestamp once everything is confirmed saved.

## Security note on the passphrase

The passphrase only exists inside `Code.gs`, on Google's servers — never
in the public `index.html`. Once you unlock a device, the webpage stores
a private token (not the passphrase) in that browser's storage, checked
on every write after that. Clearing that storage, or a new device,
means entering the passphrase again.

## Notes & limits

- Built for personal / single-person use across your own devices.
- Apps Script free-tier quota (~20,000 executions/day) is far beyond
  normal personal bookkeeping use, and the new request caching gives
  extra headroom on top of that.
- Interest estimates are simple (principal × rate ÷ periods) — always
  verify against your actual statement before posting.
- Documents and supporting documents are capped at 8MB per file.
- Editing details/documents on the Ledger page requires a live
  connection (these are direct saves, not offline-queued, since
  they're changes to already-synced records) — if offline, you'll see
  a clear error asking you to try again once you're back online.
- Profit & Loss and Balance Sheet are computed live from your entries
  each time you open them — no separate "closing" step needed, but for
  very large numbers of entries (thousands+) they may take a moment
  longer to load than the other pages.
- Responsive down to small phone screens; identical behavior on mobile
  and desktop, with keyboard field-navigation (Enter/↑/↓) on desktop.
