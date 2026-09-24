# Shift Desk

A real-time call center workforce dashboard — an autonomous team wallboard showing every agent's live status, plus a persistent daily/weekly/monthly team report broken down by agent and task type. Built with the [Claude API](https://docs.claude.com).

## Live demo

**[Try it here](https://claude.ai/artifact/KaFqGu6hVBKiXNtYzEqfUH)** — note: trying the interactive part requires a free Claude account to sign in with. This project also uses Anthropic's `db` persistence capability, which requires the artifact to stay within the owner's organization rather than being fully publicly shareable — a platform requirement for that feature.

## The problem

Call center supervisors need a real-time read on their whole team's day — who's on a call, who's stuck in after-call work, who's buried in administrative tasks — plus reliable rollups over time (today, this week, this month) broken down by both agent and task type, without that data living in five different disconnected systems. This project models that as a single live wallboard backed by real persistent state tracking.

## What it does

**Live Team View** — a pure observation wallboard. Every agent (a 5-person roster) works through their own shift autonomously: taking inbound calls, making outbound calls, after-call work, breaks, lunch, and administrative tasks (Letter Review, Autopay Setup, Damaged Item Review, Refund Processing, Other) — entirely on their own timing, no manual control. Each card shows:
- A live status badge and ticking timer
- Every category broken out with count, average handle time, and total time (e.g. "4 taken · 3.2m avg handle · 12.8m total")
- A running "Total time logged today" summary

**Team Report** — Today / This Week / This Month toggle, using the exact same per-agent card format as the live view (for consistency), plus a per-agent admin-task-by-type breakdown nested under each agent's admin summary, a whole-team admin breakdown table, and an AI-written operational insight naming the most notable pattern worth a manager's attention (e.g. unusually long ACW for one agent, a heavy admin load on another).

**Generate Demo History** — backfills 13 days of realistic shift data across the full roster so weekly/monthly reports have real substance immediately, without touching today's live data.

## Why the simulation is autonomous, not manually driven

An earlier version let the viewer manually click an agent through their state machine (start a call, end it, go to lunch, etc.). That's useful for understanding the state machine, but it's not how a real supervisor dashboard works — a supervisor observes what agents are actually doing, they don't drive it themselves. The final version has each of the 5 agents independently decide their own next action on realistic probabilities and timing, and the dashboard purely reflects that live — a more honest model of what this tool would actually do in production.

## How it works

- A compressed-time simulation (real time runs ~18x faster than logged time) drives each agent through a probabilistic state transition graph — call → after-call work → ready → next action — so a "9-minute call" is watchable in real time but still logs as 9 real minutes in the data
- Each state change persists to two places in Anthropic's `db` capability: a `status` doc (current state, for the live wallboard) and a `shifts` doc keyed by agent + date (an array of completed sessions, for reporting)
- The Team Report aggregates real stored session data across a selected date range — genuinely computed, not simulated at report time
- One Claude API call generates the operational insight narrative from the real aggregated numbers

## Tech

- HTML / CSS / JavaScript (single file, no build step)
- Claude API, via Anthropic's Artifact runtime
- Real persistent storage (`db` capability) for live status and historical reporting
- Autonomous multi-agent state simulation
