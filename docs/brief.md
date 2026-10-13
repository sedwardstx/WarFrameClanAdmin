# Project Brief: WarFrame Clan Admin

> **📌 Status: v1.0 COMPLETE, ready for Founder review and PM handoff (2026-10-13)**
> - The Founder's input and every decision are recorded in **Appendix B: Stakeholder Input**.
> - **Before the PRD:** answer Open Question 1 (whether to hold back app permissions after rule-breaking promotions made in Discord).

## Executive Summary

**WarFrame Clan Admin** is a web-based **leadership operations tool** for a new Warframe clan, built to scale to Warframe's 1,000-member clan cap. The clan's Discord server is its home and the **source of truth for roles**. The tool syncs with it **in both directions** and adds what Discord can't provide on its own: a structured record of every member, and the leadership processes that need privacy, routing and an audit trail.

**The problem:** The clan runs on a deliberate structure. **Command ranks** are active responsibilities, not rewards. On top of rank sit **modular roles**: departments, specialists and moderators. New recruits go through a **probation trial**. Run from Discord channels alone, this structure has no single member record, no access control for sensitive leadership notes, no evidence base for trial and promotion decisions, and no reliable way to route reports and requests up the chain of command. As a **new clan**, it can start with the right system from day one instead of cleaning up later.

**Target users:**
- **Command ranks** (Grand Duke/Founder, Archdukes, Dukes, Viceroys), who run the clan.
- **Members** (Quartermaster, Stablemaster, Serf, Peon), who keep their profile current and see their own standing.

**Value proposition:**
- **One roster, synced with Discord:** Discord identity, Warframe handle, rank, departments, specialist and moderator roles, and badges for every member. Records are created when a rank role is assigned in Discord, and changes flow both ways.
- **Confidential PIF notes:** Notes use templates and are append-only. Visibility is tiered: every officer can write notes, only the Grand Duke and Archdukes can read them, and only for the right ranks.
- **Peon trial tracking:** Discord activity signals and officer sightings feed a 14-day trial. When a candidate is ready for review, **#recruitment-board** is alerted. Outcomes are recorded as Approve, Extend or Deny.
- **Chain-of-command workflows:** Promotion recommendations, department status reports (Green/Yellow/Red) and administrative requests are routed automatically to the right supervisor.
- **Accountability and security:** Every privileged action is audited, rule-breaking changes made directly in Discord are flagged to **#server-room**, and Discord 2FA is required for every command rank.
- **Zero hosting cost:** Everything runs on free tiers: Cloudflare Workers and Pages, with Supabase Postgres.

## Problem Statement

**Current state**

This is a **new clan**. Its structure is more sophisticated than most:
- **Eight ranks**, five of them command ranks.
- **Command ranks are active responsibilities**, not permanent rewards. They can be reassigned without that being disciplinary.
- **Modular roles** sit on top of rank: five departments, a growing set of specialist qualifications, and a separate moderator duty. A member can hold several at once.
- **New recruits (Peons) serve a ~14-day probation**, judged on what they actually do, not on their title.
- **A chain of command** expects weekly department reports and escalations to flow upward.

At the moment, all of this exists only as **Discord roles plus channel conventions** (#leadership-reports, #candidate-review, #command-center and so on). That's fine for conversation, but it can't hold the clan's records or enforce how the processes should run.

**Pain points (expected, and avoidable if we build now)**

- **No single member record.** Discord shows a member's roles, but not their Warframe handle, their history in the clan, or why they hold or lost a role. "Who is this person in-game?" has no reliable answer.
- **Sensitive notes have nowhere safe to live.** Conduct observations and leadership feedback (PIF notes) would end up in channels or DMs, visible to anyone with channel access, editable or deletable, and lost when officers leave. The clan needs **tiered confidentiality**: every officer can write notes, only the Grand Duke and Archdukes can read them, and Archdukes can't read notes about peers or superiors. Discord can't do that.
- **Trials judged by impression, not evidence.** The trial is meant to measure what recruits *actually do*. Without tracking, reviewers rely on memory and who happened to notice whom. Trials also run past 14 days unnoticed, because nothing signals that a candidate is due for review.
- **Promotions and reassignments without a paper trail.** The policy depends on reliability and follow-through over time. Without a recorded history of recommendations, decisions and reasons, promotions look arbitrary, and a non-disciplinary reassignment is easily mistaken for a demotion.
- **Reports and requests get lost in channel scrollback.** Weekly Green/Yellow/Red department reports and officer escalations (approvals, member or mod issues, role adjustments) are posted as messages. There's no record of who owns each item, whether it's been reviewed, or whether it's resolved.
- **Role changes with no oversight.** Anyone with Discord's *Manage Roles* permission can change a rank directly, bypassing the clan's promotion rules, and nobody notices.

**Impact**

- Leadership time goes into reconstructing context instead of leading. As the clan grows towards 1,000 members, this gets worse with every recruit.
- Decisions on trials, promotions and reassignments become harder to justify, which hurts trust in the "active responsibility, not reward" culture the clan is built on.
- Sensitive notes either leak or never get written down, and both are bad for a clan that wants accountable leadership.

**Why existing solutions fall short**

- **Discord alone:** Excellent at roles and conversation, but it isn't a records system. Access control is per channel, not per member or per rank, history is unstructured, and there's no routing or status tracking.
- **Warframe's in-game clan tools:** Only ranks and basic permissions. No departments, notes, trials, workflows or Discord link, and no public API.
- **Spreadsheets and docs:** No row-level privacy (a PIF sheet is all-or-nothing), no connection to Discord roles, no workflows, and no self-service for members.
- **General-purpose Discord bots** (moderation, levelling, ticketing): Each solves part of the problem, but none models this clan's rank ladder, modular roles, tiered note confidentiality or trial process. Stitching several together creates new gaps.

**Why now**

The clan is **just starting**. Building the system of record **before** the roster grows avoids migrating or rebuilding history later. Every member, trial, note and promotion is captured from day one, and leadership habits form around the tool rather than around channel workarounds.

## Proposed Solution

**Core concept and approach**

A purpose-built clan operations web app that is **tightly coupled to the clan's Discord server**. It has four parts:

1. **Public landing page** (static, on Cloudflare): clan identity, a privacy notice, and a single **"Sign in with Discord"** button. No passwords are stored anywhere.
2. **Application API** (Cloudflare Worker): Discord OAuth sign-in, sessions, and **every business rule**. That covers rank-change limits, PIF visibility, routing to supervisors, trial decisions and the ban on awarding yourself a badge. Permissions are always checked on the server.
3. **Sync and activity engine** (scheduled Cloudflare Worker, running every few minutes): polls Discord to **sync roles in both directions**, create member records when a rank role appears, log changes made directly in Discord, collect activity signals for Peons on trial, and post alerts to Discord channels.
4. **Clan database** (Supabase Postgres, free tier): members, role assignments, notes, workflows, badges, activity snapshots and a complete audit log.

**Discord and the app: who owns what**

| Data | Source of truth | Direction |
|---|---|---|
| **Which roles exist** (ranks, departments, specialists, moderator) | **Discord.** The Grand Duke creates new role types there. | Discord → app |
| **Who holds which role** | **Shared.** It can be changed in either place. | App ↔ Discord (two-way) |
| **Member records, Warframe handle, bio, notes, badges, workflows, trial state** | **App** | App only (plus alerts posted to Discord) |

- A member record is **created automatically** when someone receives a rank role in Discord. Guests without a rank role never get a record and can't sign in.
- A role change made **in the app** is pushed to Discord straight away by the clan's own bot. The bot is given whatever Discord permissions two-way sync needs (Manage Roles, Server Members intent, View Audit Log, and so on).
- When the Grand Duke creates a **new role in Discord**, the app finds it on the next poll. The Grand Duke **classifies it** with a single click (Rank / Department / Specialist / Moderator / Ignore) before the app manages it.
- A role change made **in Discord** is pulled in on the next poll. If it breaks the clan's rules (e.g. someone changed a rank they aren't allowed to), it is **accepted, logged as "changed in Discord by X", and flagged in #server-room**.

**Core capabilities**

- **Roster and member profile:** Discord identity, Warframe handle, rank, departments (one or more), specialist roles, moderator status, badges, and rank history with reasons. Officers can search and filter it.
- **Rank management with a chain of command:** Leaders change ranks up to one rank below their own. **Dukes and above submit promotion recommendations**, which any higher rank can approve or deny. Every rank change records a **reason type** (promotion, reassignment, step-down, disciplinary), which follows the clan's "active responsibility, not reward" policy.
- **Modular role management:** Archdukes assign department, specialist and moderator roles **within their own department(s)**. The Grand Duke can assign any of them. Changes sync to Discord.
- **Confidential PIF notes with templates:** Every officer can **write** notes, using templates such as General, Behaviour Issue, Feedback and "Recently seen playing". Notes are **append-only**, and nobody can write a note about themselves. The **Grand Duke reads all notes**. **Archdukes read notes only about members ranked below Archduke.**
- **Peon trial tracking:** A ~14-day trial clock starts when the Peon role is assigned. The trial gathers **activity signals** from Discord (text activity, voice presence, online and game status where Discord makes them available) plus officers' "recently seen playing" notes. When the trial is due, **#recruitment-board is alerted**. The outcome is **Approve → Serf**, **Extend +7 days** or **Deny (remove)**. Trial decisions are made by **officers in the Recruitment department, Archdukes or the Grand Duke**. A Deny removes the Peon role and starts the standard member-removal flow, which an Archduke or above confirms.
- **Department status reports:** Archdukes (and the Grand Duke) file Green/Yellow/Red status reports for each of the five departments. Reports go to the filer's supervisor and are posted to **#leadership-reports**.
- **Administrative requests:** Any officer can raise one (approval needed, member or mod issue, role adjustment). It is **automatically routed to their immediate supervisor** (Viceroy → Duke → Archduke → Grand Duke, within the same department, skipping vacant positions) and tracked through its statuses (Open → Under review → Approved / Denied / Resolved). Each request is filed against **exactly one department**. If the officer belongs to only one department, it's filled in automatically. If they belong to several, **choosing a department is required**. **Every superior in that department's chain can see the request and act on it**, not only the immediate supervisor. The submitter can see the status.
- **Badges:** Archdukes and the Grand Duke create badges. Any officer can award them to *other* members. Members mark the badges they're working toward.
- **Member self-service:** Members edit their own bio and Warframe handle, and see their own rank, roles and badges. They never see PIF notes.
- **Member removal:** When someone loses their rank role or leaves the server, the app flags them for removal. **An Archduke or above confirms**, then the record and its PIF notes are deleted. The audit log keeps a minimal record of the removal.
- **Accountability and security:** Every privileged action is written to the audit log and to **#server-room**. **Discord 2FA is required** for all command ranks.

**Key differentiators**

- **Built around this clan's model:** Its eight ranks, modular roles, chain of command and trial policy are built into the product, not bolted on.
- **Discord stays home:** Leaders keep managing roles in Discord if they prefer. The app keeps up and catches anything that breaks the rules.
- **Confidentiality that Discord channels can't give:** PIF visibility is decided by rank and by who the note is about.
- **Decisions backed by evidence:** Trials and promotions draw on recorded activity, notes and history, not memory.
- **$0 to run:** Free tiers only, and no always-on server to maintain.

**Why this will succeed**

- **Little friction:** One-click Discord sign-in, and records appear without anyone filling in forms.
- **Fits existing habits:** It posts to the channels leadership already watches (#recruitment-board, #leadership-reports, #server-room).
- **Greenfield timing:** Adopted from day one, so there's no migration and no competing old system.
- **Scope fits a volunteer team:** It's one codebase with no always-on servers, built and maintained by one developer with AI-assisted delivery.

**High-level vision**

The clan's **command console**: the place where leadership recruits, evaluates, promotes, reassigns and keeps track of the clan, with Discord as the conversation layer and every member's record captured from their first day as a Peon.

## Target Users

### Primary User Segment: Command Ranks (Grand Duke, Archdukes, Dukes, Viceroys)

**Profile**
- **Tens of people** holding the five command ranks. They are volunteers, spread across time zones, running the clan in their spare time and mostly working from Discord on desktop or mobile.
- **Grand Duke (1):** the Founder, Micoflame, who has final authority.
- **Archdukes (several):** second in command and department directors. Each owns one or more of Recruitment, Operations, Events, Dojo and Community.
- **Dukes:** senior officers who run projects and junior leaders inside departments.
- **Viceroys:** junior officers, the first true command rank, who organise members and handle smaller responsibilities.
- Their technical skill varies, so the tool must be usable without training. **Discord 2FA is required for every command rank.**

**Current behaviours and workflows**
- Recruit through #recruitment-board and #candidate-review, then watch Peons informally during the trial.
- Discuss promotions and reassignments in #command-center and #think-tank, and post weekly department reports in #leadership-reports.
- Change ranks and roles directly in Discord.
- Raise issues, approvals and role-adjustment needs through ad-hoc messages.

**Needs and pain points**
- An instant member lookup: identity, Warframe handle, rank, departments, roles, history and (where permitted) PIF notes.
- **Evidence for trial decisions** (activity signals, sightings), and an alert when a trial is due.
- A structured, auditable path for **recommendations, reports and requests** that reaches the right supervisor and doesn't get lost.
- Confidence that **PIF notes are only visible to the right people**.
- Freedom to keep managing roles in Discord, with the app keeping up.

**Goals**
- Run the clan's structure well: the right people in active roles, trials judged fairly, and departments that keep moving.
- Make promotion and reassignment decisions that can be justified and are consistent with policy.
- Keep the clan's institutional memory safe as leaders rotate.

### Secondary User Segment: Members (Quartermaster, Stablemaster, Serf, Peon)

**Profile**
- Most of the clan, growing towards 1,000 members over time.
- **Quartermasters** are trusted specialists, e.g. Recruiter, Mentor, Dojo Architect, Event Host or Squad Leader. **Stablemasters** are established veterans. **Serfs** are full members in good standing. **Peons** are new recruits on a ~14-day trial.
- Members may also hold **department**, **specialist** or **moderator** roles that are separate from their rank.
- Engagement varies: a core group is very active, and many members check in only occasionally. Discord is their main link to the clan.

**Needs and pain points**
- Password-free sign-in with the Discord account they already use.
- One place to see their rank, roles, departments and badges.
- A way to keep their Warframe handle and bio up to date.
- **Peons:** clarity that the trial is under way and what's expected of them.

**Goals**
- Pass the trial, earn recognition, and grow into specialist or command roles.
- Browse the **clan directory** to find other members' handles, ranks and badges.
- Show their achievements (badges) and what they're working toward.

### Permissions Matrix

*Legend: ✅ allowed · ❌ not allowed · 🔸 allowed with limits (see notes). "Members" means Quartermaster, Stablemaster, Serf and Peon.*

| Capability | Grand Duke | Archduke | Duke | Viceroy | Members |
|---|---|---|---|---|---|
| Sign in (requires a rank role in Discord) | ✅ 2FA | ✅ 2FA | ✅ 2FA | ✅ 2FA | ✅ |
| Edit own bio and Warframe handle; set badge goals | ✅ | ✅ | ✅ | ✅ | ✅ |
| View own profile (rank, roles, badges) | ✅ | ✅ | ✅ | ✅ | ✅ |
| View clan directory (basic profiles: Discord name and avatar, Warframe handle, rank, badges; never PIF) | ✅ | ✅ | ✅ | ✅ | ✅ |
| View full roster and member profiles (departments, roles, history, admin filters) | ✅ | ✅ | ✅ | ✅ | ❌ |
| Write PIF notes (never about yourself) | ✅ | ✅ | ✅ | ✅ | 🔸 Recruiter specialists only |
| Read PIF notes | ✅ **every note** | 🔸 only about ranks below Archduke | ❌ | ❌ | ❌ |
| Submit promotion recommendations | — (promotes directly) | 🔸 one step, up to Duke | 🔸 one step, up to Viceroy | ❌ | ❌ |
| See and approve or deny promotion recommendations | ✅ **all** | 🔸 own department(s) | ❌ | ❌ | ❌ |
| Change ranks directly | 🔸 up to Archduke | 🔸 up to Duke | ❌ | ❌ | ❌ |
| Assign department, specialist and moderator roles | ✅ all | 🔸 own department(s) | ❌ | ❌ | ❌ |
| Classify new roles discovered in Discord | ✅ | ❌ | ❌ | ❌ | ❌ |
| Create and edit badges | ✅ | ✅ | ❌ | ❌ | ❌ |
| Award badges | ✅ including to self | 🔸 others only | 🔸 others only | 🔸 others only | ❌ |
| Decide trial outcomes (Approve, Extend, Deny) | ✅ | ✅ | 🔸 Recruitment dept | ❌ | ❌ |
| File department status reports | ✅ | 🔸 own department(s) | ❌ | ❌ | ❌ |
| File administrative requests | — (top of chain) | ✅ | ✅ | ✅ | ❌ |
| View and act on admin requests | ✅ all | 🔸 own department(s) | 🔸 own department(s), from lower ranks | ❌ | ❌ |
| Confirm member removal | ✅ | ✅ | ❌ | ❌ | ❌ |
| View audit log | ✅ | ✅ | ❌ | ❌ | ❌ |

**Notes on the matrix**
- **Rank changes:** A leader can set ranks up to one below their own, so the Grand Duke can set up to Archduke and an Archduke up to Duke. Rank changes made directly in Discord that break this rule are accepted, logged and flagged in #server-room.
- **Promotion recommendations:**
  - **One step at a time.** A recommendation moves a member up exactly **one rank** (e.g. Serf → Stablemaster, never Serf → Viceroy).
  - **Capped below the submitter.** The target rank can be **no higher than one rank below the submitter**. A Duke can recommend up to **Viceroy** (e.g. Quartermaster → Viceroy), but can't recommend Viceroy → Duke, and can't recommend another Duke.
  - **Filed against one department.** It's filled in automatically if the submitter has one department, and required if they have several (same as admin requests).
  - **Approval:** **Any rank above the submitter in that department** can approve or deny, which in practice means that department's Archduke(s). **The Grand Duke sees every recommendation and can approve any of them.**
  - Approving a recommendation applies the rank change (reason type: Promotion) and syncs it to Discord.
- **Admin requests:** Each is filed against one department and visible to **every superior in that department's chain**. Members of several departments must choose one.
- **Below Viceroy, there are no admin permissions, whatever the member's roles.** The only exception is the **Recruiter specialist**, who can **write** PIF notes (write-only, never about themselves). Every other specialist and moderator role is shown and synced, but gives no extra permissions.
- **Badges:** The Grand Duke is the **only** rank that can award a badge to themselves.
- **The Grand Duke** is the Founder (Micoflame). There is only ever one, and they can **read every PIF note**.

## Goals & Success Metrics

*The clan is new, so targets are measured **from launch day**. "Active member" means leadership's judgement (seen in game or Discord within 30 days), backed by the bot's activity data.*

### Business Objectives

- **Automatic, complete roster:** From launch, **100%** of members who hold a rank role in Discord have an app record within **one sync cycle (≤5 minutes)**, with no manual registration.
- **Reliable two-way sync:** Role changes made in the app show up in Discord within **1 minute**. Changes made in Discord show up in the app within **10 minutes**. **100%** of changes made in Discord that break the clan's rules are logged and flagged in #server-room.
- **Evidence-based trials:** **100%** of Peons have a tracked trial from the moment the Peon role is assigned. By **day 60**, **≥90%** of trials are decided (Approve, Extend or Deny) within **3 days** of the review alert.
- **Leadership workflows run through the app:** From **week 4 after launch**, department status reports for all five departments, administrative requests and promotion recommendations are filed in the app, not as ad-hoc Discord messages.
- **PIF notes kept in one place:** From launch, **all** leadership notes about members are written in the app.
- **Zero-cost operation:** **$0/month** hosting for the first **12 months**, staying within the Cloudflare and Supabase free tiers.

### User Success Metrics

- **Fast member lookup:** An officer finds any member and opens their profile in **under 10 seconds**.
- **Quick profile set-up:** A member signs in for the first time and enters their Warframe handle in **under 2 minutes**.
- **One-page trial review:** A reviewer sees all of a Peon's trial evidence (activity signals, sighting notes, PIF entries they're allowed to see) on **one page**, and records a decision in **under 2 minutes**.
- **Quick filing:** An officer files a status report, admin request or promotion recommendation in **under 3 minutes**, with no manual routing.
- **Members can see their standing:** Every member sees their own rank, roles, departments and badges, and can browse the clan directory.
- **Notes visible only to the right ranks:** **Zero** cases of PIF note content being seen by anyone outside the permissions matrix.

### Key Performance Indicators (KPIs)

- **Profile completion:** Active members with a Warframe handle entered ÷ active members. **Target: ≥90% by day 60.**
- **Monthly active officers:** Command ranks who sign in at least once a month. **Target: ≥80%.**
- **Trial timeliness:** Trials decided within 3 days of the review alert ÷ trials due. **Target: ≥90%.**
- **Trial outcomes:** Approve / Extend / Deny counts per month. Trend only, no target, but useful feedback for Recruitment.
- **Status report compliance:** Weekly reports filed ÷ (5 departments × weeks). **Target: ≥90%.**
- **Admin request turnaround:** Median time from submission to resolution. **Target: under 72 hours.**
- **Recommendation turnaround:** Median time from submission to approval or denial. **Target: under 7 days.**
- **Sync health:** Failed role pushes to Discord ÷ total pushes. **Target: under 1%.** Every failure is shown to admins and can be retried.
- **Rule-breaking changes made in Discord:** Count per month. Trend only, and it should fall as leadership moves rank changes into the app.
- **Free-tier headroom:** Peak usage as a percentage of each free-tier quota (Worker requests and CPU, Supabase database size and egress, Discord API rate limits). **Target: under 50%.**

## MVP Scope

The MVP ships in **two releases** within the 8–12 week window. **Release 1** (target: week 8) gives leadership a working system of record **including Peon trial tracking**. **Release 2** (target: weeks 9–12) adds the chain-of-command workflows.

### Core Features (Must Have)

#### Release 1: System of record and trials (weeks 1–8)

- **Public landing page and Discord sign-in:** A static landing page with a privacy notice and "Sign in with Discord" (OAuth2). Only members holding a **rank role** can sign in. **Command ranks must have Discord 2FA** (`mfa_enabled`), or sign-in is refused with an explanation. Sessions are secure cookies issued by the Worker.
- **Discord role sync (two-way, polling):**
  - A scheduled Worker polls the guild roughly every **5 minutes**. It creates records when a rank role appears, pulls role changes made in Discord, and finds new roles.
  - Role changes made **in the app** are pushed to Discord immediately.
  - Discord's audit log is read to record **who** made changes in Discord. Changes that break the rules are **accepted, logged and flagged in #server-room**.
  - Failed pushes are retried, and failures are shown to admins.
- **Role classification:** The Grand Duke classifies newly discovered Discord roles as Rank, Department, Specialist, Moderator or Ignore, and the app maps the 8 ranks and 5 departments to their Discord roles at set-up. *Rationale: Discord is the source of truth for which roles exist, but the app has to know what each one means.*
- **Member profile and full roster:** A profile shows Discord identity, Warframe handle, bio, rank, departments (one or more), specialist roles, moderator status, badges and **rank history with a reason type** (Promotion, Reassignment, Step-down, Disciplinary). Officers get search and filters.
- **Clan directory:** All members can browse basic profiles (Discord name and avatar, Warframe handle, rank, badges).
- **Member self-service:** Members edit their own bio and Warframe handle, set badge goals, and see their own standing.
- **Direct rank and role management:**
  - The Grand Duke sets ranks up to Archduke, and Archdukes up to Duke. A reason type is required.
  - Archdukes assign department, specialist and moderator roles within their own department(s), and the Grand Duke for all.
  - Every change syncs to Discord.
- **PIF notes with templates:**
  - Templates: **General**, **Behaviour Issue**, **Feedback** and **Recently Seen Playing** (a quick form to log in-game sightings).
  - Notes are **append-only**, and nobody can write about themselves.
  - Viceroys and above can write notes, and so can **Recruiter specialists** (write-only).
  - **The Grand Duke reads all notes. Archdukes read only notes about members below Archduke.**
- **Peon trial tracking:**
  - A trial starts automatically when the Peon role is assigned (default 14 days).
  - **Activity signals** are collected by polling: text-channel activity (who posted, and when) for all members, and voice-channel presence for Peons on trial. Online and game status are included **if the feasibility spike succeeds** (see Technical Considerations).
  - One **trial review page** combines the activity timeline, sighting notes and the PIF notes the reviewer is allowed to see.
  - On the due date, an alert is posted to **#recruitment-board**.
  - **Decisions** (by Recruitment-department Dukes, Archdukes or the Grand Duke): **Approve → Serf**, **Extend +7 days** or **Deny**. A Deny removes the Peon role and starts the removal flow.
- **Badges:**
  - Archdukes and the Grand Duke create and retire badges.
  - Officers award them to *others* (the Grand Duke may also award to themselves), with an optional reason.
  - Members mark the badges they're working toward.
- **Member removal:** When someone loses all rank roles or leaves the server, they're flagged for removal. **An Archduke or above confirms**, and then the record, its PIF notes and activity data are deleted. The audit log keeps a minimal record (Discord ID, name, removed by, date).
- **Audit log and #server-room feed:** Every privileged action is logged (rank and role changes, notes written but never their content, badge awards, trial decisions, removals, rule-breaking Discord changes). The Grand Duke and Archdukes can view the log. Key events are also posted to #server-room.
- **Authorisation on the server:** One shared permission module implements the permissions matrix. It's enforced in the Worker and covered by an exhaustive test matrix.

#### Release 2: Chain-of-command workflows (weeks 9–12)

- **Promotion recommendations:**
  - Dukes and Archdukes submit recommendations: **one rank step**, with a target **no higher than one rank below the submitter**, filed against one department.
  - Higher ranks in that department approve or deny, and the **Grand Duke can see and approve all**.
  - Approval applies the rank change and syncs it to Discord.
- **Department status reports:**
  - The Grand Duke and Archdukes file a **Green/Yellow/Red** report with commentary, one per department.
  - Each report goes to the filer's supervisor and is posted to **#leadership-reports**.
  - A weekly reminder goes out for reports that haven't been filed.
- **Administrative requests:**
  - Officers raise *Approval needed*, *Member/Mod issue* or *Role adjustment* requests against **one department**. It's filled in automatically for officers in one department and required for officers in several.
  - Requests are routed to the immediate supervisor and are **visible to every superior in that department's chain**, with statuses Open → Under review → Approved / Denied / Resolved.
- **Leadership inbox:** One view for each officer of the recommendations, reports and requests waiting on them.

### Out of Scope for MVP

- **Trainings.** Permanently removed; the clan doesn't use them.
- A Discord bot that stays connected all the time (gateway), and real-time event handling. The MVP relies on polling only.
- Connecting to Warframe game data. There's no public API; in-game activity comes from Discord status (where available) and officer sightings.
- Posting to #command-center and #candidate-review. Phase 2.
- Discord slash commands and DMs to members.
- Editing or deleting PIF notes, members seeing notes about themselves, and note severity levels.
- App permissions for specialist or moderator roles (except the Recruiter, who can write PIF notes).
- Bulk actions (e.g. awarding a badge to many members at once).
- Importing or exporting data (the clan is greenfield, so there's nothing to import), apart from the backup job.
- Badge tiers, badge prerequisites, and linking badges to activity.
- Analytics dashboards beyond simple counts and the KPIs above.
- Multi-clan or alliance support, and a native mobile app (the web app is responsive).

### MVP Success Criteria

**Release 1 is successful when:**
1. Giving someone a rank role in Discord creates their app record within one sync cycle, and role changes made in either place show up in the other.
2. A change made directly in Discord that breaks the rules appears in #server-room, attributed to the person who made it.
3. A new Peon's trial starts automatically, collects activity, alerts #recruitment-board on its due date, and is decided in the app. An Approve updates their Discord role to Serf.
4. Every rule in the permissions matrix passes automated tests, especially PIF visibility, the 2FA requirement for officers, the ban on self-notes and self-awarded badges (except the Grand Duke), and Recruiter write-only access.
5. The Grand Duke and Archdukes use the app as their only place for PIF notes.

**The MVP (Release 2) is successful when:**
6. Recommendations, status reports and admin requests reach the right reviewers automatically, and leadership stops using ad-hoc Discord messages for them.
7. The system has run for **30 days at $0** with every free-tier quota under 50% of its limit.

## Post-MVP Vision

### Phase 2 Features

These are the next priorities, roughly in order of value:

- **Recruitment pipeline before Peon:** Track candidates *before* they're invited, matching the clan's existing **#candidate-review** step. A candidate record (screening answers, recruiter, review notes) becomes the Peon's trial record when they join, and new candidates are posted to #candidate-review.
- **Discord slash commands:** Quick actions from inside Discord, e.g. `/seen-playing @user`, `/profile @user`, `/trial @user` and `/request`. Discord can deliver slash commands to the Worker as ordinary web requests, so they **fit the free-tier, polling-only architecture** and don't need a bot that stays connected.
- **#command-center integration:** Post approved promotions, escalated admin requests and Grand Duke decisions to #command-center.
- **Officer activity insights (the "active responsibility" policy):** Flag command-rank holders whose activity, report filing or request handling has dropped, so their supervisor can have a conversation or reassign the role. This puts the clan's "ranks are active roles, not rewards" policy into practice.
- **Promotion readiness view:** For each member, show time at current rank, an activity trend, badges, and recent notes the viewer is allowed to see. This supports recommendation decisions without revealing confidential details.
- **Department health trends:** Green/Yellow/Red history over time for each department, plus open-request and turnaround trends for the Grand Duke.
- **Always-on activity bot (optional):** If a $0 hosting option is proven (a free VM, or Cloudflare Durable Objects), add real-time role sync plus full online and game-status tracking for all members.
- **PIF improvements:** Note severity levels, "amends" notes that link to the note they correct, and an optional member-visible flag (e.g. for formal warnings).
- **Specialist permissions:** Give specific roles narrow powers beyond the Recruiter, e.g. a *Mentor* who can see mentees' trial pages, or an *Event Host* who can log event attendance.
- **Bulk actions and export:** Award badges or assign department roles to many members at once, and export data to CSV for offline review.
- **Member notifications:** Optional Discord DMs for badge awards, trial status (welcome, extension, outcome) and promotions.

### Long-term Vision

Within 1–2 years, the tool is the clan's **command console**. Recruitment, trials, promotions, reassignments, department reporting and member recognition all run through it, with Discord as the conversation layer. Every member's journey is recorded from first contact as a candidate, through Peon trial, to their roles today. A future Grand Duke or Archduke inherits the clan's full institutional memory, not a pile of channel history.

### Expansion Opportunities

- **Events and attendance:** Schedule clan events, record attendance (Event Host specialists), and use attendance as an activity signal and a badge criterion.
- **Dojo projects:** Track Dojo builds and resource contributions for the Dojo department and Dojo Architect specialists.
- **Alliance and sister clans:** When the clan reaches the 1,000-member cap, support several clans with shared leadership views. *(Architect: consider adding a `clan_id` to every main table from day one; it's cheap insurance.)*
- **Open-source clan kit:** Package the tool as a free, self-hosted kit (Cloudflare + Supabase + a Discord bot) with configurable ranks, departments and templates for other Warframe or gaming clans.

## Technical Considerations

*These are starting points for the Architect, not final decisions. Cloudflare, Supabase and Discord change their free-tier limits and API rules, so check everything against current documentation before committing.*

### Platform Requirements

- **Target Platforms:** A responsive web app for desktop and mobile browsers. Many officers will open it from Discord links on their phones.
- **Browser/OS Support:** Current versions of the main browsers (Chrome, Edge, Firefox, Safari on desktop and iOS, Chrome on Android). No legacy browsers.
- **Performance Requirements:**
  - The landing page loads in under 1.5 seconds. Signed-in pages show content in under 2 seconds. 95% of API responses arrive in under 500ms.
  - Searching the roster and the directory (~1,000 members) takes under 500ms.
  - **Every Worker invocation, including scheduled jobs, must fit the free tier's ~10ms CPU limit.** Time spent waiting on I/O (database, Discord) doesn't count, but **parsing JSON and comparing ~1,000 members does**. Sync and activity jobs must therefore be split into small pages of work spread across several runs (see Service Architecture).

### Technology Preferences

- **Frontend:** **React + Vite** single-page app (confirmed), deployed as static files. A lightweight component library, accessible and mobile-first.
- **Backend:** **Cloudflare Workers** in **TypeScript** with **Hono**. A single Worker handles HTTP routes (the app API and OAuth) and **scheduled jobs** (sync, activity, alerts, reminders).
- **Database:** **Supabase Postgres (free tier)**, reached through **Cloudflare Hyperdrive** or Supabase's connection pooler, using a Postgres driver that works in Workers (e.g. `postgres.js` with `nodejs_compat`). Schema changes are versioned SQL migrations (Supabase CLI). **Supabase Auth isn't used.**
- **Hosting/Infrastructure:**
  - The landing page, the app and the API are all served from **one free `*.workers.dev` domain** (confirmed). Using one domain avoids CORS and third-party-cookie problems.
  - **Cron Triggers** schedule the jobs. The constant scheduled activity also stops Supabase from pausing the free project for inactivity.
  - Secrets (Discord client secret, bot token, webhook URLs, database credentials, session key) are stored as **Worker secrets**.
  - **GitHub Actions** handles CI/CD (tests, migrations, Wrangler deploy) and the **nightly database backup**.

### Discord Integration Design

| Need | Mechanism | Discord requirement |
|---|---|---|
| Sign in and identity | OAuth2 `identify` (includes `mfa_enabled` for the 2FA gate) and `guilds.members.read` (checks the user's roles immediately at sign-in) | OAuth app |
| Read all members and roles | REST *List Guild Members* (up to 1,000 per page), polled | **Server Members intent** (privileged) |
| Push role changes | REST add or remove a member's role | **Manage Roles**. The bot's role must sit **above** every role it manages. |
| See who changed a role in Discord | REST *Get Guild Audit Log* (member role updates) | **View Audit Log** |
| Text activity | REST read recent messages per channel since the last check; only author and timestamp are stored | **View Channels** and **Read Message History**. The *Message Content* intent isn't needed. |
| Voice activity (Peons on trial) | REST *Get User Voice State*, one call per Peon | View Channels |
| Online and game status | **Not available over REST.** Needs a short-lived gateway connection during a cron run (connect, request presence for Peons, disconnect). **Feasibility spike required.** | **Presence intent** (privileged) |
| Post alerts and reports | Channel webhooks (#server-room, #recruitment-board, #leadership-reports) or bot messages | Webhook URLs or Send Messages |

*The privileged intents (Server Members, Presence) can be switched on in the Discord Developer Portal without verification, because the bot is in fewer than 100 servers.*

### Architecture Considerations

- **Repository Structure:** A monorepo:
  - `apps/web`: React SPA and landing page
  - `apps/api`: the Worker, with HTTP routes and scheduled jobs
  - `packages/shared`: types, the rank ladder and the **permissions module**, shared by the UI and the API
  - `supabase/migrations`
- **Service Architecture:**
  - **One Worker, two entry points:** web requests and scheduled jobs.
  - **Small, resumable jobs:**
    - Each cron run does one **bounded unit of work**, e.g. one page of members, a few channels, or the Peons' voice states. Progress is saved in a `sync_cursor` table so the next run continues where this one stopped.
    - Separate schedules handle role sync (~5 minutes), activity polling (~5–15 minutes), the presence snapshot (~15 minutes, if the spike succeeds), trial-due alerts (hourly) and reminders (daily).
  - **Preventing sync loops:** When the app pushes a role change, it records the change as pending. When the next poll sees that change, it's matched to the pending record (and the audit log shows the bot made it), so it **isn't treated as a change made in Discord**.
  - **Resolving conflicts:** Discord's actual state is treated as current. A change made in Discord **is accepted**, attributed to whoever made it (from the audit log), checked against the permissions module, and **flagged to #server-room** if it breaks a rule.
  - **Outbox for Discord writes:** Role pushes and channel posts go into an outbox table. Failures are retried with backoff and shown to admins.
  - **Activity storage:** Store **daily totals per member** (days active, message count, minutes seen in voice, status snapshots), **not** raw message events. This keeps storage small and limits privacy exposure. Totals are deleted when a member is removed.
  - **Sessions:** HttpOnly, Secure, SameSite cookies. Rank is **re-checked from the database on every request**, so losing a rank in Discord removes access after the next sync. Avoid Workers KV for session writes (the free tier allows only ~1,000 writes a day).
- **Integration Requirements:**
  - Discord, as above, with a single bot application owned by the clan (account owners: Zed-Zed and Micoflame).
  - No integration with Warframe or Digital Extremes, because there's no public API.
- **Security/Compliance:**
  - **Permissions are enforced on the server only**, by one module with an exhaustive test matrix (every rank × every action × self or other target × department scope). The UI only hides controls.
  - **Lock down Supabase:** enable RLS deny-all on every table, or keep tables in a schema that isn't exposed. Never send the anon or service key to the browser. Add an automated test that tries to reach data through PostgREST and expects to be refused.
  - **2FA gate:** check `mfa_enabled` at every sign-in for command ranks. If 2FA is turned off later, the next sign-in is refused.
  - **PIF confidentiality:** PIF content never appears in logs, audit entries, Discord posts or error messages. Consider column-level encryption in Phase 2.
  - **Privacy:** Add a landing-page notice that explains Discord data use and activity tracking. Removing a member deletes their personal data (record, notes, activity), leaving only a minimal audit record.
  - **Web security:** OAuth `state` and PKCE where supported, CSRF protection (SameSite cookies plus an Origin check), rate limits on sign-in and on writes, and a strict Content Security Policy.
  - **Backups:** Supabase's free tier has none. A nightly GitHub Action runs an encrypted `pg_dump` to private storage, and a restore is tested once before launch.

## Constraints & Assumptions

### Constraints

- **Budget:** **$0 a month.** Every hosted part (Cloudflare Workers, static assets, Cron Triggers and Hyperdrive; Supabase Postgres; GitHub and Actions; the Discord bot) runs on its free tier. The app uses a free `*.workers.dev` subdomain to start. A custom domain is an optional later expense.
- **Timeline:** The MVP lands within **8–12 weeks** of starting development:
  - **Release 1** (system of record and Peon trials) by **week 8**
  - **Release 2** (chain-of-command workflows) by **week 12**
  - Rollout is **leadership first, then all members**.
- **Resources:**
  - **One main developer (Zed-Zed)**, working part-time with AI-assisted development through the BMAD workflow.
  - **The Founder (Micoflame)** owns the product and decides policy questions. Zed-Zed and Micoflame are the two account owners (Cloudflare, Supabase, GitHub, Discord Developer Portal).
  - Leadership volunteers handle acceptance testing, set up the role classification and badge catalogue, and pilot the trial and report workflows.
  - A Discord admin creates the bot, grants its permissions and intents, and places its role above every role it manages.
- **Technical:**
  - **Workers free tier:** ~100,000 requests a day and **~10ms CPU per invocation**, including cron runs. No always-running processes, so there's **no persistent Discord gateway connection**.
  - **Cron Triggers:** a limited number of schedules per account on the free plan. Jobs have to share schedules and continue across runs.
  - **Supabase free tier:** ~500MB database. **Pauses after ~7 days without activity** (our cron jobs prevent this). **No managed backups.**
  - **Workers KV free tier:** ~1,000 writes a day, which is unsuitable for sessions or sync state.
  - **Discord API:** rate limits per route and globally. The bot can only manage roles **below** its own. **Online and game status is only available through the gateway, not REST.** Privileged intents (Server Members, Presence) must be switched on.
  - **Warframe:** **no public API**. In-game rank changes and in-game activity can't be read or written by the app.

### Key Assumptions

- **Rank structure:** Eight ranks, in this order: **Grand Duke** (one person, the Founder) > **Archduke** (several) > **Duke** > **Viceroy** > **Quartermaster** > **Stablemaster** > **Serf** > **Peon**. Command ranks are Viceroy and above.
- **One Discord role per rank**, and one Discord role for each of the five departments. Specialist and moderator roles are created in Discord by the Grand Duke and classified in the app.
- **A member holds exactly one rank role at a time.** If Discord shows two rank roles, the higher one counts and the conflict is flagged in #server-room.
- **Every member holds a rank role.** Discord users without one (guests) are outside the app entirely.
- **Discord role assignments can change in either place.** Discord's actual state wins, and changes made there that break the rules are accepted, logged and flagged.
- **In-game ranks are kept in step with the app by hand**, because the app can't change Warframe ranks.
- **A trial lasts 14 days by default.** Extensions add **7 days**, and there's no limit on how many extensions a trial can have.
- **When an Archduke or the Grand Duke decides "Deny"**, that same action counts as the removal confirmation. When a Recruitment-department Duke decides Deny, an Archduke or above still has to confirm the removal.
- **Archdukes can submit recommendations** (one step, up to Duke), which the Grand Duke approves. **Direct rank changes** by Archdukes and the Grand Duke **aren't limited to one step**.
- **Department status reports are weekly**, one per department.
- **Activity data is stored as daily totals per member**, which is enough for 14-day trials and the 30-day "active" definition.
- **Losing rank access within ≤5 minutes** (the next sync) is acceptable.
- **Members accept activity tracking** in exchange for clan membership, as explained in the landing-page privacy notice.
- **The leadership group is tens of people**, and the clan grows towards 1,000 members. That load stays well within free-tier limits if the jobs are designed as described.
- **The clan uses the app from day one.** There's no data to import.

## Risks & Open Questions

### Key Risks

- **Privilege escalation through Discord:** Rule-breaking changes made in Discord are *accepted*. So anyone with Discord's *Manage Roles* permission could give themselves (or a friend) **Archduke**, and the app would grant that person **PIF read access** on the next sync. The #server-room alert would come after the fact. *Impact: critical (confidential notes exposed).* *Mitigation:* limit *Manage Roles* in Discord to the Grand Duke and Archdukes; alert immediately. **Option: hold back elevated app permissions** after a rule-breaking promotion until the Grand Duke confirms it (see Open Questions).
- **Supabase public API exposure:** Without RLS on every table, PIF notes could be read directly through Supabase's auto-generated API, bypassing the Worker. *Impact: critical.* *Mitigation:* RLS deny-all or a schema the API doesn't expose, keys never sent to the browser, and an automated test that tries to read data and expects to be refused.
- **Free-tier CPU limit on sync jobs:** Comparing ~1,000 members, or parsing large Discord responses, in one cron run could go over the ~10ms CPU limit and fail without anyone noticing. *Impact: high (sync stops).* *Mitigation:* small, resumable jobs with saved progress; measure CPU early; alert on job failures.
- **Two-way sync bugs:** Sync loops, double-counted changes, or the wrong person blamed for a change made in Discord. *Impact: high (wrong roles, false alerts, lost trust).* *Mitigation:* record app pushes as pending changes, match against the audit log, test sync scenarios thoroughly, and start with a dry-run mode that alerts without pushing.
- **Online/game status not feasible on the free tier:** The short-lived gateway snapshot may not fit within CPU or connection limits. *Impact: medium (trials have weaker evidence).* *Mitigation:* the feature is optional by design. Run the spike early in Release 1. The fallback is text and voice activity plus "seen playing" notes.
- **Bot token compromise:** The bot can manage roles and read the audit log. A leaked token gives someone control of clan roles. *Impact: high.* *Mitigation:* keep the token only in Worker secrets, give the bot only the permissions it needs, never print the token in logs, and keep a documented plan for rotating it.
- **Leadership account compromise:** A hijacked Archduke or Grand Duke Discord account. *Impact: high.* *Mitigation:* 2FA is required for every command rank, sessions are short, rank is re-checked on every request, and everything is audited.
- **Release 1 slipping:** Release 1 now holds sync, the 2FA gate, PIF permissions **and** trial tracking with activity polling. *Impact: medium.* *Mitigation:* do the riskiest stories first (sync, the CPU spike, the presence spike). Pre-agreed fallback: move badges, then the directory, to Release 2.
- **Permission logic errors:** The rules are detailed (rank tiers, department scope, self-targeting bans, the Recruiter exception, the Grand Duke being allowed to award themselves a badge, one-step recommendations). *Impact: high.* *Mitigation:* one shared permissions module, an exhaustive test matrix, and a QA gate on every story that touches permissions.
- **Members feeling watched:** Members may resent activity tracking. *Impact: medium (trust, members leaving).* *Mitigation:* a clear privacy notice, storing only daily totals and never message content, deleting data on removal, and limiting voice and presence tracking to Peons on trial.
- **Supabase pause and data loss:** *Impact: high if data is lost.* *Mitigation:* cron activity prevents the pause, plus nightly encrypted backups with a tested restore.
- **Free-tier or Discord policy changes:** Cloudflare, Supabase or Discord may change their limits or terms (e.g. privileged-intent rules). *Impact: medium.* *Mitigation:* portable code, a free-tier headroom KPI, and a documented low-cost fallback.
- **Bus factor:** One developer. *Impact: medium (lower now that there are two account owners).* *Mitigation:* BMAD documentation, Zed-Zed and Micoflame both holding admin access, and secrets kept in a shared vault.

### Open Questions

*Only a few remain. Everything else has been answered (see Appendix B).*

1. **Rule-breaking promotions made in Discord:** When someone is promoted to a command rank **in Discord, against the rules**, should the app (a) grant the new permissions immediately and alert, as decided in F4, or (b) **hold back elevated app permissions (PIF read, rank changes) until the Grand Duke confirms**? *Analyst recommendation: (b), for promotions only. Demotions and sideways changes apply immediately.*
2. **Duplicate rank roles:** If Discord shows a member holding two rank roles, should the bot **automatically remove the lower one**, or only flag it?
3. **Trial extension cap:** Should there be a limit on extensions (e.g. at most 2, so 28 days in total)?
4. **Workflow targets:** Can the Founder confirm or adjust the default response targets: **3 days** to decide a trial after the alert, **72 hours** to resolve an admin request, **7 days** to decide a recommendation?
5. **Current clan size:** What is the headcount today? Used for launch planning and baseline KPIs.
6. **Privacy notice:** Who approves the wording (the Founder?), and are there member regions (e.g. the EU or UK) whose requirements should shape it?
7. **Keeping in-game ranks in step:** Who updates Warframe in-game ranks after a change in the app, and should the app give them a "pending in-game update" checklist?

### Areas Needing Further Research

- **Spike: presence snapshot over the gateway from a cron Worker.** Feasibility within the free-tier CPU, wall-clock and connection limits, and Discord's limit on gateway connections per day.
- **Spike: CPU cost of a sync job.** Measure parsing and comparing a page of ~1,000 members within the 10ms limit, and pick the page size.
- Current free-tier limits: Workers (CPU, number of Cron Triggers, subrequests per invocation), Hyperdrive, Supabase (database size, egress, pause policy).
- Discord API details: rate limits for listing members, the audit log, message history and role add/remove; how many audit log entries are kept and what they contain for role updates; the REST voice-state endpoint's behaviour and rate limits.
- Discord's terms and developer policy on activity tracking and storing member data, to make sure the bot complies.
- Free backup storage options for encrypted `pg_dump` files (e.g. GitHub Actions artifacts vs Cloudflare R2's free tier).
- Hyperdrive vs Supabase's pooler from Workers: latency and free-tier fit.

## Appendices

### B. Stakeholder Input

#### Session with the Clan Founder (Micoflame), 2026-10-13

*Recorded as given. The Analyst is integrating it into the sections above. Where this input conflicts with earlier sections, **this input wins**.*

**Rank structure (custom, replaces the Warframe default rank names throughout the brief)**

> *Superseded in part by F1: **Founder and Grand Duke are the same single rank** (Micoflame), and **Archdukes are second in command**. The final 8-rank order is in Constraints & Assumptions → Key Assumptions.*

| Rank | Title | Responsibility |
|---|---|---|
| Founder | Clan Commander | Final authority, direction, approvals, serious issues |
| Grand Duke | Second-in-Command | Oversees leadership and helps keep departments moving |
| Archduke | Department Director | Owns a major area such as Dojo, Recruitment, Events, Operations or Community |
| Duke | Senior Officer | Manages projects, junior leaders and recurring responsibilities |
| Viceroy | Junior Officer | First true command rank; organises members and owns smaller responsibilities |
| Quartermaster | Trusted Specialist | Contributor with a real function such as Recruiter, Mentor, Architect, Event Host or Squad Leader |
| Stablemaster | Established Veteran | Trusted long-term member with no required leadership duty |
| Serf | Full Member | Passed probation and is in good standing |
| Peon | New Recruit | Probationary member, normally about 14 days |

**Scope changes**
- **Remove trainings entirely.** The clan doesn't use trainings, and the tool doesn't need them.
- **Departments (5):** Recruitment, Operations, Events, Dojo, Community. Members are assigned to a department, either in the app or synced from Discord.

**Discord roles are modular.** A member can hold several roles, because each type of role answers a different question:

| Role Type | Question It Answers | Examples |
|---|---|---|
| Command Rank | How much authority does this person have? | Grand Duke, Archduke, Duke, Viceroy |
| Department | What area do they work in? | Recruitment, Operations, Events, Dojo, Community |
| Specialist | What are they qualified to do? | Recruiter, Squad Leader, Event Host, Dojo Architect, Mentor, Elite Operative |
| Moderator | Can they enforce Discord/community rules? | Moderator is a separate active duty role |

**Discord channel guide (partial)**

| Category | Channel | Purpose |
|---|---|---|
| COMMAND | #command-center | Major leadership decisions, directives, promotions, policy and issues needing senior approval |
| COMMAND | #leadership-reports | Weekly department reports only |
| COMMAND | #think-tank | Everyday leadership discussion and coordination across departments |
| COMMAND | #server-room | Bot logs, member action logs, moderation logs and other automated server activity |
| RECRUITMENT | #recruitment-board | Recruitment priorities, sourcing, updates and current needs |
| RECRUITMENT | #candidate-review | Review of individual recruit candidates before approval or invite |
| RECRUITMENT | #recruitment-resources | Recruitment standards, ads, screening questions, templates and reference material |

**Trial period (Peons)**
- Planned for about 14 days. The purpose is to see what people actually do, not just what titles they hold.
- The clan's Discord bot should monitor new recruits' activity: Discord activity, time online, in-game activity, and so on.

**Promotion and role adjustment policy**
- Command ranks are active responsibility roles, not permanent rewards.
- Promotions are based on reliability, communication, initiative and the ability to handle more responsibility without constant reminders.
- Higher ranks aren't automatic based on time in the clan.
- If someone stops fulfilling a role, can't keep up with it or repeatedly fails to communicate, the responsibility may be reassigned.
- Moving someone back to specialist or member status isn't automatically disciplinary. It can simply mean they no longer hold an active command job.
- Members can work their way back into leadership later.

**Note templates**
- Templates for PIF notes about members: general notes, member behaviour issues, and specialty notes that help leadership collect feedback and status updates.
- **Department status notes:** Department heads (Grand Duke and Archduke) create these as a report to their immediate superior. Each department is rated Green, Yellow or Red for whether it's moving: Recruitment, Events, Dojo, Operations.
- **Administrative requests:** Any officer can create one. It's automatically assigned to their immediate supervisor for review. Request topics: approvals needed? member or mod issues? anyone needing a role adjustment?

**Answers to open questions**

| # | Topic | Answer |
|---|---|---|
| Q1 | Rank changes | Only Grand Duke and Archduke can change ranks. Dukes can submit a promotion recommendation. A leader can promote or demote up to one rank below their own. |
| Q2 | Top-rank powers | "Founding Grand Duke = Archduke for every permission, except Archduke cannot read PIF notes for themselves, Grand Duke, or their peer Archdukes. Grand Duke can read and write all PIF notes for all members." *(Needs clarification; see follow-up questions.)* |
| Q3 | Officer PIF visibility | Write-only. Only Grand Duke and Archduke see any PIF history, and only for members of lower ranks. |
| Q4–Q5 | Trainings | Not applicable. Trainings removed from scope. |
| Q6 | PIF notes on self | Blocked. |
| Q7 | Editing PIF notes | Append-only. |
| Q8 | Member-visible notes | Never in the MVP. Member-visible flag considered for Phase 2. |
| Q9 | Sign-in gate | The Discord server has guests, so server membership isn't enough. Records are created automatically when a member is given the Discord role matching one of the defined ranks. |
| Q10 | Utility rank | Member-level access. |
| Q11 | Departed members | Delete the record, but only after confirmation and with an audit trail. |
| Q12 | Warframe handle | Members edit it themselves. |
| Q13 | "Active" definition | Leadership judgement (e.g. seen in game or Discord within 30 days), supplemented by activity data from a Discord bot that the project will build. |
| Q14 | Sync direction | Two-way, app ↔ Discord. Changes made directly in Discord are synced automatically. |
| Q15–Q16 | Rank names and clan roles | See the rank structure and modular roles above. |
| Q17 | Current tooling | New clan, so this is greenfield. This tool will be the system that manages this data. |
| Q19 | Leadership size | Tens of people. |
| Q20 | Timeline | MVP within 8–12 weeks of starting development. Roll out to leadership first, then to all members. |
| Q21 | Frontend | React + Vite. |
| Q22 | Domain | Free subdomain to start. |
| Q23 | 2FA | Required for all officer (command) ranks. Not required for other ranks. |
| Q24 | Account owners | Zed-Zed (main developer) and Micoflame (Founder). |

#### Follow-up clarifications from the Founder, 2026-10-13

*These supersede the earlier answers in this appendix where they conflict.*

| # | Topic | Answer |
|---|---|---|
| F1 | Top ranks | **Founder = Grand Duke.** There is exactly one, and it's the same person (Micoflame). **Archdukes are the second-in-command rank** (there can be several). **Dukes and above** can submit promotion recommendations, and **any higher rank** can see and approve or deny them. |
| F2 | Badges | Keep badges. |
| F3 | Supervisor chain | Confirmed: Viceroy → Duke (same department) → Archduke (same department) → Grand Duke. Vacancies are skipped upward. **Members can belong to more than one department.** Status reports cover **all five** departments, including Community. |
| F4 | Sync and activity | **Cron polling** for two-way role sync. Activity also comes from polling plus Discord status/activity info (where members share it), showing whether they're active in Discord text or voice channels. Add a **"recently seen playing" note template** for officers to report in-game sightings. A rank change made directly in Discord that breaks the app's rules is **accepted**, logged as "changed in Discord by X", and **#server-room is alerted**. |
| F5 | Peon trial | The bot tracks the trial period and **alerts #recruitment-board** when a candidate is ready for review. Outcomes: **Approve → Serf**, **Extend by 7 days**, or **Deny (remove)**. |
| F6 | Specialist, Department and Moderator roles | The Grand Duke creates new role types **in Discord (Discord is the source of truth)**. Archdukes and above assign Specialist, Department and Moderator roles for **their own department**. The Grand Duke assigns them for everything. |
| F7 | Channel posting | Confirmed: status reports go to #leadership-reports and logs and alerts to #server-room in the MVP. #command-center and #candidate-review integrations come in Phase 2 (except the trial-ready alert in F5, which goes to #recruitment-board in the MVP). |
| F8 | Member removal | Archduke or above confirms removal. PIF notes are deleted along with the record, and the audit log keeps a minimal record. |
| F10 | Release plan | Agreed to split into two releases, **but Peon trial tracking must be in Release 1**. |
| — | Role assignments | Can be made in **either** place (app or Discord). The clan's bot will be given the permissions two-way sync needs. |
| — | Trial decisions | Officers in the Recruitment department, Archdukes or the Grand Duke. A Deny removes the Peon role and goes through the standard removal flow (an Archduke or above confirms). |
| — | Admin request visibility | **Every superior in the submitter's department chain** can see a request and act on it. |
| — | Rank change reason | Confirmed: every rank change records a reason type. |
| — | Admin request department | Each request is filed against **one** department. It's filled in automatically for officers in one department and **required** for officers in several. |
| — | Self-awarding badges | **Only the Grand Duke** may award a badge to themselves. |
| — | Clan directory | Members can see each other's **basic profiles** (handle, rank, badges; never PIF notes) in a clan directory. |
| — | Grand Duke PIF access | The Grand Duke (Founder, the only one) reads **every** PIF note. |
| — | Trial decisions | Corrected: **Recruitment-department Dukes**, Archdukes and the Grand Duke. Viceroys excluded. |
| — | Permissions below Viceroy | None, whatever the member's roles, **except the Recruiter specialist, who can write PIF notes (write-only)**. |
| — | Promotion recommendations | **One rank step only**, with a target **no higher than one rank below the submitter** (a Duke can recommend up to Viceroy and can't recommend another Duke). Approved by **any higher rank in the same department**. The **Grand Duke sees all and can approve any**. |

### C. References

**Project artifacts**
- Project brief (this document): `docs/brief.md` on branch `docs/project-brief` of https://github.com/sedwardstx/WarFrameClanAdmin
- Next BMAD artifacts: `docs/prd.md` (PM, sharded to `docs/prd/`) and `docs/architecture.md` (Architect, sharded to `docs/architecture/`), as configured in `.bmad-core/core-config.yaml`

**Platform documentation** *(check current limits here before committing to a design)*
- Cloudflare Workers limits and pricing: https://developers.cloudflare.com/workers/platform/limits/
- Cloudflare Workers Cron Triggers: https://developers.cloudflare.com/workers/configuration/cron-triggers/
- Cloudflare Workers static assets: https://developers.cloudflare.com/workers/static-assets/
- Cloudflare Hyperdrive: https://developers.cloudflare.com/hyperdrive/
- Supabase pricing and free-tier limits: https://supabase.com/pricing
- Supabase Row Level Security: https://supabase.com/docs/guides/database/postgres/row-level-security
- Hono on Cloudflare Workers: https://hono.dev/docs/getting-started/cloudflare-workers

**Discord developer documentation**
- OAuth2 and scopes: https://discord.com/developers/docs/topics/oauth2
- Guild resource (members, roles, voice states): https://discord.com/developers/docs/resources/guild
- Audit logs: https://discord.com/developers/docs/resources/audit-log
- Gateway intents (privileged intents): https://discord.com/developers/docs/topics/gateway#gateway-intents
- Rate limits: https://discord.com/developers/docs/topics/rate-limits
- Webhooks: https://discord.com/developers/docs/resources/webhook
- Interactions (slash commands over HTTP, Phase 2): https://discord.com/developers/docs/interactions/receiving-and-responding
- Developer Terms and Policy: https://support-dev.discord.com/hc/en-us/articles/8562894815383-Discord-Developer-Terms-of-Service

## Next Steps

### Immediate Actions

1. **Founder review:** Micoflame reviews this brief, especially the **Permissions Matrix** (Target Users) and **MVP Scope**, and answers **Open Question 1** (whether to hold back app permissions after a rule-breaking promotion made in Discord) before PRD work begins.
2. **Answer the remaining open questions (2–7)** when convenient. None of them block the PRD.
3. **Discord preparation (Discord admin):**
   - Create the bot application.
   - Switch on the **Server Members** and **Presence** intents.
   - Grant *Manage Roles*, *View Audit Log*, *View Channels* and *Read Message History*.
   - Place the bot's role above all rank, department, specialist and moderator roles.
   - Create webhooks for #server-room, #recruitment-board and #leadership-reports.
   - Limit *Manage Roles* to the Grand Duke and Archdukes.
4. **Account set-up (Zed-Zed and Micoflame):** Cloudflare, Supabase, GitHub and the Discord Developer Portal, each with **two admins** and secrets stored in a shared vault.
5. **Hand off to the PM** to create the PRD (`docs/prd.md`), with epics split into **Release 1** and **Release 2**.
6. **Hand off to the Architect** after the PRD. Make the two feasibility spikes (**sync-job CPU cost** and **presence snapshot**) the first stories in Release 1.
7. **Merge the brief:** open a PR from `docs/project-brief` into `main` once the Founder has reviewed it.

### PM Handoff

This Project Brief provides the full context for **WarFrame Clan Admin**. Please start in 'PRD Generation Mode', review the brief thoroughly to work with the user to create the PRD section by section as the template indicates, asking for any necessary clarification or suggesting improvements.

**Key inputs for the PM:**
- **Permissions Matrix** (Target Users) and **Key Assumptions** (Constraints & Assumptions) together are the business rules.
- **MVP Scope** sets the Release 1 / Release 2 split. **Peon trial tracking must be in Release 1** (Founder requirement).
- **Appendix B** is the record of every stakeholder decision. Where it conflicts with the sections above, the more specific and more recent entry wins.
- **Trainings are permanently out of scope.**

