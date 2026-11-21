# CAST Season Volunteer Sign-Up — one form for the whole 2027 season (standalone)

The season volunteer sign-up as a plain web page, so it can live on any
ordinary website instead of inside Google's script service. That is the
fix for the phone error Darve hit Nov 19 with the Premiere form: when
the form is served by Google, a phone signed into two Google accounts
gets the Drive "unable to open the file" error (the /u/ account-index
problem — the plain exec URL returns 200, the /u/0/ and /u/1/ forms
return 404). A normal web page never hits it.

## How it works

- `index.html` — the whole form: same CAST theme, same phone
  validation, same remember-me (email in localStorage, details looked
  up from the contact list), plus a "Return to the CAST Home Page"
  button on the confirmation step. A remembered device first shows
  "You are logged in as Name (email)" with Continue and "Log in as
  someone else" — it no longer jumps straight to the picker.
- The flow, step by step:
  1. Your details (name, email, mobile number).
  2. A dropdown — "Which production would you like to volunteer
     for?" — populated from the backend. A production that has closed
     is no longer listed; a fully-booked one is shown disabled with
     "— Full". Then pick ONE performance night (each shows how many
     spots are still open in total; a full night is shown disabled),
     then ONE role for that night (only roles with open spots are
     shown, with the spots left; a role already in your list for that
     production and night is shown disabled). "Add this signup" puts
     the pick in your "Your signups" list (each entry has a Remove
     link). "Yes, add another show" loops back to the production
     dropdown — productions already picked stay available for another
     night. "Not right now — submit" sends all collected signups in
     one call.
  3. Confirmation listing every signup that went through. If some
     signups in a batch failed (a spot just filled, or a duplicate),
     exactly which ones failed is shown with the reason, the ones that
     went through stay confirmed and are never submitted again, and
     the failed ones stay in the list so the volunteer can re-pick or
     remove them.
- It talks to the Apps Script backend over a JSON API using plain
  `fetch()` with no credentials, so a phone signed into several Google
  accounts cannot be bounced to a `/u/` address that 404s (the backend
  answers with `Access-Control-Allow-Origin: *`). JSONP is kept as a
  fallback for browsers without fetch. Actions:
  `?action=getAvailability`,
  `?action=lookupContact&email=...`,
  `?action=submitSignups&name=...&email=...&phone=...&
  signups=<JSON array of {productionId, night, role}>` (JSONP adds
  `&callback=`).
- Backend: `~/workspace/cast-season-volunteer-form/Code.gs` (doGet
  serves the API when `?action=` is present, the short Index.html
  page otherwise). Signups land in each production's signups sheet in
  the exact A–M layout the Counts formulas expect (role only in its
  night's column). Darve is emailed when a role fills for a night.
- The API base URL is the `API_URL` constant at the top of the script
  in `index.html`. It is the live "CAST Season Volunteer Form" web app
  exec URL (deployed Nov 21, 2026) — change that one line only if the
  backend is ever redeployed to a new URL.
- Remember-me uses its own localStorage key,
  `castSeasonVolunteerEmail`, so it never collides with the earlier
  forms' remembered emails on the same device.
- The page contains NO 2027 show names anywhere — not in its text,
  comments, or placeholders. Production labels come only from the
  backend, which shows month placeholders until the show-names flag
  is flipped after the Dec 1, 2026 Premiere.

## What the public sees — manager roles are not on this form

Only the five non-manager roles are ever offered: Concession Stand,
Usher, Helper (Wherever Needed), Ticket Scanner, Door Greeter. The three board-only manager roles
(Concession Manager, Ticket Manager, House Manager) are deliberately
absent from the public form — the board fills those spots itself.

## Privacy note

Remember-me has no password: anyone who knows a volunteer's email can
pull up that person's name and mobile through the lookup. Darve was
told this plainly before the Premiere form went live (Nov 19, 2026)
and chose remember-me anyway; the same trade-off applies here.

## Deploy

1. Deploy the backend first (see `~/workspace/cast-season-volunteer-
   form/README.md`) and put its exec URL in `API_URL` in `index.html`.
   (Done Nov 21, 2026 — the live exec URL is already in the file.)
2. Verify the API: `curl "<exec-url>?action=getAvailability&
   callback=test"` should return
   `test({"ok":true,"productions":[...]});`. (Verified Nov 21, 2026:
   January Production, 8 nights, 9 open spots per night.)
3. Put `index.html` on a static host (GitHub Pages, like the Premiere
   form at volunteer.baycitycast.com) and embed it on
   baycitycast.com/volunteer and baycitycast.com/be-a-volunteer. No
   server code, no build step, no cost. No new dependency. The chosen
   address is season-volunteer.baycitycast.com (approved by Darve Nov
   21, 2026; he changes the Squarespace CNAME himself).
4. Verify on a phone signed into two Google accounts: the page loads,
   the production dropdown shows the month placeholder, nights show
   with live counts, a signup lands in the production's signups sheet
   in the exact A–M layout and Counts drops by one for that night +
   role. Then turn off the old single-show Google Form
   limiter's triggers and close the old Google Form.

## Status

- Built Nov 21, 2026. Backend deployed and verified live Nov 21
  (January Production, 8 nights, 9 open spots per night). Frontend
  API_URL set to the live exec URL. Hosting in progress.
  Waiting on Darve's go-ahead to deploy and host.
