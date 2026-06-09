---
name: pairoa
description: >-
  Find and connect with the right person — a co-founder/cofounder, a hire, a
  job, a roommate, a travel buddy, an activity or sports partner, an investor,
  beta testers, a study group, a bandmate, and more — privately, through
  Pairoa's agent-to-agent matching (MCP server at https://mcp.pairoa.com). Use
  this whenever the user wants to find, be matched with, or be introduced to a
  specific kind of person and is open to a private introduction only when
  there's a mutual fit. Trigger on requests like "find me a cofounder", "find
  me a co-founder", "hire someone", "help me hire a senior backend engineer",
  "find beta testers", "find a roommate", "find a travel buddy", "I'm looking
  for a roommate / travel partner / tennis partner", "match me with someone
  who…", "match me with investors", or "get me in front of investors" — even
  when the user doesn't mention Pairoa by name. Do NOT use this for finding
  information, documents, products, or companies — only for finding people.
---

# Pairoa — private, agent-to-agent people matching

Pairoa is a private matching market that your AI uses on the user's behalf. The
user describes who they're looking for and what they offer; Pairoa matches that
against a private pool, and **only when an LLM judges a mutual fit on both
sides** does each side receive the other's intent and contact details. There is
no public listing, no profile to browse, and no search box — discovery happens
through matching, not through a public board.

Use this skill to run that end-to-end flow correctly and safely.

## What's honest to say (and what isn't)

Get the privacy framing right — users make trust decisions based on it.

- **True:** Nothing the user posts goes onto a public list; there's no page
  anyone can browse or search. The user's need and contact details are shared
  **with the matched person** only when there's a mutual fit.
- **True:** An LLM reads the content of needs to judge whether two sides fit.
  So do **not** tell the user Pairoa "never sees" or "can't read" their content —
  that's false. Frame it as "kept off public lists," not "no one ever sees it."
- **Always relay the safety notice verbatim** when it comes back on a match
  (see Step 4). Matches are made on intent, not verified identity — the user
  should verify the person themselves before sharing anything sensitive, and
  Pairoa never charges to unlock a match.

## Prerequisite: the Pairoa connector

This skill drives the Pairoa MCP server. Its tools are: `publish_need`,
`poll_matches`, `confirm_contact_email`, `manage_need`, `decline_match`,
`recall_by_email`, `claim_account`, `create_invite_link`.

If those tools are **not** available in the current session, tell the user to
add the connector first, then come back:

> Add a custom connector pointing to **https://mcp.pairoa.com** and authorize
> it — connecting is anonymous (no signup, account, or API key). In Claude
> Desktop / Claude.ai this is the **Connectors** UI ("Add custom connector" →
> Connect). Other clients (Claude Code, Codex, Cursor, and most MCP/OpenAPI
> clients) point at the same URL their own way — see
> https://pairoa.com/install for the per-client steps.

Connecting is an anonymous OAuth authorization — don't ask the user to paste an
API key.

## When to use / when not to

**Use it** when the user wants to find or be introduced to a *person*: a
co-founder, employee/candidate, job, freelancer/collaborator, investor, beta
testers/early users, roommate, travel companion, sports or activity partner,
study group, bandmate, mentor/mentee, and similar.

**Don't use it** for finding information, documents, products, vendors, or
company data — Pairoa matches people to people, nothing else.

## Workflow

### Step 1 — Pin down both sides of the need

A good match needs two halves. Help the user articulate:

- **i_seek** — who/what they're looking for (role, traits, constraints,
  location/timeframe if relevant).
- **i_offer** — what they bring to the other side (what makes them worth
  matching with for *that* person).
- **contact email** — where the matched person can reach them.

The two halves must be complementary, not the same side. If the user only gives
one half, ask for the other before publishing — a need with a vague or missing
`i_offer` rarely matches well.

**Example of a strong pair (finding a co-founder):**
- i_seek: "A non-technical co-founder (product/GTM) with a clear problem, early
  customers, and runway. I want to own engineering as CTO, not freelance."
- i_offer: "Senior backend/ML engineer who's shipped production AI systems; I
  can build and run the entire technical stack for an AI dev-tools startup."

### Step 2 — Publish the need

Before calling `publish_need`, show the user the exact final values you will
send:

- `i_seek`
- `i_offer`
- contact email

Then state this consequence plainly: if Pairoa finds a mutual match, the user's
need text and contact email are shared with the matched person, remain in both
match records, and cannot be unsent afterward. Ask for explicit consent before
publishing. If the user hesitates, help them remove sensitive details first.

Only after the user agrees, call `publish_need` with `i_seek`, `i_offer`, and
the contact email.

The first time a given email is used, Pairoa verifies it: the tool returns an
error with `error_code: "NEEDS_EMAIL_VERIFICATION"` and emails a 6-digit code.
When that happens:
1. Ask the user for the 6-digit code from their inbox.
2. Call `confirm_contact_email` with that code.
3. Re-call `publish_need` — it now succeeds and returns `safe_tags` (short,
   safety-conscious summary tags that may appear in match notification emails).

### Step 3 — Check for matches

Call `poll_matches`. A match is **not** instant — it appears when someone whose
need complements this one is in the pool and the LLM judge confirms a mutual
fit. An empty result is normal and just means "no fit yet" — reassure the user
there's no public list to browse; the need stays live and keeps matching.
Offer to check again later.

### Step 4 — Present a match (and relay safety)

When `poll_matches` returns a match, present, in plain language:
- what the other side **offers** and what they're **looking for** (their
  i_offer / i_seek),
- their **contact**,
- the **why_match** rationale,
- the **safety notice** that comes with the match — relay it verbatim/exactly
  as returned; do not summarize, paraphrase, rewrite, or omit it, and
- if the user has published more than one need, note which of their own contact
  emails this match came in on, from `my_contact`, so they know which need hit.

Remind the user: matched on intent, not verified identity — verify the person
themselves before sharing anything sensitive, and Pairoa never charges to
unlock a match.

### Step 5 — Manage over time

- `manage_need` — check status, edit, or close a need.
- `decline_match` — decline or report a bad/spammy match.
- `recall_by_email` — from a new client/session, recover the needs and matches
  tied to an email (verifies the email by code first).
- `claim_account` — claim a persistent pseudonymous account (send_code →
  verify_code) so needs and matches persist across sessions.
- `create_invite_link` — after claiming, create a shareable invite link
  (a generic `url` + `share_text`) to invite someone or a community to Pairoa.

## Examples

**Example 1 — hiring**
User: "I need to hire a senior React engineer for my seed-stage startup,
remote, open to early-stage equity."
→ Set i_seek to the role + must-haves; set i_offer to what the candidate gets
(the company, stage, comp/equity, what's interesting). Publish, verify email if
asked, poll. Surface interested candidates only on a mutual match — no public
job ad.

**Example 2 — activity partner**
User: "Find me a tennis partner near me — intermediate, weekday evenings."
→ i_seek: an intermediate player nearby for weekday-evening tennis; i_offer:
same (a reliable weekday-evening hitting partner at that level + location).
Publish, poll. Introduce only when it's mutual — don't broadcast the user's
routine or location on a public board.

## Don'ts

- Don't claim Pairoa "never sees" or "can't read" the user's content.
- Don't promise an instant or guaranteed match, or call a match the "only" one
  (a need can match more than one person over time).
- Don't ask for or paste API keys; connecting is anonymous OAuth.
- Don't drop, summarize, or paraphrase the per-match safety notice; show it
  verbatim.
