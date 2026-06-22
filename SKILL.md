---
name: care-companion-app
description: Build a customized, gamified, single-file web app for remotely supporting a recovering or aging family member — covering nutrition/medication reminders, exercise or rehab routines, an embedded video/audio course (e.g. meditation), streaks, XP, levels, badges, a check-in calendar, and optional recorded voice encouragement. Use this skill whenever the user wants to make a "care app," "recovery tracker," "陪伴APP," "康复打卡APP," a remote caregiving tool for a parent/grandparent/family member, a habit tracker for someone else's recovery, or a Duolingo-style gamified wellness app for a non-technical person to use on their phone. Trigger even if the user just describes the caregiving situation (an injury, surgery, illness, aging parent) and asks "can you build something to help," without using the word "app" or "skill."
---

# Care Companion App Builder

This skill builds a **single self-contained HTML file** (no install, no backend required) that helps someone far away support a family member who is recovering, aging, or managing a health situation. It works in any phone browser, including WeChat's in-app browser, and uses `localStorage` so progress is saved on the device without needing an account.

The defining trait of this skill is empathy-first product design: the requester is usually anxious, far from their family member, and not technical. Lead with reassurance and concrete next steps, not jargon.

## When to use this

Use this skill for requests like:
- "我爸受伤了，我想做个APP帮他恢复" / "my mom just had surgery, I want to help her remotely"
- "Can you make a recovery tracker with reminders for medication and exercise?"
- "I want something like Duolingo but for my dad's rehab"
- Any request to track nutrition, medication, physical therapy, or wellness habits for someone *other than the requester*, especially across distance

Do NOT use this for the requester's own personal habit tracking (that's a more generic tracker, not a "care companion") — though the same techniques apply if asked.

## Step 1: Interview (keep it short, propose defaults)

The person is usually stressed. Ask only what's essential, batch into 1-3 multi-select/single-select questions via `ask_user_input_v0` if available, and propose sensible defaults rather than open-ended questions. Cover:

1. **Who and what** — relationship, condition/injury, any photos/records they can share for context (e.g. X-rays, discharge papers). Never give a diagnosis yourself — only reflect back what a document or photo shows, and defer specifics to their doctor.
2. **Core modules wanted** — nutrition/medication reminders, exercise/rehab routine, an external course (video/audio link) to embed, progress tracking, voice messages. Let them pick a subset.
3. **Gamification** — streaks, XP/levels, badges, celebration animations. Most people want "make it fun like Duolingo" — confirm which specific mechanics.
4. **Language** — match the family member's language, not necessarily the requester's.
5. **Deployment** — do they have a GitHub or Replit account already? This determines which deployment path you recommend (see references/deployment-guide.md).

If the user has already described all of this conversationally (common when they're emotional/in a hurry), don't re-ask — synthesize a concrete first draft and flag your assumptions inline, then let them correct you.

## Step 2: Build from the reference template

`assets/reference-app.html` is a complete, working, generalized example (a real one built for a parent recovering from an ankle/knee injury, including: nutrition checklist, rehab exercises, an embedded 21-day external video course, XP/level system, streak counter with daily reset logic, achievement badges, a 5-week check-in calendar heatmap, a slide-up "today's remaining tasks" reminder drawer, and an optional voice-message recorder for personalized encouragement).

**Do not start from a blank file.** Copy `reference-app.html` and edit it directly — it already has correct mobile-safe CSS, animation timing, localStorage persistence with proper "new day" reset logic (including streak-breaks-if-yesterday-incomplete logic), and WeChat-browser-safe patterns (no localStorage-dependent SSR, no external JS frameworks, no fonts/CDNs that might be blocked in China).

Key things to swap per the interview, all located via simple search-and-replace in the file:

| What to change | Where | Notes |
|---|---|---|
| App title / header name | `<title>`, `.hdr-name` | Keep short — it's a sticky header |
| Nutrition/medication items | `#nut-list` (`.ni` blocks) + their `data-xp` | Each item needs a unique `data-id`; update the `6` hardcoded total in `updateNutUI()`, `checkAllDone()`, badge rules, and the home-tab progress text wherever `/6` or `>=6` appears |
| Exercises | `#sec-exercise` (`.ei` blocks) | Same pattern — update the hardcoded `5` total in matching places if you change the count |
| External course (video/audio) | `MED` array in `<script>`, `course-hdr` text | Each entry needs `{d: day, t: title, s: subtitle, p: param-for-url}`. Build the play URL pattern once (e.g. bilibili `?p=N`, YouTube playlist index, a podcast feed) and reuse it in `buildMedList()` |
| Badges | `BADGES` array + `rules` object in `checkBadge()` | Each badge needs a matching rule function |
| Levels | `LEVELS` array | Keep cumulative XP thresholds; emoji + name per level |
| Doctor/follow-up reminders | `.di` blocks under 复诊提醒/follow-up card | Always phrase as "ask your doctor" / "as advised by your doctor" — never assert specific medical timelines as fact |
| Colors | CSS custom values (`#059669` green, `#7c3aed` purple, `#f59e0b` amber) | Keep semantic meaning (green=done/positive, purple=XP/meditation, amber=streak/warning) unless asked to rebrand |

**If counts change (e.g. 4 nutrition items instead of 6):** grep the file for the old total number and update every occurrence — progress bars, badge rules, `checkAllDone()`, and any UI text. This is the single most common bug when customizing.

**Always keep:**
- The medical disclaimer pattern: exercise/nutrition guidance is presented as general supportive structure, not medical advice — never give specific drug dosages, and phrase clinical milestones ("when can they bear weight") as "ask your doctor," not as fact.
- The "new day" reset logic in `load()` — it's what makes streaks behave correctly across days and is easy to break if rewritten from scratch.
- `localStorage` as the persistence layer for any version meant to be sent as a standalone file the family member opens directly (see deployment guide for when a backend is added instead).

## Step 3: Deployment

Read `references/deployment-guide.md` before advising on deployment — it covers the tradeoffs between sending the raw HTML file directly (no persistence guarantee across browsers/devices), Replit (easiest for non-technical users, gives a permanent link), and GitHub Pages (free, durable, slightly more setup). Match the recommendation to what the requester already has accounts for; don't make them sign up for more services than necessary.

If the target family member is in mainland China and will open the link in WeChat, flag this explicitly: confirm the chosen host isn't blocked (GitHub Pages and Replit are both generally reachable, but verify if unsure), and avoid any external CDN scripts that might be blocked.

## Step 4: Publishing this as a reusable GitHub skill (if asked)

If the requester wants to share this capability publicly (e.g. so other people facing similar caregiving situations can ask their own Claude to build one), package this skill folder using the standard skill format (this SKILL.md + assets/ + references/) and suggest they publish the repo with a short README pointing to this SKILL.md. Remind them the reference app contains placeholder content (a real family's nutrition plan, a specific Bilibili meditation course) that should be called out as *example content to replace*, not literal defaults — add a visible comment block at the top of `reference-app.html` saying so if it isn't already there.

## Tone reminders

The requester is often carrying real worry about someone they love and can't be physically present for. Keep responses warm but practical — confirm what's been built, what's left to decide, and concrete next steps. Avoid making this feel like "just a coding task."
