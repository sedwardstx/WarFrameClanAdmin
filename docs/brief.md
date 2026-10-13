# Project Brief: WarFrame Clan Admin

> **📌 Status: DRAFT, paused for stakeholder answers (2026-10-13)**
> - **Sections 1–10 are drafted.** Still to do: *Appendices*.
> - **Before resuming:** fill in the **Answer** lines under *Risks & Open Questions → Open Questions — Answers Needed* (Q1–Q24). Questions marked 🔴 must be answered before the PRD.


## Executive Summary

**WarFrame Clan Admin** is a lightweight, web-based membership administration tool for a ~1,000-member Warframe clan. Members sign in with Discord, and the tool becomes the clan's single source of truth for who is in the clan, their in-game identity, their rank, and their progression through clan-defined trainings and badges.

**The problem:** A clan at Warframe's maximum (Moon) size can't be managed reliably with spreadsheets, Discord messages and officer memory. Member records, rank changes, training outcomes and conduct history end up scattered, inconsistent and hard to audit. Sensitive leadership notes have no proper access control.

**Target users:** Clan leadership (Warlords, Generals, Officers), who administer members, plus the general membership, who maintain their own bio and track their progression.

**Value proposition:**
- **One roster** that links each member's Discord identity to their Warframe handle and clan rank, with ranks synced to Discord roles.
- **Player Information File (PIF) notes** with tiered visibility. Officers can add notes, but only Generals and Warlords see the full history.
- **Configurable trainings and badges.** Leadership defines them, assigns them and tracks them with clear statuses, and members can see what they're working toward.
- **Zero hosting cost.** It runs entirely on free tiers: Cloudflare Pages and Workers, with Supabase Postgres.

## Problem Statement

**Current state and pain points**

Clan membership for a ~1,000-member clan is currently tracked informally, across some mix of Discord roles and channels, spreadsheets, in-game clan menus and leadership memory. As a result:

- **No single roster.** Nothing reliably links a member's Discord account to their Warframe handle. Leadership can't easily answer "who is this person in-game?" or "is this Discord user actually in the clan?"
- **Rank drift.** In-game ranks and Discord roles are updated by hand, separately, so they fall out of sync. Members end up with the wrong permissions or recognition in Discord.
- **Conduct history is scattered and unprotected.** Notes on member behaviour, warnings and context (PIF notes) live in DMs, private channels or individual officers' heads. They get lost when officers rotate out, and they have no access control matching how sensitive they are.
- **Training and badge progression isn't tracked.** Leadership can't see who has been assigned, who passed, who failed or who still needs a training. Badge awards have no record of who awarded them, so the system is open to inconsistency or self-promotion.
- **Members can't see their own progress.** Members have no place to see their trainings, the badges they've earned or what they could work toward next.

**Impact**

- With ~1,000 members and a relatively small leadership team, every manual lookup or reconciliation multiplies across hundreds of people. Admin time goes into bookkeeping instead of running the clan.
- Rank and role errors cause permission mistakes in Discord and frustration for members.
- Losing conduct history means repeat issues aren't recognised, and leadership decisions can't be justified or reviewed.
- Without visible progression, members have less reason to engage, train and advance.

**Why existing solutions fall short**

- **Warframe in-game clan tools** handle ranks and basic permissions, but they offer no notes, trainings, badges or Discord link, and no public API to build on.
- **Discord alone** (roles, bots, private channels) can show status but isn't a structured record system. Searching and auditing are weak, and history is buried in message scrollback.
- **Spreadsheets** have no per-row access control (a PIF note is either visible or not), no history of edits, no self-service for members, and they get hard to manage at 1,000 rows with several editors.
- **General Discord moderation and leveling bots** are built for chat moderation or XP, not for clan-specific rank structures, training outcomes or role-based note visibility.

**Why now**

The clan is at or near Warframe's maximum clan size. Informal processes don't get any easier from here, only harder as leadership turns over. A free-tier, purpose-built tool fixes this at no ongoing cost.

## Proposed Solution

**Core concept and approach**

A purpose-built clan administration web app with three parts:

1. **Public landing page** (static, Cloudflare Pages). It introduces the clan and offers a single **"Sign in with Discord"** button. No passwords are stored and no separate accounts are created.
2. **Application API** (Cloudflare Worker). It handles Discord OAuth, sessions, and every business rule: who can see which PIF notes, who can award which badges, and so on. Permission checks run on the server, never just in the browser.
3. **Clan database** (Supabase Postgres, free tier). It holds members, ranks, PIF notes, trainings, badges and an audit trail of who changed what.

The app is the **system of record for clan rank**. When leadership changes a member's rank in the app, a Discord bot updates that member's Discord role to match.

**Core capabilities**

- **Member roster:** Each member is linked by Discord ID and Warframe handle and has a rank (Warframe's built-in hierarchy) and a clan role. Searching and filtering stay fast at ~1,000 members.
- **Self-service profile:** Members edit their own bio and choose the badges they're working toward. Everything else on their record is read-only to them.
- **PIF notes with tiered visibility:** Officers and above can add notes, but only Generals and Warlords can read a member's full note history.
- **Dynamic trainings:** Generals and Warlords create trainings. Leadership assigns them, or members assign them to themselves. Each assignment has a status of *Assigned*, *Completed (Passed)* or *Completed (Failed)*.
- **Dynamic badges:** Generals and Warlords create badges. Officers and above award them to *other* members, never to themselves. Members can never award badges.
- **Rank-to-Discord sync:** A rank change in the app is reflected in the member's Discord role.
- **Accountability:** Every rank change, note, training status change and badge award is stamped with who did it and when.

**Key differentiators**

- **Fits Warframe clan structure:** Permissions follow Warframe's own rank ladder, so leadership doesn't have to learn a new model.
- **Access control built for sensitive notes:** Spreadsheets and Discord channels can't offer rank-tiered visibility for PIF history.
- **Separation-of-duties rules enforced by the system:** Creating badges and trainings is limited to Generals and Warlords, awarding badges is limited to Officers and above, and nobody can award themselves a badge. These rules are enforced in code, not by trust.
- **Discord-native identity:** Members sign in with the account they already use, and their Discord identity is linked to their record automatically.
- **$0 to run:** It's designed from the start to fit within the Cloudflare and Supabase free tiers.

**Why this will succeed**

- **Little friction:** One-click Discord sign-in means members don't need to create anything, which matters for adoption across 1,000 people.
- **Narrow scope:** It's a clan admin and records tool, not a game tracker or a general-purpose bot, so it can be built and maintained by a volunteer.
- **Real incentive for members:** Visible trainings and badges give members a reason to log in, which keeps profile data (handles, bios) current.

**High-level vision**

It becomes the clan leadership's day-to-day operating console and every member's "clan passport": the place that answers who someone is, what rank they hold, what they've trained in and what they've earned.

## Target Users

### Primary User Segment: Clan Leadership (Warlords, Generals, Officers)

**Profile**
- A small group of volunteers, likely a few dozen people at most, holding the top three Warframe clan ranks: **Warlord** (including the Founding Warlord), **General** and **Officer**.
- Gaming hobbyists in many time zones, running the clan in their spare time and mostly working from Discord on desktop or mobile.
- Varying technical skill. The tool has to be usable without training.

**Current behaviours and workflows**
- Recruit and onboard members, then promote and demote them in-game and separately in Discord.
- Run trainings and track outcomes informally.
- Discuss member conduct in private channels or DMs.
- Handle "who is this person?" questions by searching Discord or asking around.

**Needs and pain points**
- A quick way to look up any member and see their Discord identity, Warframe handle, rank, trainings, badges and (where permitted) PIF history.
- Confidence that sensitive notes are visible only to the right ranks.
- Rank changes that update Discord automatically, without separate manual steps.
- Fast bulk-friendly workflows. At ~1,000 members, one-at-a-time screens don't scale.

**Goals**
- Spend less time on bookkeeping and more on running the clan.
- Make consistent, fair, defensible decisions about promotions, discipline and recognition.
- Keep the clan's records intact when leaders rotate out.

**Permissions by rank**

| Capability | Officer | General | Warlord |
|---|---|---|---|
| View roster and member profiles | ✅ | ✅ | ✅ |
| Add PIF notes | ✅ | ✅ | ✅ |
| View full PIF history | ❌ | ✅ | ✅ |
| Create or edit trainings and badges | ❌ | ✅ | ✅ |
| Assign trainings and record outcomes | ✅ | ✅ | ✅ |
| Award badges (to others only) | ✅ | ✅ | ✅ |
| Change member ranks | TBD | TBD | ✅ |

### Secondary User Segment: Clan Members (Leader, Sage, Soldier, Initiate)

**Profile**
- The bulk of the ~1,000 members. They range from brand-new Initiates to long-standing veterans.
- Engagement varies widely. Many will sign in rarely, and a core group will use it actively.
- Discord is their main link to the clan outside the game.

**Current behaviours and workflows**
- Join through Discord or in-game invites.
- Learn about trainings and recognition through Discord announcements and word of mouth.
- Have no personal view of their own standing or progress in the clan.

**Needs and pain points**
- An easy, password-free sign-in using the Discord account they already have.
- One place to see their rank, completed trainings, earned badges and what's available next.
- A way to show who they are through their personal bio.

**Goals**
- Advance in the clan and be recognised for their contributions.
- Know clearly what's expected to earn the next badge or pass a required training.
- Show their achievements.

**What members can do**
- Edit their own **bio**.
- Choose badges they're **working toward**. This is a "goal" marker only and doesn't award anything.
- **Assign themselves** trainings.
- See their own rank, training statuses and earned badges. They can never see PIF notes about themselves or anyone else.

*Note: Warframe's **Utility** rank, typically used for alt accounts and bots, is assumed to have member-level or no access. To be confirmed.*

## Goals & Success Metrics

### Business Objectives

- **Complete roster:** At least 90% of active clan members registered (signed in with Discord and Warframe handle entered) within **60 days of launch**.
- **Rank and Discord roles stay in sync:** 100% of rank changes made in the app show up in Discord within **5 minutes**, with **zero** mismatches found by the nightly reconciliation check after the first 30 days.
- **Single record for leadership notes:** **All** new PIF notes recorded in the app, not in Discord channels or DMs, from **launch day onward**.
- **Trainings and badges catalogue set up:** All of the clan's current trainings and badges defined in the app, and historical awards back-filled where known, within **30 days of launch**.
- **Zero-cost operation:** **$0/month** hosting cost, staying within the Cloudflare and Supabase free-tier limits, for the first **12 months**.

### User Success Metrics

- **Fast member lookup:** A leader can find any member and open their full profile in **under 10 seconds**.
- **Quick onboarding:** A new member goes from landing page to a complete profile in **under 2 minutes**.
- **Fast rank change:** A rank change, including the Discord role update, takes **one action** and **under 30 seconds**, compared with today's separate in-game and Discord steps.
- **Members can see their progress:** Members can see their rank, trainings (with status), earned badges and badge goals on **one page**.
- **Notes visible only to the right ranks:** **Zero** cases of PIF note content being seen by anyone without permission.

### Key Performance Indicators (KPIs)

- **Registration coverage:** Registered members ÷ in-game clan roster size. **Target: ≥90% by day 60.**
- **Monthly active leaders:** Officers and above who sign in at least once a month. **Target: ≥80% of leadership.**
- **Monthly active members:** Non-leadership members who sign in at least once a month. **Target: ≥25%.** This is a soft target, because many members will only check in occasionally.
- **Training throughput:** Training assignments that reach a *Completed* status (Passed or Failed) each month. Tracked as a trend, with no target until a baseline exists.
- **Badges awarded:** Badge awards each month, plus the share of member "working toward" badge goals that end up awarded. This is a trend metric that shows engagement.
- **Rank sync failure rate:** Failed Discord role updates ÷ total rank changes. **Target: <1%**, and every failure is shown to admins and can be retried.
- **Free-tier headroom:** Peak usage as a percentage of each free-tier quota (Worker requests per day, Supabase database size and egress). **Target: stay under 50%** so there's room to grow.

## MVP Scope

### Core Features (Must Have)

- **Public landing page:** A static Cloudflare Pages site with clan branding, a short description and a "Sign in with Discord" button. It's the only page visible to the public.
- **Discord OAuth sign-in with a server membership check:** Sign-in uses Discord OAuth2. Only members of the clan's Discord server can sign in. A member record is created automatically on first sign-in from the Discord ID, username and avatar. Sessions are held securely by the Worker. *Rationale: no passwords, and the membership check stops outsiders from creating records.*
- **Member profile:** Discord details (ID, username, display name, avatar), Warframe handle, rank, clan role(s), personal bio, join date and status (Active, Inactive or Left). Members edit their own **bio** and **Warframe handle**. Everything else is edited by leadership.
- **Rank management:** Warframe's built-in ranks (Warlord, General, Officer, Leader, Sage, Soldier, Initiate, plus Utility). Authorised leaders change ranks, and each change is logged with who made it, when, and the old and new values.
- **Rank → Discord role sync:** Each rank maps to a Discord role. A rank change triggers a bot API call that updates the member's Discord roles. Failed syncs are shown to admins and can be retried. *Rationale: an explicit requirement, and the main fix for rank drift.*
- **Roster and search:** A list of all members that pages through results, can be searched by Discord name or Warframe handle, and can be filtered by rank, role, status, training status and badge. Leadership only.
- **PIF notes:** Officers and above add notes to any member's file except their own. Generals and Warlords view the full history. Notes are append-only: they can't be edited or deleted once saved, so the record can be trusted. Each note is stamped with its author and time.
- **Training catalogue and assignments:** Generals and Warlords create and edit trainings (name, description, active or retired). Trainings are assigned by leadership or self-assigned by members. Each assignment has a status of **Assigned**, **Completed (Passed)** or **Completed (Failed)**. Only Officers and above set completion statuses, and never on their own assignments. Each status change is logged.
- **Badge catalogue and awards:** Generals and Warlords create and edit badges (name, description, icon, active or retired). Officers and above award badges to other members, never themselves. Members can't award badges. Each award records who gave it, when and an optional reason.
- **Badge goals:** Members mark badges they're "working toward". These are shown on their profile and visible to leadership when deciding awards.
- **Member dashboard:** A single "My Profile" page showing a member's rank, roles, bio, trainings with status, earned badges and badge goals.
- **Authorisation enforced on the server:** Every permission rule (rank-based access, no self-awarding, PIF visibility) is enforced in the Worker API. The UI only hides controls a user can't use.
- **Audit log:** A record of every privileged action (rank changes, notes added, training status changes, badge awards, changes to the training and badge catalogues). Generals and Warlords can view it.

### Out of Scope for MVP

- Two-way Discord sync (Discord role changes updating the app); MVP sync is app → Discord only
- Any connection to Warframe game data (no public clan API exists)
- Bulk CSV or spreadsheet import of historical records (MVP back-filling is manual)
- Bulk actions (e.g. assign a training to 50 members at once); these are first in line for Phase 2
- Discord bot slash commands (e.g. `/profile @user`) and notifications or DMs from the bot
- Editing or deleting PIF notes, member-visible notes, and note categories or severities
- Expiring or recurring trainings (e.g. annual refreshers), training prerequisites and badge tiers
- Public member profiles or a public clan leaderboard
- Multi-clan or alliance support
- Native mobile app (mobile-responsive web only)
- Reporting dashboards and analytics beyond simple counts
- Custom rank names or a custom rank hierarchy beyond Warframe's built-in ranks

### MVP Success Criteria

The MVP is successful when:

1. Any member of the clan's Discord server can sign in with Discord, complete their profile and see their rank, trainings and badges, without help from leadership.
2. Leadership can find any member, change their rank and see the matching Discord role update **without touching Discord directly**.
3. Every permission rule in the permissions table holds, verified by automated tests. In particular, PIF history is visible only to Generals and Warlords, and nobody can award themselves a badge or mark their own training complete.
4. Generals and Warlords have set up the clan's existing trainings and badges, and leadership has stopped tracking them elsewhere.
5. The system has run for **30 days** on free tiers at **$0 cost** with no free-tier limits exceeded.

## Post-MVP Vision

### Phase 2 Features

Priorities for scaling leadership work and deepening the Discord connection:

- **Bulk actions:** Assign a training, award a badge or change status for many members at once, chosen through roster filters.
- **Discord bot commands and notifications:** Slash commands such as `/profile @user`, `/mybadges` and `/trainings`, plus optional DMs or channel posts when a badge is awarded, a training is assigned or a result is recorded.
- **Two-way rank reconciliation:** A scheduled job compares Discord roles with app ranks and flags mismatches for leadership to resolve. Real-time gateway sync could follow if a free hosting option allows it.
- **Data import and export:** CSV import to back-fill historical rosters, trainings and badges, and CSV export for backups and offline review.
- **PIF note improvements:** Note categories or severity (e.g. Commendation, Informal Warning, Formal Warning), "amends" notes that link to the note they correct, and an optional member-visible flag for formal warnings.
- **Training improvements:** Prerequisites (Training B needs Training A), expiry and recertification dates, assigned trainers, and scheduled sessions.
- **Badge improvements:** Badge tiers (Bronze/Silver/Gold), linking badges to trainings so a passed training can auto-suggest a badge, and badge requirement checklists.
- **Promotion readiness:** Configurable criteria per rank (e.g. "Soldier → Sage needs Trainings X and Y and 30 days at the current rank"), with a list of members who qualify.
- **Inactivity management:** Flag members who haven't signed in or been active for N days, to help with roster cleanup as the clan nears its 1,000-member cap.

### Long-term Vision

Over 1–2 years, the tool becomes the clan's **operating system**: leadership runs recruitment, onboarding, training, recognition, promotion and discipline in one place, and Discord is the conversation layer on top. Every member has a living **"clan passport"** showing their history, achievements and path to the next rank. Leadership handovers become painless because the clan's institutional memory is in the system rather than in individual people.

### Expansion Opportunities

- **Alliance support:** Warframe alliances link several clans. The tool could support several clans with their own leadership and a shared alliance-level view, e.g. for sister clans or overflow clans once the 1,000-member cap is reached.
- **Recruitment pipeline:** Track applicants from first contact through trial period to full membership, with recruiter assignment and interview notes.
- **Events and activities:** Schedule clan events, record attendance, and feed attendance into badge or promotion criteria.
- **Public clan showcase:** An optional public page of clan achievements, a leaderboard of top badge holders, and recruitment info to attract new members.
- **Open-source template:** Package it as a free, self-hostable clan admin kit that other Warframe (or other game) clans can deploy to their own free-tier Cloudflare and Supabase accounts.

## Technical Considerations

*These are starting points for the Architect, not final decisions. Free-tier limits change often, so check them against current Cloudflare and Supabase documentation before committing.*

### Platform Requirements

- **Target Platforms:** A responsive web app for desktop and mobile browsers. Many members will open it from links in Discord on their phones.
- **Browser/OS Support:** Current versions of the main browsers (Chrome, Edge, Firefox, Safari on desktop and iOS, Chrome on Android). No support for legacy browsers such as IE.
- **Performance Requirements:**
  - Landing page loads in under 1.5s on broadband, which is easy for static content served from Cloudflare's edge.
  - Authenticated pages show content in under 2s. 95% of API responses take under 500ms, including the database round-trip.
  - Roster search over ~1,000 members returns in under 500ms.
  - Worker code has to stay inside the **free tier's 10ms CPU limit per request**. Time spent waiting on the database or Discord doesn't count towards it, so typical CRUD work fits easily, but heavy processing inside the Worker is ruled out.

### Technology Preferences

- **Frontend:** A static single-page app (e.g. **React + Vite** or **SvelteKit** in static mode) built and deployed as static files. Use a lightweight component library to keep it fast. *Framework still to be chosen; I recommend React + Vite because it has the widest pool of examples and contributors.*
- **Backend:** **Cloudflare Workers** written in **TypeScript**, with a lightweight router such as **Hono**, which is built for Workers. All business logic and permission checks live here.
- **Database:** **Supabase Postgres (free tier)**. The Worker connects through **Cloudflare Hyperdrive** (connection pooling and caching) or Supabase's built-in pooler, using a Postgres driver that runs on Workers (e.g. `postgres.js` with `nodejs_compat`). Schema changes are managed as versioned SQL migrations (Supabase CLI). **Supabase Auth isn't used**, because Discord sign-in is handled by the Worker.
- **Hosting/Infrastructure:**
  - **Cloudflare Workers with static assets** (or Cloudflare Pages) serve the landing page, the SPA and the API from **one domain**. A single origin avoids CORS and third-party cookie problems with sessions.
  - **Cloudflare Cron Triggers** (free tier) run scheduled jobs: retrying failed Discord syncs, a nightly reconciliation report, and a keep-alive query so Supabase doesn't pause the project.
  - Secrets (Discord client secret, bot token, database credentials, session signing key) are stored as **Worker secrets**, never in the repo or the browser.
  - CI/CD runs on **GitHub Actions** (free for public repos, with generous minutes for private ones), deploying with Wrangler.

### Architecture Considerations

- **Repository Structure:** A single **monorepo** containing `apps/web` (the SPA and landing page), `apps/api` (the Worker), `packages/shared` (shared TypeScript types, the rank list and permission rules) and `supabase/migrations`. Keeping the permission rules in a shared package means the UI and the API can't disagree about them.
- **Service Architecture:**
  - One Worker handles both the API and the static assets. There are no separate microservices.
  - Discord role sync is a call made from inside the Worker. It's recorded in a sync log or outbox table, so failures are visible and can be retried from cron.
  - Sessions are kept in a **secure, HttpOnly, SameSite cookie**, either backed by a Postgres sessions table or as a signed, short-lived token. *Avoid Workers KV for session writes: the free tier allows only ~1,000 KV writes a day.*
- **Integration Requirements:**
  - **Discord OAuth2** with scopes `identify` and `guilds.members.read`, to read the user's identity and check that they belong to the clan server.
  - **Discord Bot (REST only):** A bot application with the *Manage Roles* permission. Its role has to sit higher in the server's role list than every rank role it manages. It's used to add and remove rank roles. It doesn't need a gateway connection in the MVP.
  - No integration with Warframe or Digital Extremes, since there's no public clan API.
- **Security/Compliance:**
  - **Every permission rule is enforced on the server** in the Worker, using one permission module covered by unit tests: rank-based access, PIF visibility, no self-awarding badges, and no self-certifying trainings.
  - **Lock down Supabase's auto-generated API:** Supabase exposes tables publicly through its PostgREST API using the project's anon key. **Enable Row-Level Security with no policies on every table, or keep tables in a schema that isn't exposed.** All access then has to go through the Worker using server-side credentials only.
  - **PIF notes are sensitive personal data.** Use the minimum privilege and avoid exposing notes in logs or error messages. Consider column-level encryption as a later improvement.
  - **Personal data is minimal:** Discord ID, username, avatar, a self-reported handle and a bio. Members are in many countries, so give the clan a simple way to delete a member's personal data on request. Consider a short privacy notice on the landing page.
  - **Basic web security:** CSRF protection (SameSite cookies plus checking the Origin header), a `state` parameter on the OAuth flow, rate limiting on sign-in, and a Content Security Policy.
  - **Backups:** Supabase's free tier **doesn't include managed backups**. Plan a scheduled logical export (e.g. a GitHub Action running `pg_dump` to private storage) so leadership history can't be lost.

## Constraints & Assumptions

### Constraints

- **Budget:** **$0 a month** in running costs. Every hosted part (Cloudflare Workers, Pages and static assets, Hyperdrive, Cron Triggers; Supabase Postgres; GitHub and GitHub Actions) has to run on its free tier. The only cost that may be allowed is a one-off custom domain (~$10–15/year), which is optional because a free `*.workers.dev` / `*.pages.dev` subdomain works.
- **Timeline:** No fixed deadline. It's a volunteer project, developed in increments. *Suggested target:* MVP live within **8–12 weeks** of starting development, with the Phase 1 rollout to leadership before opening it to all members.
- **Resources:**
  - Development by a small volunteer team, probably one main developer (the project owner) working with AI-assisted development.
  - Leadership volunteers available for user acceptance testing (UAT) and for setting up the initial training and badge catalogues.
  - A Discord server admin with permission to create the bot and arrange role order.
- **Technical:**
  - **Workers free tier:** ~100,000 requests a day and **10ms CPU time per request**. There are no long-running processes or persistent connections, so the Discord gateway isn't available.
  - **Supabase free tier:** ~500MB database. The **project pauses after about 7 days of inactivity**. There are **no managed backups** and only a limited number of free projects per account.
  - **Workers KV free tier:** ~1,000 writes a day. That's unsuitable for sessions or other frequently written data.
  - **Discord API:** Rate limits apply per route and globally. The bot can only manage roles **below** its own role in the server's role list.
  - **Warframe:** There's **no public API** for clan rosters or ranks. In-game rank changes still have to be made by hand in the game client.

### Key Assumptions

- Every clan member has a Discord account and is (or will become) a member of the clan's Discord server.
- The app becomes the **source of truth for clan rank**. Leadership changes a rank in the app first and then mirrors it in-game; Discord follows the app automatically.
- Each Warframe rank maps one-to-one to a Discord role.
- The leadership group (Warlords, Generals, Officers) is small (tens of people), and **Founding Warlord** is treated as Warlord for permissions.
- Leadership will move PIF note-taking and training and badge tracking out of Discord channels and spreadsheets and into the app at launch.
- Trainings and badges are managed by **Generals and Warlords only**. Officers award and assess but don't define.
- Members accept that their Discord ID, username and avatar are stored for clan administration.
- Expected load (~1,000 members, mostly occasional use, tens of daily active leaders) stays well within free-tier limits.
- The project owner has (or will create) accounts on Cloudflare, Supabase, GitHub and the Discord Developer Portal, with administrative access to the clan's Discord server.
- The clan is happy to start with a `*.workers.dev` / `*.pages.dev` URL if no custom domain is bought.

## Risks & Open Questions

### Key Risks

- **Supabase public API exposure:** If Row-Level Security isn't enabled on every table, anyone with the project's anon key could read PIF notes through Supabase's auto-generated REST API, bypassing the Worker entirely. *Impact: critical (sensitive notes leaked).* *Mitigation:* RLS deny-all on every table (or tables kept in a schema the API doesn't expose), an automated test that checks this, and the anon key never shipped to the browser.
- **In-game rank drift:** The app can't change Warframe ranks. If leaders update the app but forget the game client (or the other way round), the records disagree. *Impact: medium; erodes trust in the roster.* *Mitigation:* a clear "app first, then in-game" leadership procedure, a "Mirrored in-game ✔" checkbox on rank changes, and a periodic manual audit.
- **Discord role sync failures:** Bot misconfiguration (role order, missing permission), rate limits, or members who have left the server can make role updates fail. *Impact: medium.* *Mitigation:* an outbox with retries via cron, failures shown to admins, and a setup checklist for bot permissions and role order.
- **Supabase free-tier pause and data loss:** The project pauses after ~7 days without activity, and the free tier has no managed backups. *Impact: high if data is lost; low for an outage.* *Mitigation:* a cron keep-alive query and a scheduled `pg_dump` backup to private storage.
- **Free-tier limits change:** Cloudflare or Supabase could tighten free-tier limits or terms. *Impact: medium.* *Mitigation:* keep the code portable (standard Postgres, a thin Workers adapter), watch the free-tier headroom KPI, and document a fallback plan for a low-cost paid tier.
- **Permission logic bugs:** The rules are detailed (rank tiers, no self-awarding, no self-certifying, PIF visibility), and a bug could leak notes or allow privilege escalation. *Impact: high.* *Mitigation:* one shared permission module, an exhaustive unit test matrix (every rank × every action, including self-targeting), and QA review of every story that touches permissions.
- **Leadership account compromise:** A hijacked General or Warlord Discord account would get full PIF access. *Impact: high.* *Mitigation:* short session lifetimes, re-checking rank from the database on every request, the audit log, and encouraging leadership to turn on Discord 2FA. Consider requiring 2FA (Discord's `mfa_enabled` flag) for leadership ranks.
- **Low member adoption:** Members may not bother to sign in and fill in their Warframe handle. *Impact: medium; incomplete roster.* *Mitigation:* links to sign-in from Discord announcements, leadership asking members to sign in at promotion time, and badges and trainings as incentives.
- **Bus factor of one:** A single volunteer developer and owner. If they leave, the tool may go unmaintained, and cloud accounts and secrets may become inaccessible. *Impact: high over the long term.* *Mitigation:* clear docs, at least two leaders with admin access to the Cloudflare, Supabase, GitHub and Discord accounts, and secrets documented in a shared vault.
- **Privacy and moderation concerns:** PIF notes are subjective records about real people across many countries. *Impact: reputational, or a dispute within the clan.* *Mitigation:* a privacy notice, append-only notes, the audit trail, and a process for deleting a departed member's data.

### Open Questions — Answers Needed

> **How to use this list:** Take it to clan leadership, write each answer in the **Answer** line, and bring it back to the Analyst session. **Current assumption** is what the brief uses until you answer, so a blank answer means "keep the assumption". **🔴 Blocks PRD** marks questions that set the permission matrix and have to be answered before PM work starts.

#### A. Permissions and governance

**Q1. Rank change permissions** 🔴 Blocks PRD
Who can change a member's rank, and up to which rank? Who can appoint or remove Generals and Warlords?
- *Current assumption:* Only Warlords can change ranks. *Suggested pattern:* a leader can promote or demote up to one rank below their own (e.g. a General can set ranks up to Officer).
- **Answer:** ______

**Q2. Founding Warlord powers** 🔴 Blocks PRD
Does the Founding Warlord have powers beyond a regular Warlord's (e.g. the only one who can appoint or remove Warlords, or transfer ownership of the app)?
- *Current assumption:* Founding Warlord = Warlord for every permission.
- **Answer:** ______

**Q3. What Officers can see of PIF notes** 🔴 Blocks PRD
Officers can add PIF notes. Can they read **the notes they themselves wrote**, the **most recent note(s)**, or nothing at all (write-only)?
- *Current assumption:* Write-only. Only Generals and Warlords see any PIF history.
- **Answer:** ______

**Q4. Recording training outcomes** 🔴 Blocks PRD
(a) Can Officers mark trainings Completed (Passed/Failed), or only Generals and above? (b) Should leaders be blocked from marking their **own** trainings complete?
- *Current assumption:* (a) Officers and above can. (b) Yes, nobody can certify their own training.
- **Answer:** ______

**Q5. Self-assigning trainings** 🔴 Blocks PRD
When members self-assign a training, can they only set it to *Assigned*, and is setting *Passed/Failed* reserved for leadership?
- *Current assumption:* Yes. Members can self-assign only, and leadership records results.
- **Answer:** ______

**Q6. PIF notes on yourself**
Should leaders be blocked from adding PIF notes to their **own** file?
- *Current assumption:* Yes, blocked (matches the no-self-award rule).
- **Answer:** ______

**Q7. Editing PIF notes**
Should PIF notes be append-only (no edits or deletes; corrections are made by adding a follow-up note)?
- *Current assumption:* Append-only.
- **Answer:** ______

**Q8. Member-visible notes**
Should members ever see PIF entries about themselves (e.g. formal warnings)?
- *Current assumption:* Never in the MVP. A "member-visible" flag is considered for Phase 2.
- **Answer:** ______

#### B. Membership and sign-in

**Q9. Sign-in gate**
Is being a member of the clan's Discord server enough to sign in and get a record created? Or should new sign-ins wait in a **Pending** state until leadership approves them? Does your server have guests or visitors who aren't clan members?
- *Current assumption:* Server membership is enough. Records are created automatically.
- **Answer:** ______

**Q10. Utility rank**
How should Warframe's **Utility** rank (often alt or bot accounts) be treated: normal member access, no access, or excluded from the roster?
- *Current assumption:* Member-level access. To be confirmed.
- **Answer:** ______

**Q11. Departed members**
When someone leaves the clan, do we keep their record (marked *Left*, useful if they rejoin) or delete it? After how long, if at all?
- *Current assumption:* Keep, marked *Left*. Personal data deleted on request.
- **Answer:** ______

**Q12. Editing the Warframe handle**
Can members edit their own Warframe handle, or should leadership control it?
- *Current assumption:* Members can edit it themselves (handles change).
- **Answer:** ______

**Q13. What "active member" means**
How does leadership decide whether a member is "active"? This feeds the 90% registration goal.
- *Current assumption:* Leadership's judgement (e.g. seen in game or Discord within 30 days).
- **Answer:** ______

#### C. Ranks, roles and Discord

**Q14. Which way Discord sync goes**
Confirm the app is the source of truth, so rank changes in the app push to Discord roles. What should happen if someone changes a rank role **directly in Discord**?
- *Current assumption:* One-way, app → Discord. Changes made directly in Discord get flagged by a nightly reconciliation (Phase 2) and aren't overwritten automatically.
- **Answer:** ______

**Q15. Rank names and Discord role mapping**
Does your clan use Warframe's default rank names or custom ones? List each rank and the Discord role it maps to.
- *Current assumption:* Default names (Warlord, General, Officer, Leader, Sage, Soldier, Initiate, Utility), each mapped to one Discord role.
- **Answer:** ______

**Q16. Clan roles**
What clan roles exist (e.g. Recruiter, Trainer, Event Lead)? Are they different from rank? Should they also sync to Discord roles?
- *Current assumption:* Roles are separate labels with no permissions attached, managed by Generals and Warlords, and not synced to Discord in the MVP.
- **Answer:** ______

#### D. Context, logistics and technology

**Q17. Current tooling and data**
What does leadership use today (spreadsheets, a Discord bot, private channels)? Is there existing data to migrate (roster, notes, training and badge history)?
- *Current assumption:* Informal tools, with manual back-filling after launch.
- **Answer:** ______

**Q18. Pain-point evidence (optional)**
Any rough numbers? For example, hours a week spent on admin, number of rank or role mismatches, or how often notes get lost. These strengthen the Problem Statement and give a baseline for the metrics.
- **Answer:** ______

**Q19. Size of leadership**
About how many Warlords, Generals and Officers are there?
- *Current assumption:* Tens of people.
- **Answer:** ______

**Q20. Timeline**
Is there a target launch date or event?
- *Current assumption:* MVP within 8–12 weeks of starting development. Rollout to leadership first, then to all members.
- **Answer:** ______

**Q21. Frontend framework**
Do you or your contributors prefer React, Svelte or something else?
- *Current assumption:* React + Vite (recommended).
- **Answer:** ______

**Q22. Custom domain**
A clan domain (~$10–15/yr, e.g. `admin.yourclan.gg`) or a free `*.workers.dev` subdomain?
- *Current assumption:* Free subdomain to start.
- **Answer:** ______

**Q23. 2FA for leadership**
Should Officers and above be required to have Discord two-factor authentication turned on to use the app?
- *Current assumption:* Recommended, not decided yet.
- **Answer:** ______

**Q24. Account ownership (bus factor)**
Which two (or more) leaders will have admin access to the Cloudflare, Supabase, GitHub and Discord developer accounts?
- **Answer:** ______

### Areas Needing Further Research

- Current free-tier limits and terms for Cloudflare Workers, Hyperdrive, Cron Triggers and static assets, and for Supabase (database size, egress, pause policy, number of projects).
- Hyperdrive vs Supabase's built-in pooler (Supavisor) from Workers: latency, connection limits and free-tier fit.
- Discord OAuth2 scopes and bot permissions: `guilds.members.read` behaviour, role hierarchy rules, and rate limits for role add/remove endpoints at bulk scale.
- Checking whether a user has 2FA via Discord OAuth (`mfa_enabled`) as an option for enforcing 2FA on leadership.
- Free backup options for Supabase on the free tier (GitHub Actions + `pg_dump` + encrypted storage target).
- Warframe clan rank structure and permissions in-game, to confirm the rank list and any custom rank names your clan uses.
- Existing open-source clan or guild management tools (Warframe or other games), for UX patterns and lessons learned.
