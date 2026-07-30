# JournalDesk — personal accounting webapp backed by Google Sheets

A double-entry journal + account-wise ledger app, installable as a PWA
and usable offline for both data and document uploads. Backed entirely
by a Google Sheet (data) and Google Drive (documents).

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
  "Supportings" subfolder — both under
  https://drive.google.com/drive/u/0/folders/1sKBik3-ZmiVLG2k_NfxgcIhI6dmv2YQi

## Redeploying this update

`Code.gs` changed again (edit/delete for account details & documents,
plus the new `SupportingDocuments` sheet):

1. Paste the updated `Code.gs` into Extensions > Apps Script, replacing
   the old code. Save.
2. Deploy > Manage deployments > pencil icon > Version: **New version**
   > Deploy.
3. Push the updated `index.html` to your repo root (other files
   unchanged this round, but fine to re-push all of them).

Existing accounts, entries, categories, and documents are untouched.

## What's new this round

- **Bell + sync status** moved to the top right next to the dark-mode
  toggle, visible on every page.
- **Ledger page is now editable.** Click **✎ Edit** on the account
  details panel to update or delete existing details/documents, or add
  new ones — right from the Ledger page, no need to go back to Accounts.
- **Smaller close buttons** on journal entry lines — a compact "×" at
  the end of each line instead of a large button.
- **Supporting documents on journal entries.** A "📷 Add supporting
  document" button opens your camera (or file picker on desktop) to
  attach a receipt/bill photo to an entry. It uploads to a shared
  "Supportings" Drive folder, named after the entry's voucher number.
- **Voucher numbers are now always auto-generated** — the manual field
  is gone. Each entry's voucher number doubles as its unique ID and its
  supporting document's filename prefix.
- **Faster loading.** Dashboard, New Entry, and Ledgers now paint
  instantly from your device's local cache, then quietly refresh from
  the Sheet in the background — no more waiting on a network round trip
  before you can start typing.
- **Red sync status.** The sync stamp turns red if something has failed
  to sync for a while *despite being online* (as opposed to normal
  amber, which just means it's still catching up or you're offline).
  This is your signal to check your connection or come back to this
  device later.
- **Keyboard navigation (desktop).** Press Enter, ↓, or ↑ in any field
  to jump to the next/previous field — handy for fast data entry
  without reaching for the mouse.

## Full feature list

**Journal entries** — classic accountant Dr / To layout, auto-numbered
vouchers, optional supporting document capture, type-to-search account
fields grouped by category.

**Ledgers** — search and pick an account to see an editable details
panel (head, category, account number, interest info, custom details,
document chips) above the full transaction history with running
balance. Defaults to the current financial year (1 April) through
today.

**Accounts & categories** — group accounts under each head. When adding
an account: unlimited custom details (CIF, IFSC, address, anything),
document uploads (Drive, per-account folder), and optional
interest-bearing setup (rate, accrual frequency, linked income account).

**🔔 Reminders** — badge for interest due within 7 days (or overdue) on
interest-bearing accounts; post a pre-filled entry in two clicks.

**Dashboard** — totals by head and your most recent entries.

**🌙/☀️ Dark mode**, remembered per device.

**Same layout everywhere** — one header (brand + bell + sync status +
theme toggle, then four nav buttons) on mobile and desktop alike.

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
- A background sync loop retries every ~20 seconds, and immediately
  when the tab regains focus or the device comes back online.
- Everything carries a unique client-generated ID, so a retried sync
  never creates a duplicate row — or a duplicate Drive upload — even
  if an earlier attempt actually succeeded but its response was lost.
- Custom details, documents, and supporting documents wait for their
  parent record (account or entry) to be confirmed saved before syncing
  themselves, so nothing ever attaches to the wrong (or a
  not-yet-existing) record.
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
  normal personal bookkeeping use.
- Interest estimates are simple (principal × rate ÷ periods) — always
  verify against your actual statement before posting.
- Documents and supporting documents are capped at 8MB per file.
- Editing details/documents on the Ledger page requires a live
  connection (these are direct saves, not offline-queued, since
  they're changes to already-synced records) — if offline, you'll see
  a clear error asking you to try again once you're back online.
- Responsive down to small phone screens; identical behavior on mobile
  and desktop, with keyboard field-navigation on desktop.
