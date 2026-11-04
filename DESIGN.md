# tugas-task: Design guidelines

Owner of UI/UX decisions: the "Redesign task board UI/UX" thread. Any agent that changes how the app looks or behaves on screen must follow this file, and must ask that thread before adding a new pattern that is not covered here.

Reference: Google Tasks board view (Material 3 feel). Clean, calm, lots of whitespace inside cards, little chrome.

## 1. Themes

- **Light is the default.** Dark is opt-in through the moon/sun button in the top bar. The choice is saved in `localStorage["taskboard.ui.v1"].theme` (`"light"` | `"dark"`), applied before first paint by a tiny inline script in `<head>`.
- All colours are CSS custom properties on `:root` (light) and `html[data-theme="dark"]` (dark). **Never hard-code a colour in a rule**; add or reuse a token so both themes stay in sync.

| Token | Light | Dark | Use |
|---|---|---|---|
| `--bg` | #f8fafd | #131314 | page background, top bar |
| `--surface` | #ffffff | #1e1f20 | list cards, menus, dialogs |
| `--surface-2` / `-3` | #f0f4f9 / #e4e9f1 | #282a2c / #333537 | search field, tonal buttons, badges |
| `--line` / `--line-2` | #e1e5ea / #c4c7c5 | #2f3133 / #444746 | card borders / input borders |
| `--text` / `--text-2` / `--muted` | #1f1f1f / #444746 / #747775 | #e3e3e3 / #c4c7c5 / #8e918f | primary, secondary, tertiary text |
| `--accent` (+ `-ink`, `-soft`) | #0b57d0 | #a8c7fa | primary actions, active nav, focus ring |
| `--danger` (+ `-soft`) | #b3261e | #f2b8b5 | overdue, destructive actions |
| `--ok` (+ `-soft`) | #146c2e | #6dd58c | due tomorrow / within 7 days |
| `--hover`, `--hover-2` | 5% / 8% ink overlay | same idea | hover states |
| `--c0`…`--c7` | 8 list colours | lighter variants | list dot, sidebar checkbox |

List colour = `--c{index % 8}` by list position. Do not store colours in the data.

## 2. Type, spacing, shape

- Font: **Plus Jakarta Sans** (Google Fonts) 400/500/600/700, fallback `system-ui`. Base 14px / 1.45.
- Sizes: app title 19/600, list title 16/600, dialog title 22/600, task title 14/500, subtask 13.5/400, chips & section labels 12/600 (section labels uppercase, letter-spacing .06em).
- Spacing on a 4px grid. Board padding 20px 24px (12px on mobile), gap between cards 16px (12px on mobile).
- Radii: 8 (inputs, menus), 12 (task rows), 16 (cards, create button), 28 (dialogs), pill (buttons, chips, search, nav items).
- Elevation: cards are flat with a 1px border; only the create button, menus, dialogs, toast and drag ghost get shadows.

## 3. Icons

- **SVG only**, from the inline sprite at the top of `<body>` (`<svg class="i"><use href="#i-name"/></svg>`, helper `icon(name, size)` in JS). Stroke style, 24px viewBox, 1.9 stroke width, round caps. Sizes: `.i` 20px, `.i.sm` 16px, `.i.xs` 14px.
- **No emoji or Unicode symbols as icons** (no ✎ ⬇ 🗑 📋 ☰ ⋮⋮). They render inconsistently across Windows/Android; that was the main bug in v1. One emoji is allowed in copy ("Semua tugas selesai 🎉").
- To add an icon: add a `<symbol id="i-…">` in the same style.

## 4. Layout

- Top bar (64px): menu toggle, logo + "tugas-task", search pill (centre, max 620px, `/` focuses it), theme toggle, "Tempel jadwal" (tonal), "Ekspor .ics" (primary). Only **one** primary (filled) button on screen.
- Sidebar (280px, collapsible): "Buat tugas" button → time filters (Semua, Terlambat, Hari ini, 7 hari, Tanpa tanggal, each with a live count; overdue count in danger colour) → "Daftar" list toggles with coloured checkboxes and counts, "Tampilkan semua", "Buat daftar baru" → "Tampilkan selesai" switch → backup (JSON export/import).
- Overview (above the board, collapsible via the chevron, pref `overview`): **month calendar** (Monday-first, today = filled accent circle, dots in list colours for open tasks, overdue day numbers in danger; click a day = date filter shown as a removable chip; drop a task on a day = set its date) and **Aktivitas heatmap** (GitHub-style, ~26 weeks, cell size adapts 11–18px, levels 0/1–2/3–5/6–9/10+, stats: aktivitas, hari aktif, hari beruntun). Side by side ≥1180px, stacked below.
- Board: **CSS grid that wraps** (`repeat(auto-fill, minmax(300px, 1fr))`). Never horizontal scrolling, never cut-off columns. Empty board shows an empty state with a CTA.
- ≤900px: sidebar becomes an overlay drawer with scrim, board becomes one column, button labels collapse to icons. ≤480px: "Tempel jadwal" moves into the drawer.

## 5. Components

- **List card**: colour dot, name (double-click = rename), open-task count badge, ⋮ menu (Ganti nama, Ekspor .ics, Sembunyikan, Hapus). Then "+ Tambahkan tugas" inline input (Enter adds, Esc clears). Completed tasks collapse under "Selesai (n)".
- **Task row**: grip (appears on hover; always visible on touch), round check, editable title, meta row of chips. Hover actions on the right: date button (only if no date yet) and ⋮ menu (Tambah subtugas, Atur/Ubah/Hapus tanggal, Google Calendar, Unduh .ics, Hapus tugas). Keep the row quiet: no more than two visible action icons.
- **Due chip**: overdue = danger-soft ("2 hari lalu", "Kemarin"); today = accent-soft ("Hari ini"); tomorrow and ≤7 days = ok-soft; later = outlined neutral ("Sen, 16 Nov"). Clicking the chip opens the native date picker.
- **Task details dialog** (GCal-style, from ⋮ → Detail or clicking the due chip): title, date + time, location, reminder, list, notes. Location chip links to Google Maps; time shows in the due chip ("Hari ini, 14:30") with a bell when a reminder is set.
- **Share dialog** (top bar "Bagikan"): email + role (Bisa edit / Hanya lihat) + Undang, then "Orang yang punya akses" list with avatar, role, remove. Account avatar at far right of the top bar; member avatars stack left of Bagikan when 2+ members.
- **Subtasks**: indented 32px under the title, small check, same grip pattern. A "x/y" chip shows progress when there are 2+ subtasks.
- **Dialogs**: native `<dialog>` with the shared `.dlg` layout. **Never use `prompt()`, `confirm()` or `alert()`**; use `ask({...})`. Destructive confirmations use `.btn.danger`.
- **Menus**: `openMenu(anchor, items)` popover; Esc or outside click closes.
- **Toast**: short confirmation after every non-obvious action (export, delete, paste import).
- Buttons: `.btn.primary` (filled accent), `.btn.tonal`, `.btn.text`, `.btn.danger`, `.iconbtn` (40/32/28px round).

## 6. Interaction rules

- Drag and drop uses **pointer events** (mouse, pen, touch). Drag only from the grip. Drop zones: top of a task = before, middle = make subtask, bottom (or below its subtasks) = after, empty space of a card = end of that list. Show a 3px accent line for before/after, accent-soft fill for "inside", accent outline for card end. A floating ghost follows the pointer and the content auto-scrolls near edges. Esc cancels.
- Inline editing: Enter or Esc commits/blurs; empty title deletes the item; paste is plain text only.
- Every interactive element is keyboard reachable with a visible `:focus-visible` ring (accent). Touch targets ≥ 28px, primary ones 40px.
- Respect `prefers-reduced-motion`. Motion: 120–250ms, easing `cubic-bezier(.2,0,0,1)`.
- Copy is Bahasa Indonesia, short, sentence case.

## 7. Activity & notifications

- Activity = `state.activity` (`{"YYYY-MM-DD": count}`), incremented by `bump()` on: add task, complete task/subtask, add subtask, paste import (n items), save details. Not on un-complete.
- Notifications are local (Notification API, fallback toast) and only fire while a tab is open: timed tasks at time − `remind` minutes, all-day tasks at 08:00 on the due day; skipped if >12h late. Sent keys stored in `taskboard.notified.v1`. Real push when the app is closed needs a service worker + backend (future).

## 8. Backend adapter contract (for the Supabase thread)

`index.html` owns all UI. The backend script only defines `window.tugasBackend` (before or after load) and calls the app hooks. Do not edit UI markup or CSS; ask the UI thread.

```js
window.tugasBackend = {
  user(),                 // sync → {email, name?, avatar?} | null
  signIn(), signOut(),    // may redirect (Google OAuth)
  members(),              // async → [{email, name?, avatar?, role:'owner'|'editor'|'viewer', status:'active'|'pending'}]
  invite(email, role),    // async; role 'editor' | 'viewer'; throw Error(message) to show it in a toast
  removeMember(email),    // async
  save(state),            // called after every local save; debounce/upsert on your side
};
window.tugasApp.setState(remoteState); // when remote data arrives (also writes localStorage)
window.tugasApp.refreshAuth();         // after login/logout or membership change
window.tugasApp.toast(msg);
```

## 9. Do not break

`localStorage["taskboard.v1"]` data shape (`{lists:[{id,name,visible,tasks:[{id,title,done,due,time?,location?,notes?,remind?,subs:[{id,title,done}]}]}], activity?:{}}`; new fields are optional and additive), `parseSchedule()` (Indonesian weekday headers, "- " items, indented subtasks), `icsFor()` / `gcalUrl()` export, JSON backup import/export, filters and search. UI preferences (`theme`, `dueF`, `showDone`, `overview`, `notif`) live separately in `taskboard.ui.v1`.
