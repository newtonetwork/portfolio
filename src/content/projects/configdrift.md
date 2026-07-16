---
title: "ConfigDrift — self-hosted configuration drift monitor"
summary: "A Go agent snapshots host state into SQLite and diffs it against a promoted baseline; a FastAPI + HTMX UI reviews, acknowledges, and promotes drift. Built with NIST 800-53 configuration-management controls in mind."
stack: ["Go", "SQLite", "FastAPI", "HTMX", "Docker"]
repo: "https://github.com/newtonetwork/configdrift"
order: 2
---

## Problem

Configuration drift is how compliant systems quietly stop being compliant.
NIST 800-53's CM family (CM-2 baseline configuration, CM-3 change control,
CM-6 configuration settings) assumes you can answer: *what changed on this
host since the approved baseline, and who acknowledged it?* Most homelab
and small-team setups can't answer that at all.

## Architecture

Two components, both self-hosted in Docker:

- **Agent (Go):** snapshots watched state into SQLite — file and directory
  metadata/content, installed packages (`dpkg-query` with `rpm -qa`
  fallback), systemd units, per-user crontabs and timers, and running
  Docker containers via the Engine API. Each run diffs against the
  promoted baseline and records drift events.
- **UI (FastAPI + HTMX):** syntax-highlighted diff viewer to inspect each
  drift event, acknowledge it, or promote the new snapshot as the
  baseline. Immediate and digest alerts go out through an optional ntfy
  webhook — no external services otherwise.

## Decisions

- **Baseline + promote, not just alerting:** drift tools that only alert
  train you to ignore them. The acknowledge/promote workflow mirrors how
  change control actually works: drift is either approved (promote) or a
  finding (investigate).
- **Ignore rules and content redaction** for noisy or secret-bearing paths,
  so the drift log stays reviewable and safe to retain as evidence.
- **SQLite over a server database:** one host, one file, trivial backup —
  the operational cost of Postgres buys nothing here.

## Compliance angle

The drift event log — what changed, when, and its acknowledge/promote
disposition — is exactly the shape of evidence a CM-family control
assessment asks for. The roadmap pairs it with LLM-generated evidence
narratives per drift event.
