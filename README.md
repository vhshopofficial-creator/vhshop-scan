# VH Shop Scan Verify — version 1.4

Updated 24 Nov 2026.

## What's in this folder

| File | What it is | Changed in 1.4? |
|---|---|---|
| `index.html` | **Scanner page** for the staff phones | ✅ Yes |
| `upload.html` | **Upload page** for the manager (uploads + reports) | ✅ Yes |
| `count.html` | Stock count page | No |
| `txt.json` | Data file | No |
| `README.md` | This file | New |

## How to put it live

1. Replace `index.html` and `upload.html` wherever the site is hosted (for example the GitHub repository). Keep `count.html` and `txt.json` as they are.
2. Open the scanner page on each phone once and reload it, so the phone picks up the new version.
3. Nothing to change in Firebase. No new rules are needed.

The first time the upload page runs its daily clean-up, it deletes all the old issues that piled up. This happens once, by itself.

---

## What changed

### 1. Issues are gone: staff pick a reason
- Nothing is saved as an "issue" in Firebase any more, so issues can't pile up.
- No manager approval and no supervisor PIN are needed.
- When something is not normal, staff pick a reason (see the list below). The reason is saved on the order in the shift list.

### 2. Shift list on the phone (🧾 Shift tab)
- Every order scanned during the shift appears with its **order number, items (packed / ordered), price total and issue** if it has one.
- At the top: orders scanned · complete OK · with issues · pieces packed · 💲 total shipped.
- The newest 100 orders show first; the **Show all** button shows the rest.

### 3. Submit shift → goes straight to the upload page
- At the end of the shift, staff tap **✅ Submit shift · បញ្ចប់វេន**.
- The whole list is uploaded as **one small compressed document** (about 10 KB for 1,000 orders). No files and no Telegram are involved.
- The phone is then cleared for the next shift.
- If there is no internet, the list stays safe on the phone and uploads by itself every minute. There is also a **☁ Upload now** button.
- If an order is still half-scanned, the phone asks staff to finish it or ⏸ hold it first.

### 4. Upload page: 🧾 Scanners tab
One section per scanner:
- Their shift times, orders scanned, pieces shipped, 💲 shipped and number of issues.
- Their **⚠ orders with issues**: order #, customer, items, total, result, reason.
- **📋 All their orders**: folded until you open it; shows up to 300 rows.
- A CSV download for each scanner, and one for all orders.

The **🔎 search box** finds an order #, customer or reason and opens the right scanner's list.

An order held by one scanner and finished by another counts for the one who finished it.

### 5. Upload page: 📊 6-day summary
6 days is one cycle of about 3 lives. For each day it shows:
- Vrich orders, pieces, 💲 sold (Vrich price).
- Lines that can't scan.
- Orders scanned, 💲 shipped, orders with issues.
- Orders **not scanned yet**.

It also shows scanned orders and issues **by who**, with a CSV download.

### 6. Other changes
- The upload page "To fix" area now has only: **Orders that can't scan · 🧾 Scanners · 📊 6-day summary · Catalog problems**.
- Removed: the old issue review, ship-short / pack-again approvals, staff accuracy and the old scan export. These read thousands of documents.
- **Data is kept 7 days** (was 35): orders, scan logs, shift lists and old issues are deleted automatically once a day.
- The scanner page is now in **day mode** (light colours).
- Built for **1,000+ orders a day**: the phone stays fast. Each scan takes about 2 ms of work, and opening the Shift tab about 7 ms.

---

## Reasons staff can choose

Every list also has a note box. The note is optional, except for **Other**, which needs it.

**⏸ Hold**
- រកអត់ឃើញ · Cannot find
- អីវ៉ាន់ចូលខ្វះ · Stock in late
- អីវ៉ាន់ខូច · Damaged goods
- ស្លាកមិនមានក្នុងប្រព័ន្ធ · Label not in the system
- អតិថិជនឲ្យរង់ចាំ · Customer asked to wait
- ផ្សេងៗ · Other (write why)

**📦 Ship short** (staff also tap the missing item(s))
- រាប់ខុស · Miscount
- រកអត់ឃើញ · Cannot find
- អីវ៉ាន់ខូច · Damaged goods
- អីវ៉ាន់ចូលខ្វះ · Stock in late
- អីវ៉ាន់សល់តិចៗ · Left-over pcs

**🔁 Pack again** (the order was already packed)
- អតិថិជនប្តូរ Order · Customer changed the order
- កញ្ចប់ខូច · Box damaged
- យកទំនិញចេញ · Items were taken out
- ផ្សេងៗ · Other (write why)

**⚡ System quirk**: staff type what the quirk is (required).

> The Khmer for the new Hold and Pack-again reasons should be checked by a Khmer speaker.

---

## Firebase usage

Measured with a test copy of Firebase that counts usage the way Firebase bills it. These are not real-Firebase numbers.

| | Old version | Version 1.4 |
|---|---|---|
| Open the scanner page | ~505 reads | 5 reads |
| Each order scanned | ~3.3 reads + 1.3 writes | 2 reads + 1 write |
| Submit a shift | — | 1 write |
| Open the upload page | ~1,012 reads | 6 reads |
| 🧾 Scanners tab | ~503 reads | 1 read per shift |
| **Estimated per day (1,000 orders)** | **~40,000 reads** | **~2,200 reads, ~1,010 writes** |

The Firebase free daily limit is 50,000 reads and 20,000 writes. Uploading the Vrich order file still costs about 1 write per order, the same as before.

You can check the real numbers in the Firebase console → Firestore → Usage.

---

## Technical notes (for whoever maintains the code)

**Shift documents**
- Each shift is saved in `scan_log` as `shift_<id>`, with `kind: 'shift'` and readable totals.
- The order rows are gzipped in the field `z`. Each row is:
  `[order, customer, result, time, staff, issue kinds, reason, scans not counted, pcs ordered, pcs packed, total packed, order total, codes not in PokPok]`
- `scan_log` is used because phones can already write there, and the daily clean-up deletes it by `d`.
- Phones only look up `scan_log` by `order`, so they never read shift documents.

**Shared code**
- The `VHS core` block must stay **identical** in `index.html` and `upload.html`.
- `KEEP_DAYS = 7` lives there.

**Limits on the phone**
- The offline order copy keeps the newest 300 orders.
- The scan log keeps 1,500 lines.
- An uploaded shift keeps only its totals on the phone (for 7 days).
