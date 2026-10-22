# EZQ update v15: named staff, audit trail, CSV, atomic tickets

Folder on your computer: `D:\EZQ\dev`
Live site: https://panelco-i-queue-system.web.app

## What changed in this version

| Area | Change |
|---|---|
| Passwords | Staff change their own password from the employee page. |
| Users | `window.html` (company admin page) now creates staff and kiosk accounts, each with a **real name**. No more shared branch logins. |
| Audit trail | Every call, no-show, reset and user change records **who** did it. Tickets carry `calledByName` / `closedByName`. Admin page has a recent-activity list. |
| Announcements | Always **beep first** (even when voice is off), then voice: "Membership, Window 2, now serving number 7". |
| Reports | From/To dates, "By day" and "By employee" tables, average service time, **Export CSV** (opens in Excel). |
| Tickets | Number allocation is atomic with retry, so two kiosks cannot issue the same number. **Reprint** button on the kiosk popup. |
| Windows | Renaming a window label keeps its queue. Warning before deleting the last window of a service that has people waiting. New windows never overwrite old ones. |
| Time zone | Per company (system-admin -> Edit company -> Time zone). Default Asia/Manila. |
| Backup | system-admin -> **DOWNLOAD BACKUP** saves all companies as a JSON file. |
| Offline | No offline mode: with no connection the buttons are disabled (queue suspended). |
| Display screen | **Layout 2: video + news.** Employees switch layouts live on their own page (Display screen: Classic / Video + news). Same `display.html` address, no reload. See "Display screen layout 2". |
| Kiosk | **Full screen.** A "Tap to start" screen switches the kiosk to full screen (no address bar); also works as an installed Home Screen app. See "Kiosk full screen". |
| Logos | Company logos in the display and kiosk header, chosen by company (see "Company logos"). The display header now shows just the company name (e.g. PANELCO I, ACWD). |
| Removed | `staff2.html`, `index2.html`, the `/queue2` address and the old legacy rules. |

## Step 1. Publish the new rules (required)

**Rules changed in v11:** company admins can now write an office's `settings` (needed for the display video/news form). Publish the new `database.rules.json` again.

Open `database.rules.json`, copy all, paste into **Realtime Database -> Rules**, **Publish**.
Back up your current rules first (copy them to a text file).
These rules were written but could not be tested against a real Firebase from here. If anything fails, paste your backup back and tell me the exact error.

## Step 2. Deploy

```
cd /d D:\EZQ\dev
del staff2.html index2.html
git rm --cached staff2.html index2.html
firebase hosting:channel:deploy test --expires 7d      (private preview, optional)
firebase deploy --only hosting                         (live)
```
Copy this package over `D:\EZQ\dev` first (replace files). Delete `staff2.html` and `index2.html` yourself, as they are not overwritten by copying.

## Step 3. Create people (company admin)

1. Log in as `admin@<company>` (e.g. `admin@panelcoi.com`) -> `window.html`.
2. In **Staff**: enter **full name**, **login name** (the part before the @; the company domain is added), **temporary password** (6+ characters), **role** and **branch**, then **CREATE**.
3. Roles: `employee` (calls customers), `kiosk` (opens only the kiosk page), `userAdmin` (company admin).
4. Per person: **SAVE** (name/role/branch) and **DEACTIVATE / ACTIVATE**.
5. Login names equal to a branch ID (like `mainoffice`) are refused (reserved for the older branch logins).
6. Accounts are **deactivated, not deleted** (keeps the audit trail). A deactivated person is refused at login and loses database access.

**Passwords:** the email domains are only company references, so reset e-mails cannot be delivered and there is no reset button. Give each person a temporary password; they change it themselves with **Change password** at the top of the employee page (needs the current password). If someone **forgets** their password: deactivate the account and create a new login for the same person (e.g. `maria.santos2`) with a new temporary password. The name stays the same, so the audit trail still reads correctly. A true admin-side password reset needs a Cloud Function (Blaze plan); ask if you want it.

The admin stays signed in while creating accounts (a second, temporary login session is used).

**Kiosk:** create one `kiosk` account per branch (e.g. "Lobby kiosk"). Log in with it on the kiosk computer; it goes straight to `customer.html`. **To sign out of a kiosk:** press and hold the company name at the top-left for 3 seconds, or press Ctrl+Shift+L, then confirm. Customers see no sign-out button. If a browser is stuck on the kiosk, open `https://panelco-i-queue-system.web.app/index.html?logout` to sign it out. Employee accounts opening `customer.html` still work.

**Old branch logins** (`mainoffice@panelcoi.com`) still work as before, so nothing breaks while you migrate. Names cannot be attached to them, so switch people to named accounts and then deactivate/remove the shared ones in Firebase Authentication.

## Who goes where (login)

| Login | Opens |
|---|---|
| `operator@panelcoi.com` | `system-admin.html` |
| `admin@<company>` or role `userAdmin` | `window.html` |
| role `kiosk` | `customer.html` |
| role `employee` (and old branch logins) | `employee.html` |

**Note:** `admin@panelcoi.com` is no longer a global super-user for data; it administers PANELCO I. Only `operator@panelcoi.com` works across companies.

## How the data is stored

```
companies/<company>/name, tagline, timezone
companies/<company>/users/<uid>      name, email, role, officeId, active, createdAt, createdBy
companies/<company>/auditLog/...     who did what (user changes, resets)
companies/<company>/offices/<branch>/
    windows/<id>                 label, order, serviceKey, ticketId, calledAt, queue
    counters/<date>/<service>    last number issued
    tickets/<date>/<id>          service, number, status, createdAt, calledBy(Name), closedBy(Name) ...
    settings/voiceEnabled
access/<company>/<branch>/<uid>      role (written together with the user record; used by the rules)
```

* A **service** = a `serviceKey`. Windows sharing one share a queue and one kiosk button. Renaming a label does not change the key, so waiting customers stay in the queue. Typing the label of an existing service joins that queue.
* Numbers restart each day in the company time zone.
* Ticket status: `waiting` -> `serving` -> `done` / `noshow`.

## Daily use (employees)

1. Log in with your own account, tap your **window**. Your name shows at the top.
2. **CALL NEXT** calls the longest-waiting customer for that window's service; the previous customer is marked done under your name. The display beeps, then announces the service, window and number.
3. **RECALL** repeats the announcement. **NO SHOW** clears the window.
4. **Daily report**: pick From/To (up to 62 days). Shows totals, by service, window, employee, day (for ranges) and hour. **Print** or **Export CSV** (one line per ticket, includes who called/finished it).
5. **Reset today**: emergency button; logged in the audit trail.


## Display screen layout 2 (video + news)

Same page as before: `display.html?company=panelcoi&office=mainoffice`. Employees choose the layout with **Display screen: Classic / Video + news** on their page; the TV changes by itself (no reload, sound stays on). Add `&v=2` to the address to preview layout 2 without changing the setting.

Layout 2: header and clock on top; **video (16:9) top-left**; **news top-right**; **windows along the bottom** (they shrink to fit; everything from the classic screen is still there: beep, voice, glow, clock, connection, tagline, sound and full-screen buttons).

Set it up in `window.html` -> **DISPLAY SCREEN** (pick the office, fill in, **SAVE DISPLAY SETTINGS**):
* **Video links**, one per line: `.mp4` / `.webm` files (played in a loop, one after another), or YouTube links. Videos play **muted** (the queue announcements need to be heard). The file must be reachable from the internet (not a file on your PC); `.m3u8` streams are not supported.
* **News API address**: any `https://` address that returns a list as JSON. It understands a plain list, or a list inside `articles`, `items`, `news`, `data`, `results`, `entries`, `posts` or `stories`. For each story it uses `title`/`headline`, `description`/`summary`/`content`/`body`, `source`/`author`, `publishedAt`/`pubDate`/`date`. Strings alone also work. HTML is removed from the text. **The API must allow browsers to call it (CORS);** if it does not, the news stays empty and the panel says "Offline". Tell me the API and I can match its format exactly.
* **Reload news every (minutes)**: default 10. If the API fails, the last news stays on screen.
* **Own announcements**: one per line, shown before the news (works without any API).
* News display is automatic: **4 stories or fewer** are shown one page at a time (12 seconds each, with dots); **more than 4** scroll upward like a ticker. The numbers are at the top of the script in `display.html` (`NEWS_PAGE_MAX`, `NEWS_PAGE_SECONDS`).

**If the video/news row does not appear on a TV:** (1) check the layout really is **Video + news** (employee page toggle, or `window.html` Layout), (2) make the TV load the newest page: close the browser tab completely and reopen it, or clear its cache, because TVs keep old copies of pages, (3) try the same address with `&v=2` added. The video/news row is sized by script (not by newer CSS features), so older smart-TV browsers can show it; very old TV browsers may still not support YouTube embedding.

## Company logos

Put two pictures per company in the **same folder as the pages** (`D:\EZQ\dev`), named after the company ID:
* `panelcoi_logo.png` and `panelcoi_iso.png`
* `acwd_logo.png` and `acwd_iso.png`

They appear to the left of the company name in the header of the display (both layouts) and the kiosk, in the order logo, iso, name. The company ID is the one in the display address (`?company=panelcoi`) or, on the kiosk, the login's company. A missing file is simply not shown. They are deployed with the site (`firebase deploy --only hosting`); no database setting is involved. Use transparent PNGs, roughly 3:1 wide for the logo and square for the iso; they are shown about 70 px tall on a 1080p screen.
The display header shows the company's name from system-admin (e.g. `PANELCO I`), without "QUEUE MANAGEMENT SYSTEM".

## Kiosk full screen

* Opening `customer.html` shows **Tap to start**; one tap switches the page to full screen (no address bar). If someone leaves full screen, the start screen covers the kiosk again until it is tapped.
* **Best for tablets:** open the kiosk in Chrome (Android) or Safari (iPad), choose **Add to Home Screen**, and open it from the new icon. It then runs as an app without any browser bar and the start screen does not appear.
* iPhone Safari has no full-screen support, so use Add to Home Screen there.
* A web page cannot block the tablet's own gestures (swiping home, notifications). To lock the tablet itself: **Android "screen pinning"** (Settings > Security) or **iPad Guided Access** (Settings > Accessibility). Staff sign out with the hidden hold on the company name.
* On a PC, Chrome `--kiosk` still works as before.

## Kiosk

Same setup as before (default printer, Chrome `--kiosk --kiosk-printing`, shortcut in `shell:startup`), logged in as the branch's kiosk account. The ticket popup has **Reprint** for paper jams. Ticket layout is for an 80 mm roll; for 58 mm edit `@page { size: 80mm 110mm; }` and `width: 72mm;` in `customer.html`. I could not test a physical print, so do one test print.

Display screens: `display.html?company=panelcoi&office=mainoffice`

## Adding another company (example: ACWD)

1. system-admin -> **Add company** "ACWD" (ID `acwd`, set Time zone and Tagline) -> **Add branch** "Main Office".
2. Firebase console -> Authentication -> Add user `admin@acwd.com` (the first admin; after that they create everyone else in window.html).
3. Log in as `admin@acwd.com`, add windows, then create staff and a kiosk account.

The company ID must match the email domain's first part (`acwd` for `@acwd.com`).

## Company tagline

system-admin -> Edit company -> **Tagline**. Shown in the display footer after the company name. Set PANELCO I to `Making Lives Better` once.

## Known limits

* Rules are untested on real Firebase (see Step 1).
* The all-branches company report was intentionally skipped.
* There is no admin-side password reset (see Passwords above).
* Firebase Authentication accounts are never deleted by the app, only deactivated.


## Kiosk Android app (new)
See kiosk-app/BUILD-APP.md. The app loads the live site and prints silently to a paired Bluetooth printer; in a normal browser customer.html still uses the print dialog.
