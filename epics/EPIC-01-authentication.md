# EPIC 1 — Authentication & Entry
**Status:** Approved for build · **Design:** locked (pack v4, `design-auth-pack.html` + `assets/design/auth/*.png`)
**Theme:** A · Playful Bold (white theme) · **Priority:** P0 — the front door; nothing ships without it
**Personas:** Nino (new student, first ever open) · Mariam (returning student, shared tablet) · Coordinator (admin, sends invites)
**Design reference:** chat version card `design-auth-pack.html` · live `earendilm83.github.io/design-auth-pack.html`

## 1 · Goal
A student who was pre-provisioned by their school reaches the roadmap in under 60 seconds, from any starting point (SMS invite, email invite, returning session, shared device, expired session) — with zero dead ends, zero blame, and zero progress lost.

## 2 · In scope
Invite dispatch (SMS + email) · splash with loader tips · offline splash · tabbed login with source-aware default · format validation + CTA gating · SMS OTP (enter, wrong, resend) · email magic link (sent state) · not-enrolled recovery · expired-link recovery · non-destructive session re-auth · account switcher + confirm · warm start.

## 3 · Out of scope
SSO/OAuth connect (post-auth, Epic 2) · account creation by students (schools provision; no self-signup exists, ever) · password flows (magic-link/OTP only, D-14) · Georgian localization of auth strings (en MVP, D-19) · RA-03 SSO handoff, RA-05, RA-12/13/14 (later epics).

## 4 · The laws (test every story against these)
- **L1 · No enumeration.** Any unknown identifier returns the same uniform "if this is on file, we sent it" response. Never "not found".
- **L2 · No dead ends.** Every failure state carries exactly one primary recovery action.
- **L3 · No progress loss.** Session expiry, app kill, or account switch is always non-destructive.
- **L4 · CTA gates on validity.** Primary buttons activate only when the input format validates — never "tap to discover an error".
- **L5 · One invite, two channels.** SMS and email fire together, same link, valid 7 days from issue, single use.
- **L6 · Design-locked visuals.** Screens match `assets/design/auth/*.png` 1:1 (white theme; violet #5057FC accents; Kata blue cat).

## 5 · Design tokens (Epic-1 subset)
| Token | Value | Used for |
|---|---|---|
| brand.violet | #5057FC | brand moments (email CTA, splash accents, Retry, join) |
| brand.violetDeep | #072AC8 | CTA bottom lip |
| action.green | #58CC02 / lip #58A700 | primary CTAs |
| action.blue | #1CB0F6 / lip #1899D6 | secondary CTAs (resend, contact) |
| feedback.error | #FF4B4B + 15% glow ring | invalid fields, wrong OTP boxes |
| feedback.ok | #D7FFB8 bg · #46A302 text | "looks right ✓" chips, success |
| surface | #FFF card · #F7F7F7 page | all white-theme screens |
| text | #131F24 primary · #777 secondary · #AFAFAF disabled/caption | |
| radius | 12px buttons · 14px cards · 99px chips | |
| button | 5px solid bottom lip (darker shade), 900 13px caps | |

## 6 · Backend contracts (shared by all stories)
| Endpoint | Rule |
|---|---|
| `POST /auth/invite` (admin) | Creates account + fires SMS + email with same one-time link. Link TTL = 7 days from issue, single use, reissuable by coordinator anytime. |
| `POST /auth/start` `{identifier}` | Detects phone vs email server-side. Phone → SMS OTP (6 digits, 10-min expiry, 3 resends/day, 60s cooldown). Email → magic link (same rules as invite link). **Uniform 200 response regardless of enrollment state.** |
| `POST /auth/verify` `{identifier, code}` | Wrong code: 401 + attempts counter (3 tries → 10-min pause). Correct: session (30-day refresh, sliding). Rate-limited per identifier + IP. |
| `GET /auth/session` | Returns active session or 401 (triggers RA-11 overlay, never a hard logout screen). |
| `POST /auth/switch` `{user_id}` | Switch between device-cached accounts; each keeps its own session + in-flight state. |
| Push channels | SMS via school's SMS provider; email via school's mailer. Both carry identical deep link `https://10x.school/j/{token}` (app link with identifier pre-filled). |

## 7 · Telemetry
`auth_invite_sent{channel}` · `auth_open` · `auth_start{route}` · `auth_identifier_valid{route}` · `auth_identifier_invalid{route}` · `auth_otp_sent` · `auth_otp_wrong{attempt}` · `auth_otp_success{route}` · `auth_resend_used{n}` · `auth_recovery_shown{kind}` · `auth_switch` · `auth_warm_start` · metric targets per story below.

---
# USER STORIES

## US-01 · Invite arrives (SMS + email)
**As a** new student, **I want** to receive my sign-in link on both my phone and my email, **so that** I can start on whichever I check first.

**Screens:** SMS message card + email template (`assets/design/auth/sms.png`, `email.png`).

**Description:**
- SMS (approved, locked): "Nino, your seat at 10x is ready 🎓 Sign in with this number — one tap: 10x.school/j/N-4812" + second bubble: "Same link just landed on your email too. Your streak is waiting 😼"
- Email: white card on warm white; 10x logo (dark + violet x) top-left; floating Kata with violet sparkles; kicker "YOU ARE IN — ALMOST" (violet caps); H1 "Your seat is ready, Nino."; body "Your coordinator already created your account — no forms, no passwords. One tap and you are on your path."; glowing violet button **SIGN IN TO 10X**; footer "link works once · valid 7 days · a fresh one is one message away".

**Behavior:** Link deep-opens the app with the identifier pre-filled and the correct tab pre-selected (SMS→phone, email→email, US-03). Single-use; reissue by coordinator creates a new token (old one dies).

**Acceptance criteria:**
1. Given the coordinator provisions a student, when provision completes, then SMS and email are both dispatched within 60s carrying the same token.
2. Given the link, when tapped >7 days after issue or after one successful use, then it routes to RA-07 (expired), never to a raw error.
3. Given a fresh link, when tapped on the phone that owns the number, then the app opens RA-02 with the phone tab pre-selected and the number pre-filled.
4. Given the email link, when tapped, then RA-02 opens with the email tab pre-selected and the address pre-filled.
5. Both messages render correctly in SMS apps and Gmail/Outlook mobile (no broken layout at 320px width).
6. `auth_invite_sent` fires per channel; delivery failures alert the coordinator, never the student.

**Edge:** student changes phone/email before first use → coordinator reissues to new contact; revoked accounts' links return RA-06.

---

## US-02 · Splash with loader tips (RA-01)
**As a** student opening the app, **I want** a fast branded loader that teaches me something while it checks my session, **so that** the first second already feels like the product.

**Screen:** `assets/design/auth/splash.png`.

**Description:** Warm white background, tiny violet sparkles, large "10x" wordmark (dark + violet x) centered, below it a white card with gold caps label "TIP WHILE WE LOAD" and one tip in bold dark text, a thin violet progress bar beneath, caption "checking your session…", Kata peeking from the bottom edge. Brand ≤1s to first paint; if the session check exceeds 3s, the bar becomes determinate-feeling.

**Tip rotation (exact set, one visible per open, cycles):**
1. "Six minutes a day beats a Sunday panic. Your first floor is shorter than this loading bar."
2. "Your streak counts floors, not perfection. Showing up is the whole game."
3. "Mistakes aren't failures here — they literally pay you back in coins."
4. "One floor today is worth more than five floors you never start."

**Acceptance criteria:**
1. First paint (wordmark) renders ≤1s on 3G.
2. Session check >3s → progress bar appears (never a lone spinner, never blank).
3. A different tip shows per open (sequential rotation, persisted index).
4. Session valid → route RA-10 (warm start). No session → route RA-02. Deep link → queue intent, route after auth.
5. Offline at check → route to RA-01 offline state (US-03) — no crash, no hang.
6. `auth_open` + tip_index logged. Metric: cold→routed ≥90% under 3s on 3G.

---

## US-03 · Splash offline state (RA-01 variant)
**As a** student with no connection, **I want** an honest, funny offline screen with one clear action, **so that** I know nothing is lost and what to do next.

**Screen:** `assets/design/auth/offline.png`.

**Description:** Warm white bg, sad Kata with droopy ears holding an unplugged wifi plug, H1 "I was about to say something wise.", sub "Then the internet left. It's not you, it's the wifi — your floors are saved right here on the phone.", one chunky violet button **RETRY** (5px lip #072AC8).

**Acceptance criteria:**
1. Retry is the ONLY action on screen (L2).
2. Retry re-runs the session check; on success routes per US-02 §4; on failure returns here with no animation loop.
3. Connectivity restore (OS event) auto-retries silently once; shows "syncing ↻" pill on success.
4. No cached content is shown as interactive — this is a dead-end-proof stop, not a fake app.
5. Offline floors (if any cached) are never mentioned as playable here — the route to them is via RA-10 only.

---

## US-04 · Login — tabbed, source-aware (RA-02)
**As a** student, **I want** to sign in with the phone number or email my school registered, **so that** I never need a password.

**Screen:** `assets/design/auth/login.png` (valid state), `login-invalid.png` (invalid), `login-email.png` (email tab).

**Description:** Kata (large) with speech bubble "New? Sit anywhere. The register sorts itself." H1 "Sign in". Sub "your number — the one your school has". Segmented tabs **📱 Phone | ✉ Email**; default tab = invite channel (SMS→Phone, Email→Email, direct open→Phone). Single field per tab (phone: +995 prefix, 9 digits; email: standard). Caption below: "from your SMS link · phone pre-selected" (or email variant). Bottom note: "School gave you credentials? They work right here." **No SSO, no password, no signup link — none of these exist in this product.**

**Behavior:** live detection on the phone tab is unnecessary (tabs are explicit); validation per US-05; on valid submit → `POST /auth/start`; route phone→US-06 (OTP), email→US-09 (link sent).

**Acceptance criteria:**
1. Given an SMS invite arrival, when RA-02 opens, then Phone tab is selected and the number is pre-filled.
2. Given an email invite arrival, when RA-02 opens, then Email tab is selected and the address is pre-filled.
3. Given a direct app open, when RA-02 opens, then Phone tab is selected, field empty.
4. Tab switch clears the error state but preserves the typed text in the other tab.
5. The field never accepts >9 digits (phone) or invalid email characters; submit button state follows US-05 at all times.
6. Metric: first-sign-in completion ≥90% from credentials message.

---

## US-05 · Format validation & CTA gating (RA-02 states)
**As a** student, **I want** the button to wake up only when my input is valid, **so that** I never submit garbage and get scolded for it.

**States (all three designed):**
- **Empty/typing:** field neutral, CTA disabled (grey #E5E5E5, text #AFAFAF).
- **Invalid:** field red border + 15% red glow ring; red hint below — phone: "that doesn't look like a phone number — 9 digits after +995"; email: "that doesn't look like an email — missing the dot?"; CTA disabled.
- **Valid:** field blue border + focus ring, green chip "looks right ✓" inside field; CTA active (green, label by route: **TEXT ME A CODE** / **EMAIL ME A LINK**).

**Acceptance criteria:**
1. Phone validates at exactly 9 digits after +995; email validates per RFC-lite (x@y.zz).
2. State transitions are instant per keystroke (no submit-time surprises, L4).
3. Invalid → user fixes to valid → error clears and CTA activates without re-tap.
4. `auth_identifier_valid/invalid{route}` logged. Metric: invalid→valid correction ≥80% without leaving the screen.

---

## US-06 · OTP entry (RA-15)
**As a** student, **I want** to type the 6-digit code from the SMS and have it submit itself, **so that** I'm in without an extra tap.

**Screen:** `assets/design/auth/otp.png` (with the resend-ready state visible).

**Description:** H1 "Texted you a code". Sub "+995 ••• •• 34 56 · expires in 10:00". Six boxes: filled digits bold, current box blue caret, remaining grey. Caption "submits itself at six digits". Bottom: button (see US-08 for its states) + "resend in 0:42" caption while cooling down. Small Kata peeking top-right.

**Acceptance criteria:**
1. Digits auto-advance; backspace steps back; paste of a 6-digit code fills all six and submits.
2. Sixth digit → auto-submit to `/auth/verify` (no button tap needed).
3. Correct → session issued → route RA-10.
4. Wrong → US-07 state. Expired (10:00) → resend-ready state (US-08).
5. Verify endpoint: 3 wrong tries → 10-min pause, shown as "take a breath — try again at {time}".
6. Metric: OTP first-try success ≥80%.

---

## US-07 · OTP wrong code (RA-15 error)
**As a** student who typed the wrong code, **I want** a clear, kind error with my tries left, **so that** I know exactly what to do and never panic.

**Screen:** `assets/design/auth/otp-wrong.png`.

**Description:** All six boxes red border + red glow, containing the wrong digits. Red message: "not it — check the newest SMS · 2 tries left before a 10-minute pause". Worried Kata peeking. Button: white grey-bordered **CLEAR & RETYPE**. Caption "resend in 0:31".

**Acceptance criteria:**
1. Error shows remaining tries (server count) and the pause rule verbatim — never a generic "invalid".
2. Clear & retype empties all boxes and refocuses box 1.
3. Third wrong attempt → pause state replaces boxes with "take a breath — try again at {time}" + resend disabled until pause ends.
4. Attempts counter is server-side (no client bypass).

---

## US-08 · OTP resend & cooldown (RA-15 resend-ready)
**As a** student whose code never arrived or expired, **I want** an active resend button with honest limits, **so that** I can recover without support.

**Description:** Resend-ready state: caption "nothing landed? that's on us, not you", active blue chunky button **RESEND THE CODE** (lip #1899D6), caption "ready now · 3 resends left today".

**Acceptance criteria:**
1. During cooldown: caption counts down "resend in 0:42", button disabled.
2. At 0: caption becomes "ready now · {n} resends left today", button activates.
3. Resend issues a NEW code (old dies), resets the 10:00 expiry, decrements the daily count (3/day/identifier, server-side).
4. After the 3rd resend: button replaced by "that's today's last one — your coordinator can send more" (L2 preserved).

---

## US-09 · Magic link sent — email route (RA-04)
**As a** student using my email, **I want** a calm "check your inbox" card, **so that** I know exactly where the link went and how long it lives.

**Screen:** `assets/design/auth/email.png` re-used as the sent state; plus a small "✉ email route" tag.

**Description:** "✉ email route" chip. H1 "Check your inbox". Body "we sent a sign-in link to **nino@gmail.com**. It works once, for **7 days**." Proud Kata. Buttons: blue **OPEN MAIL APP**, ghost **SEND IT AGAIN**. Footer "wrong address? your coordinator fixes it in a minute".

**Acceptance criteria:**
1. Link TTL 7 days, single use; opening it on any device signs in and routes to RA-10 (or onboarding if first ever).
2. "Send it again" follows US-08 resend rules (shared counter).
3. "Open mail app" deep-links to the default mail client when available; otherwise shows the address to copy.
4. The card never reveals whether the address is enrolled (L1) — text is identical either way.

---

## US-10 · Not-enrolled recovery (RA-06)
**As a** student whose identifier isn't on the register, **I want** one fast fix, **so that** I'm never stuck at a wall.

**Screen:** `assets/design/auth/not-enrolled.png`.

**Description:** Worried Kata with clipboard. H1 "We don't know this number yet". Body "Your school adds every student by hand — this one isn't on the register. If you just enrolled, give it an hour." Grey card "Fastest fix — Message your coordinator — they add you in a minute." Buttons: blue **CONTACT MY COORDINATOR**, ghost **TRY A DIFFERENT IDENTIFIER**.

**Acceptance criteria:**
1. Shown only after a *successful* `/auth/start` on a non-enrolled identifier (L1: the uniform response already happened; this state is the truthful continuation).
2. Contact CTA opens the school's coordinator channel (mailto/deep link from tenant config) with a pre-filled subject including the masked identifier.
3. "Try a different identifier" returns to RA-02 with the field cleared.
4. Metric: not-enrolled → successful sign-in within 24h ≥60%.

---

## US-11 · Link expired recovery (RA-07)
**As a** student whose link expired, **I want** a fresh one in ten seconds, **so that** expiry is a shrug, not a blocker.

**Screen:** `assets/design/auth/expired.png`.

**Description:** Sleepy Kata hugging an envelope with a broken seal. H1 "That link expired". Body "Sign-in links live 7 days and work once. A fresh one takes ten seconds." Buttons: green **SEND A FRESH LINK**, ghost **USE MY PHONE INSTEAD**.

**Acceptance criteria:**
1. Fresh-link action follows US-08 counter; issues new token, kills the old.
2. "Use my phone instead" routes to RA-02 phone tab, identifier pre-filled if known.
3. Expired-token visits after a successful first use route here too (never a dead token error page).

---

## US-12 · Session expired → re-auth (RA-11)
**As a** student whose session timed out mid-task, **I want** a quick check that keeps my half-typed work, **so that** expiry never costs me anything.

**Screen:** `assets/design/auth/reauth.png` (locked reference — dimmed Flexbox floor behind, Kata peeking).

**Description:** Background = the student's actual screen dimmed and desaturated (e.g., the floor runner). Bottom sheet (rounded top, grab handle, green top border): H1 "Quick check it is you". Body "your session timed out. Your half-typed answer is right where you left it — promise." Field pre-filled with the remembered identifier + blue pill "this device remembers you". Green **TEXT ME A CODE**. Caption "switch account".

**Acceptance criteria:**
1. Any 401 from `/auth/*` or app APIs triggers this overlay in place (never a hard logout screen).
2. Re-auth via the same OTP flow (US-06) restores the previous route AND the in-flight state byte-for-byte (typed answer, scroll position, timer state).
3. "Switch account" routes to RA-08 without dismissing the in-flight state of the current account.
4. Overlay is dismissible to read-only viewing (no input possible) — the work stays visible but locked.
5. Metric: re-auth → task resumed ≥95% with zero state loss.

---

## US-13 · Account switcher (RA-08)
**As a** student on a shared class tablet, **I want** to switch between remembered accounts in one tap, **so that** my friend and I never mix our progress.

**Screen:** `assets/design/auth/switcher.png`.

**Description:** H1 "Who's learning?" Sub "this tablet remembers two of you". Account cards: avatar circle, name, "🔥 {streak} · {program}"; current account has blue border + glow + green "current" pill. Ghost button "+ Add account". Footer "each of you keeps your own streak, floors and coins — always".

**Acceptance criteria:**
1. Lists all device-cached accounts (max 3 shown, oldest drops off silently with notice).
2. Tapping a non-current account opens US-14.
3. "+ Add account" routes to RA-02 while preserving every cached session.
4. Cards show live streak values (fresh from server when online, cached-with-timestamp offline).

---

## US-14 · Switch confirm sheet (RA-09)
**As a** student about to switch accounts, **I want** a clear promise before I commit, **so that** I never worry about losing anything.

**Description:** Bottom sheet over dimmed switcher. H1 "Switch to Mariam?" Body "Nino's session pauses mid-floor 13 — nothing is lost, nothing mixes." Green **Switch — hi Mariam 👋**. Ghost **Stay Nino**.

**Acceptance criteria:**
1. Confirm → new account's session resumes at ITS last route/state; the previous account's in-flight state is persisted server-side before the switch.
2. Cancel dismisses the sheet unchanged.
3. Post-switch, the HUD/streak/coins immediately reflect the new account only (no cross-pollution, ever).

---

## US-15 · Warm start (RA-10)
**As a** returning student, **I want** to skip login entirely and land on my path, **so that** opening the app costs me nothing.

**Screen:** `assets/design/auth/warmstart.png`.

**Description:** Warm white bg, big "10x" wordmark, below it a soft violet pill: avatar + "welcome back, Nino · 🔥 12", happy Kata waving beside it, thin violet progress bar ~80%, caption "syncing your floors… opening your path".

**Acceptance criteria:**
1. Valid session → this screen ≤1.5s, then route to the student's last surface (roadmap home by default).
2. The streak/XP/coins shown in the pill match server values before routing (no stale flash).
3. Sync failure still routes (stale-while-revalidate); shows "synced 2h ago" honesty note instead of hanging.
4. Metric: warm-start → interactive ≤2s on 3G.

---

## 8 · Definition of done (Epic 1)
- [ ] All 15 stories pass their acceptance criteria in staging (incl. negative tests: wrong OTP ×3, expired link, offline, not-enrolled).
- [ ] L1–L6 verified in code review + QA script.
- [ ] Visual match to `assets/design/auth/*.png` at 390×844 (±2px on key elements).
- [ ] Telemetry events present in the analytics stream with the exact names in §7.
- [ ] Invite TTL = 7 days everywhere (template, RA-04, RA-07 copy, backend).
- [ ] No SSO, password, or self-signup affordance exists anywhere in the auth bundle.

## 9 · Dependencies & next epic
Depends on: tenant config (coordinator contact, SMS/email providers), session service, rate limiting. Blocks: **Epic 2 — Onboarding** (RB-00…RB-13): first-run commitments (program confirm, goal, placement, contract, day-1 quest, GitHub connect, archetype).
