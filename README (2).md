# Ledger — personal accounting webpage backed by Google Sheets

A double-entry journal + account-wise ledger app, with account
categories, interest-accrual reminders, and a classic accountant-style
entry form. Just **2 files**:

| File | Where it goes |
|---|---|
| `index.html` | Your GitHub repo (this is the whole webpage — single file, no build step) |
| `Code.gs` | Pasted into your Google Sheet's Apps Script editor |

## Your setup (already done)

- **Webpage:** https://canitish780.github.io/journaldesk/
- **Apps Script Web App URL:** already wired into `index.html`
- **Passphrase:** `trusttheprocess` (set in `Code.gs`)

## What's new: categories + classic journal entry

**Categories.** On the Accounts page, a new "Categories" panel lets you
group accounts under each head — e.g. under Assets: *Savings Accounts*,
*Fixed Deposits*, *Gold*, *Land*. Categorizing is optional; uncategorized
accounts still show up fine. Every account dropdown (New Entry, Ledger
filter) now groups accounts by "Head — Category" so a long account list
stays navigable. The "All accounts" list on the Accounts page is grouped
the same way.

**Classic journal entry layout.** The New Entry form now mirrors how
accountants write journal vouchers on paper:

```
Debit
  Rent A/c                    Dr.   5,000
  Travel Expense A/c          Dr.   2,000

Credit
  To Cash A/c                        7,000
```

Add as many debit lines and credit lines as you need (compound entries
are fine) — the two sides just need to balance before you can post,
same as before. The Dashboard and Ledger pages still show plain
Debit/Credit columns, per your preference.

## Redeploying after this update

`Code.gs` changed again (new `Categories` sheet, optional `CategoryId`
on accounts):

1. Paste the updated `Code.gs` into your Sheet's Apps Script editor,
   replacing the old code. Save.
2. Deploy > Manage deployments > pencil icon > Version: **New version**
   > Deploy. (Same URL — already wired into `index.html`.)
3. Push the updated `index.html` to your `journaldesk` repo.

Existing accounts, entries, and interest settings are untouched — the
sheet gets the new `Categories` tab and column automatically, without
disturbing existing rows.

## Using it

1. **Accounts** — optionally add categories per head, then add accounts
   (assign a category if you like). For Assets, optionally mark an
   account interest-bearing (rate, accrual frequency/date, linked
   income account).
2. **New entry** — post journal entries in the classic Debit/Credit
   layout. Account dropdowns are grouped by category.
3. **Ledgers** — pick any account (grouped dropdown) to see its full
   history with a running balance, filterable by date range.
4. **Dashboard** — totals by head and your most recent entries.
5. **🔔 Reminders** — click any time to see upcoming/overdue interest
   and post it in two clicks.

## How data loss is avoided

- Every journal entry and new account is written to **browser storage
  the instant you submit it** — before any network call happens.
- A background sync loop retries every ~20 seconds, and immediately
  when the tab regains focus or the device comes back online.
- Every entry and account carries a unique client-generated ID, so a
  retried sync never creates a duplicate row even if an earlier attempt
  actually succeeded but its response was lost.
- New categories sync immediately when you add them (so they're usable
  right away); if you're offline, they queue like everything else and
  become selectable once synced.
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
- Interest estimates are simple (principal × rate ÷ periods) and meant
  as a starting point — always verify against your bank/FD statement
  before posting, especially where TDS or compounding applies.
- The app is responsive down to small phone screens: tables scroll
  horizontally instead of breaking layout, the journal form stacks
  cleanly, and the Accounts page uses a two-column layout on wider
  screens.


