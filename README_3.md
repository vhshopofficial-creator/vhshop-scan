# VH Shop Scan Verify 1.1

Two single-file pages on GitHub Pages plus one Firebase project. No build step.

| File | Who | What |
|---|---|---|
| `index.html` | Staff phones | Scan order → scan items. Green = right, red = wrong. Hold, Ship short (tick the missing items + note; ask manager or supervisor PIN), ⚡ technical quirk, first-scan confirm, dispute. |
| `upload.html` | Manager (laptop) | Upload PokPok list (full or search export), Vrich stock, Vrich orders. Fix list, 🔗 Match, Same words, ship-short approvals + follow-up, ⚡ quirk review, disputes, staff accuracy, ⬇ scan export, staff + PIN settings. |
| `firestore.rules` | Setup once | Paste into Firebase. |

There is no Catalog Cleaner any more — `upload.html` does its cleaning and saves straight to Firestore. Nobody needs GitHub for daily work.

---

## 1 · Setup (once, ~20 minutes)

**Firebase** — console.firebase.google.com
1. Add project → `vh-shop-scan` (Analytics off).
2. Build → **Firestore Database** → Create → region **asia-southeast1 (Singapore)** → Production mode.
3. Build → **Authentication** → Get started → turn on **Email/Password** and **Anonymous**.
   Users → Add user → the manager email + password.
4. Firestore → **Rules** → paste `firestore.rules`, the manager email (vhshopofficial@gmail.com) is already filled in → Publish.
5. Project settings ⚙ → Your apps → Web `</>` → Register → copy the `firebaseConfig` values.

**Config** — already filled in for project `vh-shop-scan` in both files (`BUILT_IN_FIREBASE_CONFIG`).

**GitHub** — new repository `vhshop-scan` → upload `index.html` and `upload.html` → Settings → Pages → Deploy from branch `main` / root.
- Staff: `https://<user>.github.io/vhshop-scan/`
- Manager: `https://<user>.github.io/vhshop-scan/upload.html`

Use a different URL from VHM's app and don't share a phone between the two companies' apps.

**First upload** — open `upload.html`, log in, then:
1. Staff & supervisors → type staff names → Save. Add each supervisor with a 6+ digit PIN.
2. Upload slot 1 (PokPok list) → Save, slot 2 (Vrich stock) → Save, slot 3 (orders) → Save.

**Test before real use** — on one phone: scan ~20 orders including one wrong item, one duplicate, one unknown label, one Hold and one PIN approval. Check *Missing codes & holds* and *Staff accuracy* on the upload page. Then roll out to all phones.

---

## 2 · Every day · ប្រចាំថ្ងៃ

**Manager**
- Before packing: export **Vrich orders** → upload slot 3. Re-uploading later in the day only writes new/changed orders.
- Open **To fix → Orders that can't scan**. Pick the **Order date** (date box), or tap one of the last 7 days (buttons show how many lines can't scan). Each day is checked with the current PokPok list, **Same words** and 🔗 links — saving Same words, a Match link or a new PokPok list re-checks every day at once (**⟳ Re-check all days** does it by hand). Send the day's list (⬇ CSV) to the PokPok / Vrich person. Goal: 0 before packing.
- After they fix things: export again → upload only that slot. For one or a few PokPok codes, **search the code in PokPok and export just those** — the page adds/updates only them. The page shows *fixed / still broken* and asks you to confirm any barcode that would now mean a different product.
- **Wrong order file uploaded?** Choose that same wrong file again in slot 3 → *Uploaded this file by mistake? Undo it* → **Remove these orders**. Only the orders that are online exactly as in that file are removed (orders, daily order list, the day in *Orders that can't scan*). Then upload the right file.
- **✓ Allow by code** (on a *Colour/size words differ* row, e.g. Vrich `LC069 កាបូប` = the last piece, no colour): any PokPok colour/size of that code then counts for its order lines. Every time it is used you get a notice (*Missing codes & holds → Allowed by code — what staff packed*, yellow banner) with the label that was packed → check it, tap **Seen**. Remove the allowance under *✓ allowed by code* when the piece is gone.
- Words that mean the same (e.g. `ស = សរ`) → **Same words** box. A PokPok item whose words can't match Vrich → **🔗 Match** it to the Vrich stock code. PokPok's forced Size number (`-1 -កាហ្វេ`) is ignored automatically when Vrich has no number.
- **Missing codes & holds** tab (red banner when needed): approve/reject **ship-short requests** with a note (you see exactly which items staff ticked as missing and their note); for shipped-short orders tick **Fixed in Vrich** and **Customer told**; judge disputes (*catalog problem* or *staff mistake*).
- **⚡ Technical quirks** (same tab, red banner): staff counted an item they say is correct but the system would not take. Check the scanned label → **Quirk confirmed** (fix PokPok / 🔗 Match) or **Staff mistake** (fix the parcel; counts as a mistake in Staff accuracy).
- **Staff accuracy & export** tab → **⬇ Export scans**: choose dates → one row per scanned order (staff, result, correct / wrong / duplicate / quirk counts) with **Through manager = YES** and every manager / supervisor step. Download all orders, or only the ones that went through the manager.

**Staff · បុគ្គលិក**
- ស្កេនលេខ Order → ស្កេនទំនិញម្តងមួយៗ · Scan the order number, then each item.
- 🟢 ត្រូវ · correct. 🔴 ខុស · take the item shown under EXPECTED. 🟣 ស្កេនស្ទួន · check it.
- ? មិនមានក្នុងបញ្ជី · unknown label → tap **⏸ Hold**, take the next order. The Issues tab turns green with “rescan” when the manager has uploaded the fix.
- Issues cannot be cleared by staff — they close by themselves when the order is scanned complete. Tap **📦 Open order** on the issue to finish it.
- Item missing → **📦 Ship short** → **tick each missing item** (and how many) and **write why** — both required → *Ask the manager* (no PIN; the phone shows the answer) or *Ship short now* with a supervisor PIN. Only the ticked items are left out: scan everything else and the order completes by itself. After the manager approves, scan the order again and scan the other items.
- Item is right but the system says wrong / not in catalog → **⚡ Item is correct — system quirk** → choose the line, write what is wrong → it counts now. Use only after checking by eye: the manager checks every quirk.
- Think the system is wrong? Tap **System is wrong?** on the issue — the manager decides; it doesn't count against you meanwhile.
- Always scan the PokPok label (`P00…`), not the factory barcode on the box.
- 🆕 New/changed label → look at the item and tap ✓ or ✗ (only once per label).
- 🔑 Supervisor PIN → approve one item for one line, or close an issue as *Mistake scan* / *Other* (note). Never changes the catalog.
- ⛔ CANCELLED → don't pack. ⚠ ORDER CHANGED → re-check every item.

---

## 3 · Where the data sits

| Firestore collection | Content | Kept |
|---|---|---|
| `scan_catalog` | `meta` + compressed PokPok and Vrich chunks | replaced each upload |
| `scan_orders` | 1 doc per order (no address/phone) | 35 days |
| `scan_orders_idx` | 1 doc per day: order list + change check | 35 days |
| `scan_log` | 1 doc per order scanned (all its scans) | 35 days |
| `scan_issues` | wrong / duplicate / unknown / hold | cleared ones 35 days, open ones kept |
| `scan_overrides` | supervisor approvals | 35 days |
| `scan_meta` | settings (staff, PIN hashes), `rd_YYYY-MM-DD` = each order day's line list for *Orders that can't scan* + `rdays` (counts per day), links, confirmations, daily staff stats | day lists 35 days |

Phones keep: catalog copy (IndexedDB `vhs_scan`), opened orders, own scan log, unsent logs (localStorage `vhs_…`).

**Free-plan use at ~4,300 orders/day, 4 phones:** ≈ 9,000 writes (limit 20,000), ≈ 5,000–10,000 reads (limit 50,000), ≈ 300 MB stored (limit 1 GB). Viewing *Staff accuracy* for today costs ~1 read per scanned order; a finished day costs 1 read.

## 4 · Settings in the code
- `VHS.KEEP_DAYS = 35`, `VHS.SHRINK = 0.5` (a PokPok file with >50% fewer items is treated as a search export and merged; a Vrich stock file is blocked), Cambodia time `TZ_MS` (+7h) — in the shared core block, identical in both files.
- Column names are matched by header text (`DEF` in `upload.html`). Sale code = first word of the PokPok name. A PokPok item matches an order line when every colour/size word of the PokPok attributes appears in the Vrich description.
- A dot, dash or underscore right after the code in the PokPok name is ignored: `G872.គូសភ្នែក` and `H272. អាវ` have sale code `G872` / `H272`.
- **Size in the Vrich code (shoes):** Vrich `7H38 ស្បែកជើង` = PokPok `7H ស្បែកជើង` with size `38`. When a Vrich code is not a PokPok sale code, it is read as model + size number, and only the PokPok item of that model with that size matches (another size shows *right product, WRONG size*). If only one PokPok item has that model + size, an extra colour word in PokPok (e.g. `KH -ត្នោត -36`) does not block it.

**Security note:** the Firebase web config is visible in the page, and anonymous sign-in lets any browser with the link read orders and catalog. Only the manager account can change them. Optionally restrict the API key to your GitHub Pages domain (Google Cloud console → Credentials → API key → Website restrictions).
