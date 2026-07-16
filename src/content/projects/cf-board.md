---
title: "cf-board — shared trip board on Cloudflare Workers"
summary: "A no-login, real-time-enough family trip board. One Worker serves a React app and a KV-backed API; everyone with the URL sees the same live board. Runs for ~$0/month with hourly automated backups."
stack: ["Cloudflare Workers", "Workers KV", "React", "Wrangler"]
repo: "https://github.com/newtonetwork/cf-board-template"
demo: "https://cf-board.nick-garci28.workers.dev"
order: 1
---

## Problem

Coordinating a multi-family trip meant itinerary details scattered across
group texts. Shared docs require accounts, and nobody installs an app for
one trip. The requirement: one URL anyone can open, edit, and trust to be
current — with zero onboarding.

## Architecture

A single Cloudflare Worker does everything:

- Serves the static React app via Workers static assets.
- Exposes a tiny JSON API (`/api/kv/:key`) backed by Workers KV for board
  state — lists, schedules, assignments.
- A scheduled trigger snapshots KV hourly to versioned backup keys, so a
  bad edit is recoverable.

No accounts, no database server, no build pipeline beyond `wrangler deploy`.

## Decisions

- **KV over D1:** board state is a handful of JSON documents with
  last-write-wins semantics — a key-value store fits; SQL would be
  ceremony. KV's eventual consistency is acceptable for this use.
- **Capability URL instead of auth:** the unguessable URL is the access
  control. Right-sized for a family trip board; documented as an explicit
  trade-off, not an oversight.
- **Backups before features:** the hourly KV snapshot shipped before nice-
  to-haves, because "someone deleted the schedule" is the actual risk.

## Outcome

In active family use. Total infrastructure cost: $0 on the Workers free
tier. Deploy-to-live is under a minute.

## Source

The production board runs private (it holds real family data), so the linked
repo is a genericized, reusable **template** of the same architecture — the
Worker, the KV-backed API, the hourly backup cron, and edge rate limiting —
— a feature-for-feature port with the family data swapped for a generic
4th-of-July lake weekend. Five tabs: day-slotted Meals with menus and potluck
sides, a Staples grocery list with got-it tracking, categorized Gear, a Roster
with case-insensitive identity and party headcounts, and a time-sorted Events
schedule with RSVPs — plus a claim-progress bar, live weather card, offline
mode with write replay, and a hidden fireworks easter egg. Clone it,
`wrangler deploy`, and you have your own board in about five minutes.

The **live demo** runs the template unchanged — open it in two tabs (or send
it to a friend) and watch edits sync. It's a no-login board, so treat anything
you type there as public.
