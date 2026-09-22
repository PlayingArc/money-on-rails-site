# Privacy Policy rewrite: notes for review

Branch `privacy-rewrite` of `PlayingArc/money-on-rails-site`, cut from `main` at `7615521`. Checked against the app repo `PlayingArc/money-on-rails` at `main` = `cf15d92`.

**Drop this file before merge.** The site repo has no `.scratch/` convention, and GitHub Pages would publish this file at `/PRIVACY-REWRITE-NOTES.md`.

Files changed: `privacy.html` (full rewrite of the content, same HTML skeleton, stylesheet, header, footer, badge and disclaimer) and `index.html` (one link added). `terms.html` needed no change: it never named Anthropic or Claude and never said "budget".

## Section by section

| Section | What changed and why |
|---|---|
| `<title>` | The title's em dash became a bar: "Privacy Policy \| Money on Rails" to meet the no-em-dash rule, matching the app's own titles. `terms.html` still has the em dash in its title; it was left alone because the brief said to change nothing else there. |
| Header comment | Keeps the CC BY 4.0 attribution to Basecamp's policies and the "not reviewed by a lawyer" note. Drops the old "published to satisfy YNAB's review" wording. |
| Last updated | September 22, 2026. |
| Intro | Same operator identity as before (Gerardo Sanchez Lopez, individual, Nuevo León, Mexico, not a company), same not-affiliated sentence, same "we never sell". Adds "we don't show ads". |
| The short version (new) | Four bullets a reviewer or user can read in ten seconds. Each repeats a claim sourced below. |
| What we collect and why | Rebuilt from the migrations instead of the design doc. New rows: plan name/currency/month range, target type and amount, Income Sources and payee groupings, classification history with model id, session, invite note, request form, emails sent to us. Adds a paragraph that YNAB's full-plan export sends more than we keep, and we store only the listed parts. |
| How we use AI to sort your Categories (new section) | Replaces the Anthropic bullet. Names OpenAI GPT-5.6 Luna, says exactly what is sent (names only; 8 per request for Needs/Wants/Savings; full list once per archetype for true expenses), no ids or identity, names sent as typed, no training, 30-day abuse-monitoring retention not shortened by account deletion, no cross-User use. |
| Who else handles your data | Anthropic removed. OpenAI added. Adds Cloudflare (email forwarding) and GitHub (hosts the static site), because the list is meant to be complete. Drops the hypothetical payment processor. Drops "Google Cloud holds our encrypted database backups" (not built). Adds operator-access and legal-request commitments in plainer words. |
| Asking for an invite (new section) | ADR 0017's three required disclosures, plus the note-becomes-invite-note detail from the code. |
| How we protect your data | Adds keyed hashes for sessions and invites, log allowlist with what is *not* on it, and Google Cloud's own request logs (which do hold IP addresses). Drops "database backups are also encrypted" (not built, not verified). |
| Deleting your account | Now matches ADR 0005's second 2026-09-18 amendment: sign-out at request, daily job, usually within a day, 30 days as the outer bound, no cancellation, sign-in refused while pending, return means a new invite. Keeps the backup tail (7 days) and the can't-revoke-at-YNAB fact. Adds OpenAI's 30-day copy. |
| How long we keep data | Every data class from the migrations and ADRs has a row, including request form, hashed IP, OpenAI, backups, logs and emails. The old "37 days total" line is gone as a single number; the parts (30-day outer bound, 7-day backups) are each stated. |
| Your choices and rights | Same rights, no inline-header bullets. |
| Cookies and tracking (new) | The two sign-in cookies, the theme in localStorage, no analytics, and the static site's third-party loads (Google Fonts, YNAB badge). |
| Who can use Money on Rails | Adds what happens to someone who approves at YNAB without an invite (ADR 0017). |
| California, Where your data is, Changes, Contact | Same substance, plainer words. Contact adds "your invite request". Address kept as privacy@moneyonrails.app. |

## Source for every data claim

App repo paths are relative to `PlayingArc/money-on-rails` at `cf15d92`.

| Claim in the policy | Source |
|---|---|
| YNAB gives only an opaque user id, no name or email | ADR 0017 ¶2; `migrations/0002` (`users.ynab_user_id`) |
| Read-only access, the narrowest YNAB offers | `src/ynab/oauth.ts` (`YNAB_OAUTH_SCOPE = "read-only"`) |
| Plan name, currency format, month range | `migrations/0002` (`plans`) |
| Category Groups/Categories: names, order, hidden, deleted, target type and amount | `migrations/0005` (`category_groups`, `categories.goal_type`, `goal_target`, `position`) |
| Monthly assigned, activity, available | `migrations/0005` (`category_months.budgeted/activity/balance`) |
| Emergency fund choice | `migrations/0005` (`plans.emergency_fund_category_id`) |
| Needs/Wants/Savings, corrections, needs-review | `migrations/0005` (`categories.final_bucket`, `llm_pick`, `origin`) |
| Income transactions only, fields, no memo | design-doc §7 Transaction; backlog E5.1 ("inflow-only storage filter"); ADR 0001. **E5.1 is being built** |
| Income Sources, payee mappings, your edits | design-doc §7; backlog E6.1 (`is_manual_override`) |
| Classification history with names and model id | design-doc §7 (both Event logs); ADR 0005 amendment 1; ADR 0013 |
| Ratings persisted | design-doc §7 (Category Adherence, Adherence Rating, Quality Snapshot) |
| Full-plan export read, only listed fields stored | `src/sync/months.ts` header comment (`GET /plans/{id}`); `src/ynab/http.ts` |
| OpenAI GPT-5.6 Luna | `src/llm/client.ts` (`MODEL_ID = "gpt-5.6-luna"`); ADR 0003 amendment 2026-09-21 |
| Names only, no ids | `src/llm/call.ts` (`input: JSON.stringify(request.names)`, nothing else in the request but the fixed instructions and schema); ADR 0009; ADR 0013 |
| 8 names per request | `src/llm/classify.ts` (`CLASSIFY_BATCH_SIZE = 8`) |
| Full list once per archetype | `src/llm/archetypes.ts` (`ARCHETYPES.map(... requestFor(archetype, names))`) |
| Runs at first connect and for unsorted Categories later | ADR 0013 amendment 1 (N/W/S covers names with no classification yet); design-doc §9 (trigger is "unclassified categories exist") |
| No training on API data; 30-day abuse monitoring; longer if law requires | `.scratch/.../handoff/llm-openai.md` "Verified OpenAI facts" (developers.openai.com/api/docs/guides/your-data, checked 2026-09-21) |
| No cross-User use of names | ADR 0013 amendment 1 (cache removed) |
| Google Cloud: runs app, KMS master key, secrets, logs; US | `docs/runbooks/deploy.md` (Cloud Run `us-central1`, KMS key in `us-central1`, Secret Manager table, Cloud Logging); backlog §4.3 |
| Neon on AWS, US | backlog §4.3 (`AWS us-east-2`); ADR 0007 |
| Cloudflare forwards privacy@ | backlog §4.1 (Cloudflare Email Routing) |
| GitHub hosts the static site | ADR 0004; site `README.md` |
| Request form: email + optional note only | ADR 0017; `migrations/0004` (`invite_requests`); `src/auth/invite-requests.ts` |
| Only the operator sees requests | `/admin` gated on `ADMIN_YNAB_USER_ID` (ADR 0017; `.scratch/.../handoff/e1-8.md`) |
| Request retention 30 days after joining / 12 months | ADR 0017; `migrations/0004` comment; form fine print in `src/app/_auth/views.tsx`. **Purge job E7.2 not built** |
| Request email becomes the invite note | `.scratch/.../handoff/e1-8.md` ("Minting for a request uses its address as the note") |
| Deletion of a request by email only | ADR 0017 ("manual email to the operator") |
| Keyed hash of IP, never raw IP, one-hour window, deleted on next submission | `src/auth/invite-requests.ts` (`HMAC-SHA256(SESSION_SIGNING_KEY, "invite-request-ip:"+ip)`, `REQUEST_RATE_WINDOW_MS = 3_600_000`, `purgeRequestAttemptsBefore` on every submission); `migrations/0004` (`invite_request_attempts`) |
| Invite note may be an email; redeemed invite deleted with account; no expiry | `migrations/0003` (`note`, `redeemed_by_user_id ... ON DELETE CASCADE`, "no expiry column") |
| HTTPS | Cloud Run domain mapping TLS (`docs/runbooks/deploy.md` step 4); YNAB base URL `https://api.ynab.com/v1` (`src/ynab/http.ts`); OpenAI SDK |
| Envelope encryption, per-token key, KMS master key never leaves KMS | `src/crypto/envelope.ts`, `src/crypto/kms.ts`; `migrations/0002` (`*_ciphertext`, `*_wrapped_dek`); ADR 0005, ADR 0012 |
| Sessions and invites stored as keyed hashes | `migrations/0002` (`sessions.token_hash`), `migrations/0003` (`invites.token_hash`); `src/auth/session.ts` |
| Log allowlist: ids, status, times, counts; no amounts, payees, names, emails, IPs | `src/log/fields.ts` (closed `FIELDS` set; unknown keys dropped in `filterFields`); design-doc §12 |
| Tokens deleted on revoked/expired, history kept | `src/db/connections.ts` (`clear = status === "revoked" \|\| status === "expired"`); `src/sync/months.ts`; ADR 0005 |
| Sign-out everywhere at request, daily job, usually within a day, 30-day outer bound, no cancel, sign-in refused while pending, return is a new Admission | ADR 0005 amendment 2 (2026-09-18); `src/app/(app)/settings/account/account-tab.tsx` ("Deletion runs within a day"); `src/app/_auth/views.tsx` refusal page; `src/auth/admission.ts` (`deletion_pending`). **E6.1 wiring and E7.1 job not built** |
| Deletion covers everything, including names and classification history | ADR 0005 amendment 1; every table in `migrations/0002`–`0005` cascades from `users` |
| Can't revoke at YNAB | ADR 0005 (first Considered Option) |
| Backups at most 7 days | ADR 0010 (7-day PITR and 7-day GCS rotation). **E7.3 not built**; today Neon Free keeps 6 hours of history, which is inside the bound |
| Session ends 30 days after last use, on sign-out, on deletion | `src/auth/session.ts` (`SESSION_TTL_MS`, sliding); `migrations/0002` (cascade); sign-out route |
| Two cookies, one about 10 minutes | `src/app/_auth/cookies.ts` (`mor_session`, `mor_signin_nonce`); `src/auth/state.ts` (`STATE_TTL_MS = 10 min`). `mor_minted_invite` exists but is set only for the operator on `/admin`, so it is not mentioned |
| Theme kept in the browser | `src/app/_theme/theme-store.ts` (localStorage) |
| No analytics | grep of `src/`, `package.json` and the site for common analytics SDKs found none; `next/font/google` self-hosts fonts at build |
| Static site loads Google Fonts and the YNAB badge | `assets/style.css` (`@import` from fonts.googleapis.com); footer `<img src="https://api.ynab.com/papi/works_with_ynab.svg">` |
| Uninvited approval: turned away before any token, no account | ADR 0017 ("refused before the authorization code is exchanged") |
| US-only, 13+, California, operator in Mexico | carried from the previous policy and `terms.html`; not data claims |

## YNAB reviewer's requests that live on this page

From `.scratch/money-on-rails-v1-build-effort/issues/11-respond-to-ynab-oauth-review.md` and backlog §3 / E7.5.

| Request | Where it is now |
|---|---|
| No placeholder text; a date | "Last updated: September 22, 2026" under the title. No bracketed text anywhere on the page. |
| "Other necessary information" (operator details) | Intro paragraph 2: Gerardo Sanchez Lopez, individual developer, Nuevo León, Mexico. Contact section: privacy@moneyonrails.app. |
| Say "plan", not "budget" | "plan" throughout. The page contains no "budget". |
| Category names and classification outcomes deleted with the account; nothing derived from names outlives the account | "Deleting your account" bullet 3 names Categories and their names and every suggestion and correction; "How long we keep data" row 1 includes Category sorting and its history. OpenAI's own 30-day copy is disclosed separately in both places, since it is outside our control. |
| Category names are free text that may identify someone | "How we use AI" bullet 4: names are sent as typed, including a person's name if it is in one. |
| Deletion "inside the same 30-day window" (the operator's reply to YNAB) | "Deleting your account" bullet 2: usually within a day, always within 30 days. |
| Backup tail stated with no exception (ADR 0005 amendment 1; E7.5 check) | "Deleting your account" bullet 5 and the retention table's backups row: up to 7 days after the job runs. |
| Concrete retention per data class (backlog §3) | "How long we keep data" table. |
| Deletion mechanism (backlog §3) | "Deleting your account" and "Your choices and rights". |
| Tokens encrypted at rest (backlog §3) | "How we protect your data" bullet 2. |
| Complete subprocessor list | "Who else handles your data". |
| Not-affiliated statement and unmodified badge | Intro paragraph 2 (bold) and the unchanged footer. |

## Needs operator confirmation

1. **The deletion flow is not fully built.** E6.1's request wiring (recording the request, ending sessions, the `deletionPending` flag in `src/auth/signin.ts`) and E7.1's daily job don't exist on `main` yet. The policy describes them as they will be at launch. Merge this only once both ship, or accept that the page runs ahead of the code for a few days.
2. **Backups.** E7.3 (Neon Launch plan with 7-day history, daily `pg_dump` to Cloud Storage with 7-day lifecycle) isn't built. The policy says only "up to 7 days", which is true today (Neon Free keeps 6 hours) and after E7.3 if the 7-day settings hold. It no longer says backups are encrypted. Add that back once E7.3 confirms how the bucket is encrypted.
3. **Retention jobs.** E7.2 isn't built. Until it is, invite requests are never purged, and expired session rows stay in the table even though they can't be used. The policy states the 30-day/12-month rule as at launch.
4. **Unredeemed invites have no retention rule.** An issued or revoked invite's `note`, which may be someone's email address, stays until someone deletes it. The policy says so and offers deletion by email. Consider a rule in E7.2.
5. **OpenAI DPA.** Never read or signed (`handoff/llm-openai.md`: openai.com returned 403). Runbook step 2 has the operator accept it. The policy doesn't mention a DPA, so it makes no claim that could be false. Add one after signing if you want to.
6. **OpenAI's stored responses.** `src/llm/call.ts` doesn't set `store: false` on `responses.create`. The Responses API stores response objects by default, which OpenAI's docs describe as kept for 30 days. The policy's "up to 30 days" covers this, but setting `store: false` in the app would be the tighter posture. Please confirm the default against OpenAI's current docs.
7. **Regions.** The runbook and backlog §4.3 put Cloud Run and KMS in `us-central1` and Neon in AWS `us-east-2`. The policy says only "United States". Confirm the live values in the GCP and Neon consoles.
8. **Cloud Logging retention.** Google Cloud keeps request logs (with client IP and URL) and the app's logs for the `_Default` bucket's retention period. That is usually 30 days, but this project's setting wasn't checked. The policy gives no number. Check the bucket and add the number to the retention table.
9. **Invite tokens in request logs.** Invite links are `/invite/<token>`, and Cloud Run's request logs record the URL path. So the raw invite token sits in Cloud Logging even though the database holds only its hash. This isn't a policy claim, but it weakens "stored only as a keyed hash" for anyone with log access. Worth an app-side look.
10. **Emails to privacy@.** Cloudflare forwards them to the operator's personal inbox, whose provider the policy doesn't name. The retention row says they stay until he deletes them. Decide whether to name the mailbox provider as a subprocessor and set a period.
11. **Transfers in the Transaction table.** The design doc keeps `ynab_transfer_account_id` on inflow rows, but E5.1 may drop transfers entirely instead. The policy says we keep the other account's YNAB ID for a transfer. Update the row once E5.1 settles it.
12. **Classification isn't wired yet.** `src/classification/` is only an index file, so no real names reach OpenAI today. The call shape in the policy comes from `src/llm/` and ADR 0013; confirm the E3 units call it that way, especially which Categories (hidden, deleted, YNAB's internal ones) go in the list.
13. **Landing page link.** `index.html` now links to `https://dash.moneyonrails.app/request-invite`. The route exists in the app (`src/app/request-invite/page.tsx`) and the host comes from the runbook. Confirm it serves before merge.
