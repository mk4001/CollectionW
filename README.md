# Collection Weapons

**A private, cross-platform catalog for a firearms, accessories, and ammunition collection — and, since 2.0, a full log of every shooting session.**

macOS · iPadOS · iOS — one app, one private iCloud database, no server, no account, no tracking.

- **Version**: 2.0 (build 25)
- **Platforms**: macOS 26+, iPadOS 26+, iOS 26+ — single Xcode multiplatform target, one App Store listing
- **Languages**: 11 — Italiano, English, Français, Deutsch, Español, 简体中文, Русский, Português (Brasil), العربية, Türkçe, ไทย
- **Storage**: SwiftData, synced through the user's own private CloudKit database (`iCloud.MF-Corp.CollectionWMacOS`)
- **Publisher**: MF-Lab

This document is the support-facing reference for everything the app does. It mirrors the internal architecture decision log (`PIANO_ARCHITETTURA.md`) but is organized by feature rather than by chronology, and leaves out internal implementation narrative (bug write-ups, reverted experiments) that isn't relevant to using or supporting the app.

---

## Table of contents

1. [Overview](#overview)
2. [Requirements](#requirements)
3. [The collection: record types](#the-collection-record-types)
   - [Weapons](#weapons)
   - [Accessories](#accessories)
   - [Ammunition — New](#ammunition--new)
   - [Ammunition — Reloaded](#ammunition--reloaded)
   - [Kits](#kits)
   - [Maintenance](#maintenance)
   - [Shooting Sessions](#shooting-sessions)
   - [Shooter Profile & Membership Cards](#shooter-profile--membership-cards)
4. [Photos](#photos)
5. [Organizing and finding records](#organizing-and-finding-records)
6. [Data entry tools (macOS)](#data-entry-tools-macos)
7. [Export and backup](#export-and-backup)
8. [Analytics](#analytics)
9. [Privacy and security](#privacy-and-security)
10. [Appearance and language](#appearance-and-language)
11. [Help and documentation](#help-and-documentation)
12. [Collection Weapons Pro (subscription)](#collection-weapons-pro-subscription)
13. [Platform differences at a glance](#platform-differences-at-a-glance)
14. [Currency handling](#currency-handling)
15. [Architecture and tech stack](#architecture-and-tech-stack)
16. [Privacy policy and support](#privacy-policy-and-support)

---

## Overview

Collection Weapons catalogs a firearms collection the way a serious collector actually tracks one: weapons, accessories, new and reloaded ammunition, kits (loadouts that group a weapon with its accessories), and a maintenance log — each with its own record type and its own fields, not a single generic "item" model stretched to fit everything. Since version 2.0, it also logs every trip to the range as a Shooting Session — with optional GPS, weather, and score tracking — and keeps a Shooter Profile with membership cards and Apple Wallet passes, extending the same private, cross-platform treatment from collecting to shooting.

Every record syncs automatically across a user's Mac, iPhone, and iPad through their own private iCloud account. There is no sign-up, no MF-Lab server, and no way for MF-Lab to see a user's data — see [Privacy and security](#privacy-and-security).

The app ships as **one single Xcode multiplatform target** (not two separate macOS/iOS apps), so it appears as one product on the App Store with both Mac and iPhone/iPad support, and a Pro subscription purchased on either platform unlocks the other automatically.

## Requirements

- macOS 26 or later, on Apple Silicon or Intel (biometric lock falls back to the account password automatically on Macs without Touch ID)
- iPadOS 26 or later / iOS 26 or later
- An iCloud account signed in on the device, for sync (the app also works fully offline on a single device without iCloud, with no cross-device sync)

## The collection: record types

Every field below is optional at the database level — SwiftData/CloudKit compatibility requires it — but each record type has a short list of fields required to count as **complete** rather than a **draft** (shown with a visible badge in every list, and flagged but never blocked during bulk entry or CSV import).

### Weapons

The core record type. Required for "complete": **Brand, Model, Type, Category**.

| Field | Type | Notes |
|---|---|---|
| `marca` — Brand | Text | Autocomplete |
| `modello` — Model | Text | Autocomplete |
| `tipo` — Type | Enum | Pistol, Barrel, Rifle, Revolver, Carbine, Conversion, Other (free text if Other) |
| `categoria` — Category | Text | Autocomplete — regulatory/use classification (e.g. "common firearm", "war weapon", "antique") |
| `statoConservazione` — Condition | Enum | A 4-step scale: **N** New, **1** Almost New, **2** Visible Signs of Wear, **3** Significant Signs of Wear |
| `calibro` — Caliber | Text | Autocomplete — free text, not a closed list (`.357 Magnum`, `9x21`, etc.) |
| `lunghezzaCannaTesto` — Barrel length | Text | Autocomplete, free text — accepts any notation including fractional inches (`8 3/8"`, `6 1/2"`) or millimeters |
| `numeroColpi` — Round capacity | Integer | |
| `caricatori` — Magazines | Text | |
| `matricola` — Serial number | Text | **Not unique** — historical/antique weapons may lack one or share one |
| `numeroSerieCanna` — Barrel serial number | Text | Only shown for Type = Barrel. Fully independent of `matricola` — two distinct identifiers, never merged or cross-derived |
| `annoProduzione` — Year of manufacture | Year only | |
| `prezzoAcquisto` / `valutaAcquisto` — Purchase price + currency | Number + text | See [Currency handling](#currency-handling) |
| `dataAcquisto` — Purchase date | Date | |
| `acquistatoDa` — Purchased from | Text | |
| `valoreCommerciale` / `valutaCommerciale` — Current market value + currency | Number + text | Used throughout Analytics |
| `prezzoVendita` / `valutaVendita` — Sale price + currency | Number + text | Presence of a sale date marks the weapon as **sold** (`isSold`) — sold weapons drop out of Analytics totals |
| `dataVendita` — Sale date | Date | |
| `vendutoA` — Sold to | Text | Autocomplete |
| `note` — Notes | Multiline text | |
| Photos | 1-to-many | See [Photos](#photos) |
| Kits | Many-to-many | See [Kits](#kits) |
| Maintenance history | 1-to-many | See [Maintenance](#maintenance) |

### Accessories

Required for "complete": **Category, Subcategory, Brand**.

| Field | Type | Notes |
|---|---|---|
| `categoria` — Category | Text | Autocomplete (Reloading, Magazine, Speedloader, Leather goods, Accessories, …) |
| `sottocategoria` — Subcategory | Text | Autocomplete |
| `marca` — Brand | Text | Autocomplete |
| `modello` — Model | Text | Autocomplete |
| `calibro` — Caliber | Text | Autocomplete |
| `quantita` — Quantity | Integer | |
| `prezzoAcquisto` / `valutaAcquisto` — Purchase price + currency | Number + text | |
| `dataAcquisto` — Purchase date | Date | |
| `valoreCommerciale` / `valutaCommerciale` — Market value + currency | Number + text | |
| `prezzoVendita` / `valutaVendita` — Sale price + currency | Number + text | |
| `dataVendita` — Sale date | Date | |
| `vendutoA` — Sold to | Text | Autocomplete |
| `note` — Notes | Multiline text | |
| Photos, Kits | relations | |

### Ammunition — New

Factory ammunition. There is **no static "in stock" field** — stock level is always *derived* from the transaction log below, so it can never drift out of sync with reality.

| Field | Type | Notes |
|---|---|---|
| `calibro` — Caliber | Text | Autocomplete, **shared bidirectionally** with Weapons/Accessories/Reloaded |
| `marca` — Brand | Text | Autocomplete |
| `modello` — Model | Text | Autocomplete |
| `tipo` — Bullet type | Text | Autocomplete, free text (FMJ, Hollow Point, Semi-Wadcutter, …) |
| `grain` — Bullet weight | Integer | Grains |
| `note` — Notes | Multiline text | |

**Transactions** (`MunizioneTransazione`, one-to-many): each movement of stock is its own record.

| Field | Notes |
|---|---|
| `tipo` — Transaction type | `acquisto` (purchase), `ricarica` (reload), `regaloRicevuto` (gift received), `regaloDato` (gift given), `uso` (range use), `vendita` (sale), `aggiustamentoInventario` (inventory correction) |
| `rounds` — Round count | Always entered as a **positive** number; the app derives the direction automatically from the transaction type (purchase/reload/gift-received add to stock, use/sale/gift-given subtract). Inventory correction is the one type that also accepts a negative value directly |
| `data` — Date | |
| `prezzo` / `valuta` — Price + currency | Relevant for purchase/sale |
| `controparte` — Counterparty | Autocomplete — who sold it, who received/gave the gift, etc. |
| `note` — Notes | |

### Ammunition — Reloaded

Full handload recipes, in the same Ammunition section as New, selected with a segmented control. Required for "complete": **Caliber + Bullet brand**.

| Field | Type | Notes |
|---|---|---|
| `calibro` — Caliber | Text | Shared autocomplete with all other calibers |
| `marcaBossoli` — Case brand | Text | Autocomplete |
| `marcaInneschi` — Primer brand | Text | Autocomplete |
| `marcaPalla` — Bullet brand | Text | Autocomplete |
| `pesoPalla` — Bullet weight | Integer | Grains |
| `tipoPalla` — Bullet type | Text | Autocomplete |
| `marcaPolvere` — Powder brand | Text | Autocomplete |
| `dosaggioPolvere` — Powder charge | Decimal | Grains |
| `oal` — Overall length | Decimal | Inches |
| `note` — Notes | | |

Reloaded ammunition uses the **same transaction log and stock calculation** as New ammunition — one shared `MunizioneTransazione` entity, one shared editor.

**Power Factor and chronograph testing** — an entirely optional subsection, further down the same record, for reloaders who also compete in dynamic disciplines: checks a recipe's calculated Power Factor against the official reference tables of four disciplines.

| Field | Type | Notes |
|---|---|---|
| Test Weapon | Reference | One existing Weapon record used for the chronograph test |
| Test Date | Date | |
| Discipline | Enum | IPSC, IDPA, USPSA, LSSA-CSEN — names left untranslated in every language |
| Category | Text, from a list | Options depend on the Discipline chosen, each with its own official reference value |
| Chronograph Readings | Repeatable decimals | One entry per shot, in feet per second — averaged automatically, no fixed limit |
| Power Factor | Calculated | Average reading (fps) × Bullet Weight (grains) ÷ 1000 — the standard IPSC/USPSA formula |

The calculated Power Factor is shown next to the official reference value for the chosen Discipline/Category, highlighted **green** if it meets or exceeds it, **red** if it falls short. Reference tables are fixed in the app (from the official IPSC/LSSA-CSEN rulebooks and the USPSA Competition Rules), not user-editable.

### Kits

A Kit groups an already-catalogued weapon with one or more accessories into a single named loadout (e.g. "Competition kit" = a pistol + holster + spare magazines). A piece can belong to **more than one Kit** at the same time.

| Field | Notes |
|---|---|
| `nome` — Name | Chosen by the user |
| `note` — Notes | |
| Photos | Kit's own cover photos, independent of the pieces it contains |
| `weapons` / `accessories` | Many-to-many — deleting a Kit never deletes the weapons/accessories inside it, and vice versa |

**Estimated value** is computed live as the sum of each piece's market value, converted to the reference currency — it is **never counted in any list total or in Analytics**, so a piece's value is never double-counted once it's also in a Kit. Any piece excluded from the estimate (missing value or exchange rate) is reported explicitly rather than silently ignored.

Kits are included in full backups, but intentionally **not** in PDF/CSV export (a grouping doesn't map cleanly onto a flat spreadsheet row).

### Maintenance

A log of service actually performed on a weapon — not a maintenance *schedule*; there are no due dates or reminders.

| Field | Notes |
|---|---|
| `dataEsecuzione` — Date performed | |
| `tipo` — Type of work | Free text, autocomplete |
| `note` — Notes | |
| `weapon` — Associated weapon | Required |

Each weapon's detail screen shows its **most recent maintenance date**, computed automatically from this log (never a field to keep manually in sync), with a relative "time since" readout and the full history, navigable entry by entry.

### Shooting Sessions

A record of one outing to the range or the field — what you shot, with what, how many rounds, and, optionally, where, in what weather, and how you scored. Unlike every other record type, several fields can be captured automatically from the device instead of typed in by hand. No field is required to save the record.

| Field | Type | Notes |
|---|---|---|
| `tipoSessione` — Type | Enum | 17 entries: 9 disciplines/competitions (IPSC, USPSA, IDPA, LSSA-CSEN, 3-Gun, Steel Challenge, Cowboy Action/SASS, PRS/Long Range, Trap/Skeet/Sporting Clays — names left untranslated), 7 generic types (Training/Recreational, Precision, Personal Defense, Bench Rest, Hunting, Optic Zeroing, Qualification), plus Other (free text) |
| Weapon / Kit | Reference | Mutually exclusive — a session uses either one Weapon or one Kit, never both |
| Ammunition | Reference | New or Reloaded, mutually exclusive with each other |
| `data` — Date | Date | Defaults to the current date/time when the record is created |
| `colpiSparati` — Rounds fired | Integer | Automatically deducted from the selected Ammunition's stock as a Use transaction, kept in sync if edited later, removed if cleared |
| `distanza` — Distance | Decimal + unit | Meters or feet — defaults to the device's own measurement system |
| `costo` — Cost | Number + currency | Counted in Shooting Analytics (see [Analytics](#analytics)), not in the collection's main totals |
| `note` — Notes | Multiline text | |
| Score | Repeatable decimals | One row per series/stage, summed into a running total — deliberately excluded from every PDF/CSV export |
| GPS location | Optional, off by default | Address + coordinates, captured once when turned on; tap to open in Maps |
| Altitude | Optional | From the device's own barometric sensor (iPhone 6 and later), not GPS-derived — no fallback where unavailable |
| Weather | Optional, off by default | Temperature, humidity, wind speed/direction, atmospheric pressure — fetched for the session's location and moment |
| Calendar event | Relationship | Auto-created, updated, and removed in sync with the session, in whichever calendar is chosen in Settings |
| Photos/Videos | 1-to-many | **iPhone/iPad only** — captured directly from the session's own camera screen and saved to the device's system Photos library, not duplicated into this app's iCloud database |

Location is treated as sensitive by default: GPS and weather are each their own off-by-default toggle, and turning one on affects only that single session.

### Shooter Profile & Membership Cards

The Shooter Profile is a **single record** — your own details, not a list like every other section — created automatically the first time you open it.

| Field | Type | Notes |
|---|---|---|
| First Name / Last Name | Text | |
| Date of Birth | Date | |
| Address / Phone / Email | Text | |
| Photo | One image | |

Details can be typed in by hand or imported once from the device's own Contacts app (specifically your "My Card" entry) — a one-time copy, not a live link, so a later change to My Card doesn't update what's already been imported.

**Membership Cards** (`Tessera`) are a repeatable list under the profile — one entry per membership, license, or permit (a range membership, a federation card, a carry license, and so on):

| Field | Type | Notes |
|---|---|---|
| Name | Text | |
| Number | Text | Alphanumeric, free text |
| Expiration | Date | Drives an automatic reminder |
| Cost | Number + currency | |

Setting an Expiration date creates a Reminder in the device's own Reminders app automatically, kept in sync when the card is edited or deleted — the same auto-sync principle used for Shooting Sessions' calendar events above.

**Apple Wallet**: from a card's own detail screen, **on Mac only**, a "Generate Pass for Wallet" button builds a real `.pkpass` file (Name, Number, Expiration, a QR code encoding the Number, and the profile photo) and offers to share it — AirDrop it to an iPhone, where it's recognized natively with an "Add to Wallet" prompt. Generating and signing a pass requires macOS specifically; iPhone/iPad can only **receive** an already-generated pass via AirDrop, not create one.

The Shooter Profile has its own simplified PDF export (Simple or Photographic template only, no custom templates) — see [Export and backup](#export-and-backup).

## Photos

Every record type (Weapons, Accessories, both Ammunition types, Kits) carries the same photo gallery component:

- **Add** from the Photo Library, the Files app / Finder, or — **on iPhone and iPad** — directly from the **camera**, locked to portrait orientation so the capture UI never fights the device's grip.
- **Paste from the clipboard** (⌘V on Mac, or a dedicated Paste button on both platforms) — a photo copied from Safari or anywhere else drops straight into the gallery, compressed the same way as any other source.
- **Cover photo**: scrolls horizontally between multiple photos *and* pinch-zooms in place — both gestures coexist, gated so they never fight each other.
- **Full-screen viewer**: pinch-to-zoom anchored to the exact point the gesture starts (not always re-centering), double-tap to reset, swipe between all photos of a record. **On Mac**, double-clicking a photo instead opens it in its **own resizable native window** — several photos can stay open side by side.
- **Compression**: every photo is resized to a 2048px long edge and saved as JPEG at 0.8 quality (via `ImageIO`, which handles modern HDR "gain map" photos correctly) before being stored — keeps iCloud usage and sync time down without a visible quality loss.

**Shooting Sessions use a different mechanism** (iPhone/iPad only): photos and videos are captured directly from the session's own camera screen and saved straight to the device's system Photos library, rather than compressed and stored in this app's iCloud database like the gallery above — nothing is duplicated, and cross-device viewing relies on iCloud Photos syncing that library on its own, not this app's sync.

## Organizing and finding records

- **Search** (`.searchable`) on every list — Weapons, Accessories, both Ammunition sections, Kits — matching brand, model, caliber, serial number, notes, and other relevant text fields, case- and diacritic-insensitive.
- **Sort by any field** — every field of the record (up to 19 for Weapons, the richest model), not just brand. Choosing a field also **groups** the list into sections by that field's value (with a "no value" bucket always last), exactly like the brand grouping did originally; only Notes never groups, since free text rarely repeats. The choice is remembered per list, per device.
- **Tag management** (Settings → Manage values): every autocomplete field across all entities can be **renamed** or **merged** directly from Settings, organized by entity then by field, showing how many existing records use each value. Renaming/merging is applied to the **actual records**, not to a separate suggestion list — so a typo like "Berreta" gets corrected everywhere it appears (list rows, grouping headers, search, exports) in one action, and the wrong suggestion disappears as a natural consequence, with no separate dictionary to keep in sync.
- **Advanced Search** (sidebar/Home, right below Analytics, free — not a Pro feature): combine several conditions in one query — Field · Operator · Value, all required at once (e.g. Brand *contains* "Sig Sauer" **and** Model *contains* "P320" **and** Caliber *contains* ".45") — across any of six record types (Weapons, Accessories, New/Reloaded Ammunition, Shooting Sessions, Kits), on every field of that record, not just a handful. This is the same query engine used to filter the PDF export selection (below), surfaced here as a standalone search with live, tappable results instead of a checklist.

## Data entry tools (macOS)

Three complementary ways to get data in, each suited to a different situation — all Mac-only, consistent with the app's general principle that bulk/file-based data management stays on the "power user" platform while single-record consultation and entry stay universal.

- **Bulk Entry** (toolbar button, one per section): a spreadsheet-style grid for **creating new records**. Tab/Enter navigate cells, paste a block of cells copied from Excel/Numbers directly into the grid (auto-expanding rows/columns as needed), empty rows left untouched are discarded automatically on close.
- **Table view** (segmented control, same pattern as the New/Reloaded ammunition switch): a native, lazy `Table` for **browsing and editing existing records** — every field except photos, in spreadsheet columns, with multi-select and bulk delete against the current search results. Complementary to Bulk Entry, not a replacement for it: Table view has no clipboard-paste import. **On iPad**, the same Table view appears automatically when the device is rotated to landscape — no manual toggle, and never on iPhone. **Shooting Sessions** get their own separate, **read-only** Table (same Mac-toggle/iPad-landscape pattern) — no cell editing, no Add Row, just the session's core columns (Type, Weapon/Kit, Ammunition, Date, Rounds, Distance, Cost); tapping a row opens the full detail screen instead.
- **CSV/TXT import**: a 3-step wizard (choose file & delimiter → map columns to fields, with automatic matching when a header text matches a field label → validate and confirm, skipping only the rows with errors) shared by all four record types. Handles quoted fields, embedded delimiters/newlines, and both UTF-8 and Latin-1 encoded files.

## Export and backup

- **PDF export** (Weapons, Accessories, and Shooting Sessions, both platforms — the Shooter Profile has its own separate export, see below): a **mail-merge** system, not a fixed layout — templates are ordinary `.rtfd` files (designable in TextEdit or Pages) containing `{{token}}` placeholders that get replaced with real record data at export time, plus one placeholder image for the photo. Weapons and Accessories offer three built-in templates (**Simple** — a plain table for a quick inventory; **Insurance/Appraisal** — every field including serial number, medium photo, formal tone; **Photographic** — large photo, minimal text); Shooting Sessions offer only **Simple**, a dedicated table layout with no photo (a session's photos/videos live in the system Photos library, not data the export engine can embed) and a second summary line per row for whatever GPS/weather data that session has. Every entity also accepts a fully custom user-supplied template. Export can be scoped to the whole collection or to a manually/query-filtered selection (a full query builder — field, operator, value, combined with AND — mirrors the same sortable fields used for sorting), with page orientation and per-record page breaks configurable independently of the template chosen. Score is deliberately excluded from every export, same choice as the Power Factor chronograph readings.

  <details>
  <summary>Mail-merge token reference</summary>

  **Weapon**: `marca` `modello` `tipo` `categoria` `statoConservazione` `calibro` `lunghezzaCanna` `numeroColpi` `matricola` `numeroSerieCanna` `annoProduzione` `prezzoAcquisto` `dataAcquisto` `valoreCommerciale` `prezzoVendita` `dataVendita` `vendutoA` `note`

  **Accessory**: `categoria` `sottocategoria` `marca` `modello` `calibro` `quantita` `prezzoAcquisto` `dataAcquisto` `valoreCommerciale` `prezzoVendita` `dataVendita` `vendutoA` `note`

  **Shooting Session**: `tipoSessione` `arma` (Weapon or Kit name) `munizione` `data` `colpiSparati` `distanza` `costo` `note` `indirizzoGPS` `latitudine` `longitudine` `altitudineMetri` `temperaturaCelsius` `umiditaPercento` `velocitaVentoKmh` `direzioneVentoGradi` `pressioneAtmosfericaHPa` `condizioni` (a combined summary joining every GPS/weather field above into one line)

  Tokens are case-sensitive, contain no internal spaces, may repeat anywhere in the template, and an unrecognized token is simply printed literally — useful for spotting a typo. The full, illustrated version of this reference, including a step-by-step guide to building a template from scratch, lives in Chapter 16 of the [PDF manual](#help-and-documentation).
  </details>

- **Shooter Profile export**: a separate, simpler PDF export flow, since a single record has no list to filter or select from — Simple or Photographic template only (no Insurance/Appraisal, no custom templates), with your details and a table of every Membership Card on file.
- **CSV export** (Weapons, Accessories, Shooting Sessions, and Membership Cards, both platforms): one file per entity, with the exact same column headers and order used by CSV import for the entities that support import — a file exported here re-imports without remapping, and doubles as a portable spreadsheet backup. Shooting Sessions' GPS/weather columns are always in their own canonical unit (meters, °C, km/h, hPa) rather than the user's locale, for an unambiguous round trip; Score is not included, same exclusion as PDF export. New/Reloaded Ammunition, Kits, and Maintenance have no CSV export. Decimal separator is always a period and dates are always ISO (`yyyy-MM-dd`), independent of the user's locale.
- **Full backup & restore** (macOS only): a folder package (`manifest.json` + `data.json` + a `Photos/` folder of original JPEGs) covering **every** record type — including Kits, Maintenance, Shooting Sessions, and the Shooter Profile — saved and restored anywhere via the standard system file panel, entirely independent of iCloud. Restore is a full replace of the local/synced database, with an explicit confirmation naming the operation as destructive.

## Analytics

A dashboard (Settings gear → Analytics, universal on both platforms since it's consultation, not bulk data management) built entirely from data already in the collection — no extra fields to fill in.

- **Total value**: current market value of every non-sold Weapon and Accessory, converted to the reference currency; Ammunition shows **net capital invested** instead (purchases + reloads − sales), since ammunition has no "market value" field of its own. Records that can't be converted (missing exchange rate) are called out explicitly rather than silently excluded from the total.
- **Cash flow**: a line chart per family (Weapons/Accessories/Ammunition) of purchases (red) against sales (green), aggregated by month — only months with an actual movement occupy a slot on the axis, so a collection with sparse, irregular activity doesn't produce an unreadable, evenly-spaced calendar axis.
- **Distribution**: a horizontal bar chart of record count by brand or by caliber (switchable), per family, capped at the top 10 with an explicit "+N more" note beyond that.
- **Gain/loss**: for every non-sold piece with both a purchase price and a current market value on file, the exact difference between the two — per piece and aggregated per family, plus a separate **realized** gain/loss view for pieces that have actually been sold (sale price minus purchase price).
- **Shooting Analytics** — a section of its own, separate from the four cards above and built from Shooting Sessions instead of Weapons/Accessories/Ammunition, so session costs never bleed into the collection-value totals: a bar chart of costs per session date, a line chart tracking each session's cost over time, and an average-score card (shown only when at least one session has a Score entry) that stays visible even with monetary values hidden, since it isn't a monetary figure.

## Privacy and security

- **Face ID / Touch ID app lock** (Settings → Security, on by default): `.deviceOwnerAuthentication` — tries biometrics first and automatically falls back to the device passcode/account password, including on Intel Macs without Touch ID, with no branching code needed for the hardware difference. Re-locks immediately whenever the app leaves the foreground.
- **Hide monetary values** (Settings → Privacy): one toggle blurs every price, market value, and monetary figure across record detail screens, Table view columns, and Analytics. Hiding is instant; **revealing requires biometric/passcode authentication** again — the toggle only ever moves in the safe direction without proving identity. Exports intentionally remain unaffected, so a generated PDF/CSV is never silently incomplete. The setting is local to each device, not synced — a phone can stay locked down while the Mac at home shows everything.
- **Everything lives in the user's own private iCloud database.** There is no MF-Lab server anywhere in the architecture — see the full [Privacy Policy](#privacy-policy-and-support) for the complete breakdown (no analytics, no advertising identifier, no third-party trackers, payments processed entirely by Apple).
- **Location and weather are opt-in, per session.** A Shooting Session's GPS and weather fields are each their own off-by-default toggle — nothing is tracked continuously or in the background, and turning one on affects only that single session.

## Appearance and language

- **Light / Dark / System** appearance, chosen independently of the device-wide setting, applied instantly across every window (Settings, main window, lock screen, splash) — a local-only preference, not synced.
- **11 languages** — Italiano, English, Français, Deutsch, Español, 简体中文, Русский, Português (Brasil), العربية, Türkçe, ไทย — selectable **inside the app**, independently of the device's own language, with a one-time restart to apply (the underlying `AppleLanguages` system key has to be rewritten and re-read from a clean process start; there is no reliable way to hot-swap it). Language names in the picker are always shown in the language itself ("Italiano", "Deutsch"), never translated, so the current language stays identifiable even to someone who can't read it.
- **Arabic renders fully right-to-left** with no special-case code: the app uses only semantic `leading`/`trailing` layout guides throughout, never fixed `left`/`right`, so SwiftUI mirrors the entire interface automatically when an RTL language is active.
- User-entered data (brand, model, notes, free-text fields) is **never translated or altered** — localization only ever applies to interface labels, buttons, and system messages.

## Help and documentation

- **In-app guide** (`[?]` button, both platforms, plus the Help menu on Mac): a real manual, one topic per feature — not a marketing tour — with full-text search across titles, summaries, and body text. Available in Italian and English today; other languages fall back to English.
- **Full PDF manual, all 11 languages**: a separate, far more detailed document — one PDF per language, each with a cover, a real table of contents with working page numbers, 21 chapters covering every feature in depth, and a full field-reference appendix for every record type, including the mail-merge token tables above. One tap away from the in-app Help screen, which opens the manual for the **currently active app language** in the system browser. Arabic is properly right-to-left with contextual letter shaping; every language uses a properly embedded font (not a built-in PDF font limited to Western Latin characters), so Cyrillic, CJK, Thai, and Arabic text all render correctly rather than as blank boxes.

## Collection Weapons Pro (subscription)

An optional auto-renewable subscription (monthly or annual, with a 7-day free trial), processed entirely through Apple's StoreKit — MF-Lab never sees payment details, only whether a subscription is currently active.

**Free, forever, no account:**
- Up to **3 records** in each of **seven** categories (Weapons, Accessories, New Ammunition, Reloaded Ammunition, Kits, Maintenance, Shooting Sessions) — existing records beyond that count, from before upgrading or downgrading, are never hidden or locked, only *new* inserts beyond the limit are blocked
- Full record detail on every piece — photos, tags, notes, sale history
- Search, sort, grouping, Advanced Search
- Private iCloud sync, Face ID/Touch ID lock, hide monetary values
- All 11 languages, light/dark appearance

**Pro unlocks:**
- Unlimited records in every category
- CSV/TXT import and export
- PDF export, including fully custom templates
- Analytics — collection value, cash flow, distribution, gain/loss, plus Shooting Analytics
- Bulk Entry and the spreadsheet-style Table view (macOS)
- The **Shooter Profile** section — personal details, membership cards, its own PDF export, Apple Wallet pass generation (Mac)
- **Currency exchange rate settings**, including the optional Alpha Vantage live-refresh (see [Currency handling](#currency-handling))

Analytics, the Shooter Profile section, and Currency settings are **fully locked** behind Pro regardless of record count — not just capped like the other categories above.

Because the app is a single multiplatform product with one App Store listing (not two separate apps), **a subscription purchased on one platform unlocks Pro on every platform** the same Apple ID uses the app on — no separate purchase needed for Mac vs. iPhone/iPad.

## Platform differences at a glance

| | macOS only | iPhone & iPad only | Everywhere |
|---|---|---|---|
| Data entry | Bulk Entry, paste from spreadsheet, CSV/TXT import | Camera capture straight into a record | Manual single-record entry, autocomplete |
| Browsing | Table view always available via a toggle | Table view appears automatically on iPad in landscape | Search, sort, grouping, Advanced Search |
| Shooting Sessions | — | GPS location, live weather, barometric altitude, in-session photo/video capture | Logging a session, score tracking, calendar sync, Shooting Analytics |
| Shooter Profile | Generate an Apple Wallet pass for a membership card | Receive a Wallet pass via AirDrop | Shooter Profile, membership cards, renewal reminders |
| Export | Full collection backup & restore | Dedicated Export tab (no File menu on iOS) | PDF export, CSV export |
| Photos | Each photo opens in its own resizable window | — | Gallery, pinch-zoom, scroll/paste |
| Everything else | Reachable from the File menu | — | Cataloging, Analytics, Face ID/Touch ID, hide values, 11 languages, light/dark, in-app Help, iCloud sync |

## Currency handling

Every price field carries its **own currency**, stored as free text with autocomplete rather than a closed list — a collector may have paid in currencies an enum couldn't represent cleanly, including discontinued ones like pre-Euro Italian Lire ("ITL"). A reference currency and a table of exchange rates (Settings → Currencies, **Pro**) are entered manually by default, and every total across Analytics, Kits, and the collection converts through that table. A record whose currency has no configured rate is called out explicitly as excluded, rather than silently mispriced.

**Optional live refresh (2.0)**: a user can add their own [Alpha Vantage](https://www.alphavantage.co/) API key in Settings → Currencies and tap "Refresh Rates" to pull current exchange rates automatically instead of typing them in by hand. Only the two currency codes involved (e.g. "USD", "EUR") are ever sent to Alpha Vantage — never any collection data — and only when the button is tapped, never automatically or in the background. This is the app's only network call to a service other than Apple's own; leaving the API key field empty means it's never used. See the [Privacy Policy](#privacy-policy-and-support) for the full disclosure.

## Architecture and tech stack

- **SwiftUI**, one codebase shared across macOS, iPadOS, and iOS, with platform-specific layout where the platforms genuinely differ (`NavigationSplitView` sidebar on Mac/iPad, `NavigationStack` on iPhone; native `Form`/`Table`/`Window` on Mac vs. sheets and tabs on iOS).
- **SwiftData + CloudKit** ("Host in CloudKit") for persistence and sync — no custom backend, no authentication system: identity is simply the iCloud account already on the user's devices.
- **Single Xcode multiplatform target** ("Supported Destinations"), not two separate app targets — one bundle identifier (`MF-Corp.CollectionWMacOS`), one App Store product.
- **StoreKit 2**, on-device entitlement checking only, no server-side receipt validation.
- **Shooting Sessions** add Apple's own **Core Location** (GPS), **WeatherKit** (live conditions), **Core Motion** (`CMAltimeter`, barometric altitude, iOS only) and **EventKit** (calendar sync); **Shooter Profile/Membership Cards** add **Contacts** (one-time import) and **PassKit** (Apple Wallet pass generation, macOS only) — all Apple frameworks, no third-party SDKs.
- Zero third-party **code** dependencies: no analytics SDK, no crash reporter, no advertising framework, no third-party networking library — every export/import/report format (CSV, RTF/RTFD mail-merge, PDF pagination) is built on Apple's own Foundation/AppKit/UIKit/CoreGraphics APIs. The one exception is a genuine third-party **network service**, not a library: the optional Alpha Vantage currency-rate lookup described in [Currency handling](#currency-handling), which a user must supply their own API key to activate.

## Privacy policy and support

- **Privacy Policy**: [`CollectionWeapons_Privacy-Policy.html`](./CollectionWeapons_Privacy-Policy.html) — the complete, plain-language breakdown of what stays on-device, what syncs to the user's own iCloud, and what MF-Lab can and cannot see. Updated for 2.0 to cover GPS/weather/altitude, Calendar/Reminders, Contacts import, and the Alpha Vantage currency lookup.
- **Marketing page**: [`CollectionWeapons_Landing.html`](./CollectionWeapons_Landing.html)
- **Support**: https://www.mf-lab.com/support/

---

© 2026 MF-Lab.com — Collection Weapons
