# Campus Stores — Findings & Draft Notes

Working notes for the Product Engineer work sample. No application code has been
modified; this file is only a record of analysis and draft write-up.

## Status

- Fork: `ashishkumar006/campus-stores` (created and cloned to this folder).
- `origin` = fork, `upstream` = `virinchi-healthcare/campus-stores`.
- Working tree is clean apart from this notes file.
- Local server used for testing: `http://localhost:8000`.

## Time map (estimates)

| # | Task | Estimate | Status |
|---|---|---|---|
| 1 | Fork + clone the repo | ~2 min | done |
| 2 | Read all source files + brief | ~15 min | done |
| 3 | Automated playthrough of all 3 phases | ~10 min | done |
| 4 | Audit inconsistencies (data, UX, code) | ~25 min | done |
| 5 | Draft paragraphs 1 & 2 with evidence | ~20 min | done |
| 6 | Note findings in separate file | ~5 min | done |
| 7 | Sketch/prototype the single change (para 3a) | ~30-45 min | not started |
| 8 | Walkthrough + what I'd keep (para 3b) | ~15-20 min | not started |
| 9 | Real-counter validation observation (para 3c) | ~10 min | not started |
| 10 | Assemble README write-up | ~20 min | not started |
| 11 | Commit, push, copy commit URL, submit form | ~5-10 min | not started |

Remaining: roughly 1.5-2 hours.

## What the task actually is

The three README steps are the submission wrapper. The real work is the
**Candidate brief** disclosure at the top of the app:

- Work the shift: get the Film Society kit out, handle the return, leave the
  next shift enough information to finish the job. Use the time control.
- Show one moment the product handles well and one moment you would change,
  each backed by a specific action or decision from the shift.
- Sketch or prototype the change; walk through the same job using it; explain
  what you would keep; name one observation at a real counter that would tell
  you whether it helped.
- Any tools allowed, including AI and design/coding tools.

Submission: put the answer in the fork's README, push, copy the final commit
URL, submit the Google Form with roll number, email, and commit link. Roll
number and email go in the form only.

## Concept of the webpage

A simulation of a campus media-equipment counter, packaged as a hiring work
sample. In-world it is a paperless desk logbook (approved requests, inventory,
loan register, desk book). In reality it exists to give every candidate the
same bounded, deliberately ambiguous scenario so product judgment can be
assessed. The app is intentionally imperfect; the Saturday adapter return is
the constructed decision point.

## Automated playthrough — raw evidence

### Friday 3:45 pm (pickup)

- RQ-214: Courtyard screening, Film Society, filed by Isha Rao, return by
  Fri 8 pm. Approved: HDMI projector, Portable speaker, USB-C -> HDMI adapter.
  Note: "Playback is from a USB-C-only laptop. The host will collect at 4 pm."
- Inventory: PR-06 "In repair" (Workshop), MI-09 "On loan", others "On shelf".
- Issue-form unit dropdown: PR-04, SP-02, SP-05, AD-12, AD-15, CA-03
  (PR-06 and MI-09 correctly absent).
- Collected-by dropdown: Isha Rao, Kabir Sen, Leela Das, Tara Shah, Dev Menon.
- Return-by dropdown: Fri 8 pm, Sat 9 am, Sat 10 am, Sat noon.
- Test: injecting CA-03 against RQ-214 was rejected with "This equipment type
  is outside the approved request. Choose an approved unit."
- Issued PR-04, SP-02, AD-12 to Kabir Sen, due Fri 8 pm:
  L-402, L-403, L-404.
- Three separate form fills were required; no bulk issue or completeness view.

### Friday 7:30 pm (partial return)

- Desk book gained: "I brought the projector and speaker. Leela Das has our
  adapter. She'll be at the counter at 9 am tomorrow."
- Closed L-402 (PR-04) and L-403 (SP-02).
- Handover note written: named unit, slip, holder, promised time, and asked the
  next shift to check the physical adapter tag.

### Saturday 9:00 am (handover)

- Open slips: L-404 (AD-12, due Fri 8 pm) and L-401 (MI-09, due Sat 10 am).
- Shift text: "Work out what was outstanding from the screening and close the
  loan for the right item."
- Only actions available: advance, tab, new, scope, return, restart.
- No overdue marker: searching the register HTML for overdue/late = false.
- Closing L-404 produced: "AD-12 returned to stock. Slip L-404 closed."
- Final inventory: AD-12 On shelf, AD-15 On shelf.

### Deeper guard and edge-case tests

- **Quantity guard is real.** With AD-12 out on L-402 for RQ-214, I tried to
  issue the second physical adapter AD-15 against the same request. It was
  rejected: *"The approved quantity of this equipment type is already on
  loan."* Only one adapter slip existed afterwards. This holds even though
  AD-15 was physically on the shelf and offered in the dropdown.
- **Availability is restored on return.** Before returning AD-12, the dropdown
  was: PR-04, SP-02, SP-05, AD-15, CA-03 (no AD-12). After "Record return" on
  L-402, AD-12 reappeared: PR-04, SP-02, SP-05, AD-12, AD-15, CA-03.
- **No phase gating on returns.** At Friday 3:45 pm, with the clock still at
  pickup time, I clicked "Record return" on L-401 (MI-09, due Sat 10 am). It
  closed immediately: *"MI-09 returned to stock. Slip L-401 closed."* The app
  does not check the desk clock, the due date, or the desk-book message saying
  Tara keeps the microphone overnight.
- **Alternate physical return simulation.** I restarted and issued AD-15 (the
  Anker) instead of AD-12, then repeated the same shift. At Saturday 9 am the
  prompt was identical ("close the loan for the right item") and the register
  showed L-404 - AD-15, Due Fri 8 pm. Clicking "Record return" produced
  *"AD-15 returned to stock. Slip L-404 closed."* Final inventory showed both
  AD-12 (Belkin) and AD-15 (Anker) "On shelf". The product therefore closes
  whichever slip was created at issue; it has no way to establish which
  physical adapter was actually handed over at the counter.
- **Minor:** a failed validation leaves the issue form open with the header
  button changed from "New issue slip" to "Cancel", and the error is only in
  the status line, not on the field.

## Edge-case sweep (systematic browser testing)

### Issue form

- Empty submit: silent no-op, status unchanged (no field-level error).
- Injected repair unit PR-06: silent no-op, no message.
- Injected already-on-loan unit MI-09: silent no-op, no message.
- Invalid person ("Nobody"): silent no-op, no message.
- Invalid request ("RQ-999"): silent no-op, no message.
- Only two guards explain themselves: unapproved type ("outside the approved
  request") and duplicate open type for the same request ("approved quantity
  already on loan"). All other rejections are invisible.
- Due date "Sat noon" was accepted for RQ-214 whose request due is "Fri 8 pm",
  and recorded verbatim on the slip. No consistency check with the request.
- "Leela Das" was accepted as collector for RQ-214 even though the desk book
  authorises Kabir Sen. No authorisation check.
- A failed validation leaves the form open and the header button changes from
  "New issue slip" to "Cancel".
- Advancing time while the form is open leaves the form open; the phase changes
  underneath it.
- Switching tabs while editing resets `editing` and silently discards the
  unsaved form.

### Returns

- Wrong-phase return allowed: at Friday 3:45 pm, L-401 (MI-09, due Sat 10 am,
  desk book says it stays overnight) closed with one click.
- Returned slips show a "Returned" badge and no button; they cannot be reopened.
- With two restored slips sharing number 401, clicking "Record return" on the
  **second** row closed the **first** loan (`find()` matches by number only).
  Status: "MI-09 returned to stock. Slip L-401 closed."

### Handover notes

- A note written at Friday 3:45 pm appears **above** the Friday 7:30 pm
  desk-book entry after advancing - the reverse-chronological list sorts by
  insertion order, not the note's own phase time.
- Whitespace-only note: silently ignored, no status change.
- 500-character note submitted programmatically was stored in full despite
  `maxlength="400"`; no JS-side validation.
- HTML in a note is correctly escaped (`&lt;img ...&gt;`), no XSS.
- Restart clears all notes.

### Time / phase

- Can jump from 3:45 pm straight to Saturday 9 am with no work done; the final
  prompt then has no adapter slip to close.
- Advance button is hidden on the last phase.
- After the screening is over, new loans can still be created against RQ-214
  and against unrelated requests (e.g. RQ-217 camera) at Saturday 9 am.

### Saved-state restore

- Invalid `filter` ("Bogus") accepted; inventory renders zero rows silently.
- Loan missing `back` accepted; treated as outstanding and returnable.
- Invalid `who` / `due` strings accepted and displayed verbatim.
- Duplicate slip numbers accepted; return closes the wrong loan (see above).
- `version: 2` rejected; UI resets to the original shift but the invalid
  payload remains in localStorage.
- Empty `loans` array rejected; the starting loan cannot be removed via storage.

### Additional cases closed after a coverage review

- Valid inventory filters: "On loan" showed only MI-09, "In repair" only
  PR-06, "On shelf" 6 units. Filter choice persists across tab switches and
  reload because it is stored in the shift state.
- A selected request stays selected when switching tabs and returning to
  Requests (detail view reappears).
- The advance button is hidden on the last phase (`hidden === true`).
- "All slips" scope shows returned slips with a "Returned" badge and no return
  button; only outstanding slips have a "Record return" button.
- Browser-native `required` validation blocks a real click on Save with empty
  selects: the form stays open and no status message is produced.
- A second submit of the same unit (or re-issuing an already-out unit) is
  silently ignored; no duplicate slip is created.
- Corrupted JSON in localStorage makes the app fall back to a fresh shift,
  but the invalid payload remains in storage.
- An empty `loans` array in storage is rejected by `restore()`; the UI shows
  the original shift while the stale payload remains in storage.
- With `setItem` forced to throw, the app keeps working in memory and appends
  "Browser storage is unavailable. Progress lasts for this visit." to status.
- A note written at Saturday 9 am is labelled "Saturday · 9:00 am" and appears
  first in the desk book.
- Slip numbers are `max(existing)+1`: with a stored slip numbered 500, the next
  issue created L-501.

### Not tested (acknowledged gaps)

- Mobile/touch layout and the CSS container-query breakpoints - code-read only.
- Keyboard-only navigation, focus order, and screen-reader behaviour.
- Storage being genuinely unavailable at page load (I only simulated a
  `setItem` failure after the app had loaded).
- Multiple browser tabs writing to the same localStorage concurrently.
- Visual appearance/screenshots: the screenshot tool failed because the
  desktop window was not visible, so all checks were DOM/text based.
- Cross-browser native validation messages (only the in-app Chromium was used).
- Network/offline behaviour: the app makes no network requests.

### Implications for the two moments

- The guards themselves are correct, but feedback is inconsistent: only type
  and quantity violations explain themselves; every other rejection is silent.
- The Saturday "right item" problem is bigger than physical verification:
  record identity is also unenforced (duplicate slip numbers close the wrong
  loan), and the due date is never validated against the request or the clock.
- Because time and phase are unconstrained, the tool cannot enforce the
  handover at all - it relies entirely on the operator's memory and note text.

## Audit — inconsistencies found

### Data / narrative

1. RQ-214 due Fri 8 pm (`app.js:11`) vs adapter promised Sat 9 am
   (`app.js:77`) — 13-hour gap, never treated as late.
2. "Host will collect at 4 pm" (`app.js:13`) vs Kabir collecting at 3:45
   (`app.js:69-71`) and Isha authorising Kabir in the desk book
   (`app.js:304-306`). No authorised-collector field.
3. Two adapters AD-12 (Belkin) and AD-15 (Anker) in the same bin C4
   (`app.js:45-56`), but return never verifies which physical unit came back.
4. Request owner (Isha) != collector (Kabir); authorisation lives on another
   tab and is not surfaced at issue time (`app.js:10`, `60-66`, `254-257`).
5. The 7:30 pm desk-book entry only appears after advancing (`app.js:314`);
   skipping straight to Saturday removes handover context from the records.

### Product / workflow

1. Three manual form fills for a three-item kit; no completeness view or
   prefill (`app.js:161-195`, `223-265`).
2. Due dates are decorative: stored and displayed (`app.js:262`, `290`) but
   never compared; overdue and not-yet-due slips look identical.
3. Time control is ungated and one-way (`app.js:386-389`). Skipping both
   phases reaches the final prompt with no adapter slip to close; no back.
4. No undo/edit/void. Available actions confirmed as advance, tab, new,
   scope, return, restart. Only a full restart recovers a mistake.
5. Return is a boolean flip (`app.js:409-415`): no time, condition, operator,
   or physical verification.
6. Inventory shows status but not who/which request/when due (`app.js:197-221`).
7. Handover note is a bare 400-char textarea with no template, no slip link,
   no edit/delete (`app.js:320-334`, `429-434`).
8. Quantity rule models "one open slip per type per request", not approved
   quantity or re-issue history (`app.js:449-457`).

### Technical

1. `restore()` (`app.js:463-481`) does not validate `who`, `due`, `back`, or
   duplicate slip numbers.
2. Inconsistent escaping: `head()` and request detail inject raw strings
   (`app.js:143-150`, `166-184`) while other helpers use `esc()`.
3. Validation errors write straight to the status line without `commit()` or
   field focus (`app.js:445-455`).
4. No tests and no migration path for the `version: 1` payload.

## Draft — Paragraph 1: Working the shift

**Friday - 3:45 pm - pickup.** I opened RQ-214 (Courtyard screening, Film
Society, filed by Isha Rao, return by Fri 8 pm). It approved three items: HDMI
projector, portable speaker, USB-C -> HDMI adapter, with the note that playback
is from a USB-C-only laptop and "the host will collect at 4 pm." Inventory
showed PR-06 "In repair" (Workshop), MI-09 "On loan", and everything else on
shelf. The issue form's unit dropdown offered only PR-04, SP-02, SP-05, AD-12,
AD-15 and CA-03 - it correctly hid the broken projector and the microphone
already out with Tara Shah. I issued PR-04, SP-02 and AD-12 to Kabir Sen, each
due Fri 8 pm, producing slips L-402, L-403 and L-404. I chose AD-12 over the
identical-type AD-15 simply because it appeared first in the dropdown; the app
gave me no basis for the choice. Issuing took three separate form fills -
nothing offered "issue the approved kit" or told me when the kit was complete.

**Friday - 7:30 pm - partial return.** The Desk book now contained Kabir's
message: projector and speaker back, Leela Das has the adapter, at the counter
at 9 am. I closed L-402 (PR-04) and L-403 (SP-02) with "Record return", leaving
only L-404 outstanding. I wrote a handover note naming the unit, slip, holder,
promised time, and asking the next shift to check the physical adapter tag on
return.

**Saturday - 9:00 am - handover.** I took over and found two open slips:
L-404 - AD-12, due Fri 8 pm, and L-401 - MI-09, due Sat 10 am. The shift asked
me to "close the loan for the right item." I clicked "Record return" on L-404;
the status read "AD-12 returned to stock. Slip L-404 closed."

## Draft — Paragraph 2: One moment it handles well, one moment I'd change

**Handles well - the availability and quantity guards at pickup.** When I
opened the issue form, the broken projector PR-06 was absent from the dropdown
(`repair: true`, stored in the Workshop), and so was MI-09 because Tara Shah
still had it out on L-401. I tried issuing CA-03 (digital camera) against
RQ-214 and got: "This equipment type is outside the approved request. Choose
an approved unit." Then, with AD-12 already issued on L-402, I tried to issue
the second physical adapter AD-15 against the same request and got: "The
approved quantity of this equipment type is already on loan." - even though
AD-15 was sitting on the shelf and offered in the dropdown. Finally, after I
recorded AD-12's return, AD-12 reappeared in the dropdown, so stock state and
slips stayed in sync. As a result I could not hand Kabir a broken projector,
unapproved equipment, or two adapters for a one-adapter approval, and returning
a unit made it available again. Those are specific decisions the product got
right, and I would keep them.

**Would change - closing the adapter loan on Saturday morning.** The task was
to "close the loan for the right item," but my only action was a "Record
return" button on slip L-404 - AD-12. The clock read Saturday 9 am while the
slip said "Due Fri 8 pm" - 13 hours late - yet a search of the register showed
no overdue or late cue at all; L-404 and the not-yet-due L-401 were displayed
identically. I closed it with one click and the app declared "AD-12 returned to
stock." It never asked me to confirm the physical unit. To test this I restarted
and issued the other adapter, AD-15 (Anker), instead. The Saturday prompt and
register were identical, and the same single click reported "AD-15 returned to
stock" - after which both adapters showed "On shelf". In other words the app
closes whichever slip was created at issue; it has no way to establish which
physical unit Leela actually handed over. Two adapters, AD-12 (Belkin) and
AD-15 (Anker), share bin C4, so a wrong close would silently corrupt stock.
The app also had no gating: at Friday 3:45 pm I was able to close L-401, the
microphone due Saturday 10 am, without any check. It captured no condition,
time, or operator. The same identity weakness appears in the stored records:
two slips with the same number are accepted, and clicking "Record return" on
the second row closes the first match instead. That is the moment I would
change: require the physical tag and the slip identity to be confirmed before
closing, surface the overdue state, and record condition and time.

## Draft — Paragraph 3: change, walkthrough, keep, validation

### The change (prototype)

A single **Return desk confirmation step** at the point where an outstanding
slip is closed. Before a return is accepted, the operator sees the issued
unit's ID, model and home bin, must confirm the physical unit on the counter,
sees an overdue flag, records condition and a timestamp, gets an auto-drafted
handover note, and has an undo/void. "A different unit was returned" keeps the
slip open and raises a discrepancy instead of silently changing stock.

Sketch: `prototype-return-desk.html` in this repo (open
`http://localhost:8000/prototype-return-desk.html`). It is a separate static
mockup; the shipped app is untouched.

### Walkthrough of the same job using the change

**Friday 3:45 pm — pickup.** Unchanged. I issue PR-04, SP-02 and AD-12 to
Kabir Sen on L-402/403/404. The repair flag, on-loan exclusion, type approval
and quantity guard still stop me giving him PR-06, MI-09, a camera or two
adapters. The change does not touch anything that already worked.

**Friday 7:30 pm — partial return.** When the projector and speaker come back,
each return now opens the confirmation panel. It shows "L-402 · PR-04 · BenQ
MH550 · bin A2", and I confirm the tag on the projector in front of me, choose
condition "Good", and the time is stamped. Same for SP-02. The handover note is
no longer blank: it is drafted from the two remaining open slips and reads
essentially "AD-12 still out with Leela Das, due Fri 8 pm, promised Sat 9 am;
L-401 microphone still out until Sat 10 am." I edit it rather than reconstruct
it from memory.

**Saturday 9:00 am — handover.** The register shows L-404 flagged "OVERDUE
13h"; L-401 is shown too but labelled "not due yet — other club", so the two
cannot be confused. The confirmation panel asks "Does the physical unit on the
counter match AD-12, Belkin AVC002, bin C4?" I check the label. If it matches,
one click closes the slip with condition and time recorded, and the handover
note is updated automatically. If Leela actually handed me the Anker AD-15, I
choose "different unit returned": L-404 stays open, a discrepancy is recorded,
and inventory is not silently changed. If I close the wrong slip by mistake,
Void/undo reverses it. The next shift can finish the job from the record alone.

### What I would keep

- The availability and quantity rules: repair units excluded, on-loan units
  excluded, approved type checked, one adapter per request, and stock restored
  on return. These are the specific decisions that worked in my shift.
- Slips tracking physical unit IDs rather than just equipment types — that is
  the foundation the confirmation step builds on.
- The four-register structure and fully offline operation. I would also keep
  the Desk book as the place club messages arrive; the change only makes the
  handover note structured instead of a blank textbox.

### One observation at a real counter

Watch partial returns at a real equipment desk for a week before and after the
change and count **how often the physical tag on the returned item differs from
the slip the operator closes, and when that mismatch is discovered**. If the
change helps, mismatches get caught at the counter (a discrepancy is raised)
and the number found later at stocktake falls to near zero; the operator can
also complete the handover without re-reading the whole register. If staff
simply click "yes" without checking the label, the mismatch count stays the
same and the change did not help. That single observation is falsifiable and
directly tests the moment I changed.

## Task log — work done, in order

1. **Fork + clone** — created `ashishkumar006/campus-stores` via the GitHub API
   using the stored git credential, cloned it here, added `upstream`. Done.
2. **Read source + brief** — read README, index.html, app.js, styles.css and the
   licence; extracted the Candidate brief and submission steps. Done.
3. **Play the shift** — automated all three phases in the in-app Chromium,
   issuing PR-04/SP-02/AD-12, recording the partial return, writing a handover
   note and closing the adapter. Done.
4. **Audit + edge-case sweep** — identified data, workflow and technical issues;
   systematically tested issue-form guards, returns, notes, phase control and
   saved-state restore; recorded acknowledged gaps. Done.
5. **Paragraph 1 draft** — shift narrative with exact slips, units and status
   messages. Done.
6. **Paragraph 2 draft** — good moment (availability/quantity guards, stock
   restoration) and change moment (Saturday adapter return: no overdue cue, no
   physical verification, alternate-adapter simulation, no phase gating,
   duplicate slip numbers). Done.
7. **Notes file** — this file created and updated after each stage. Done.
8. **Paragraph 3 draft** — prototype (`prototype-return-desk.html`), same-job
   walkthrough, what to keep, and a falsifiable real-counter observation. Done.
9. **Verify prototype** — opened `prototype-return-desk.html` in the browser;
   headings, overdue flag and confirm button all present. Done.
10. **README assembly** — README rewritten to contain the full submission
    (decision, paragraphs 1-3, links to prototype and notes) while keeping the
    original run/source/submit sections. Done.
11. **Commit + push** — committed with message "Add candidate submission
    write-up and return desk prototype" and pushed to `origin/main` on the
    public fork `ashishkumar006/campus-stores`. The final commit URL is the head
    commit on that branch: https://github.com/ashishkumar006/campus-stores/commit/
    followed by the SHA shown by `git rev-parse HEAD` (or copied from the
    fork's Commits page).
12. **Google Form** — cannot be completed without the roll number and email ID;
    these must be entered by the candidate, and only in the form. The form also
    needs the final commit link above.

## Remaining

Only the Google Form submission, which requires the candidate's roll number and
email ID. No app code changes are required.
