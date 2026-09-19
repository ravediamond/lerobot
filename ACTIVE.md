# ACTIVE — live state (check this first; ARCHIVE.md is the chronological log)

Split 2026-09-20 from the single `ROADMAP_CONTRIBUTION_OPPORTUNITIES.md` (now `ARCHIVE.md`). Purpose:
one page with what's live, so nothing gets re-derived from scratch each session.

---

## Dead lanes — confirmed, do not re-research

- **Draccus (1.5)** — closed 3/3. Doc's own words: "do not open a 4th PR." Gone.
- **Structured logging (1.6)** — done, superseded by `#4192`.
- **jetson-containers `#1733`** — repo has had no human merges since June 2025. Confirmed dead twice already.
- **LanceDB lane (both fast-PR gaps)** — four contributors landed on it in two days. Killed 2026-09-18.
- **ROS2 (1.9)** — graveyard of abandoned PRs; the roadmap itself only asks for a written assessment, not a PR.
- **Attention config (1.2), action tokenization (1.3), encoder freezing (1.4)** — RFC-first, cross-cutting every policy, months-long wait. Never starting these.
- **Eval-metrics / TopReward / plan-adherence (1.1)** — `s1lent4gnt`'s lane: he built ROBOMETER and TOPReward himself and got on the team via exactly that path. Not this lane.
- **Agentic orchestration + RAG (Part 2.2)** — zero issues asking, RFC-tier, lowest mergeability of anything in the doc. This is Air Liquide-identity leaking into the robotics plan, not a real opportunity — cutting it is the point.
- **Recipe authoring (3.2)** — already demoted, high collision risk (fresh blog post, everyone reaches for the same YAML win at once).

## Cadence filler only — mergeable, but say nothing about you

- 1.8 docstrings, 1.16 more devices, Part 4 Tier A (`gc.collect()` removal, type-stub noise). Keep exactly one line on these: pick one up only if a month's `≥1 PR` quota (see `lerobot_monthly_pr_target` memory) is genuinely at risk, otherwise skip.

## Archived, not deleted — see ARCHIVE.md for full detail

- The `ez1540` thread (TurboVLA plugin author outreach) — resolved cordially, 2026-08-28/29 entries.
- The cold-start funnel analysis (`s1lent4gnt`/`nepyope`/`pkooij`, 12-18 months) — explicitly superseded: "wrong reference class — this is a warm start, interview already done." See 2026-07-31 and 2026-08-27 entries.

---

## TODO (live, check off as done)

- [ ] Review `#2936` (`lerobot-analyze-dataset` CLI, Dave-London, stale ~7 months, 0 reviews) — real code review.
- [ ] Build the frame-quality check (frozen/black camera frames) — design in `FRAME_QUALITY_LIVE_HOOK_DESIGN.md`. Start with Surface 1 (episode-boundary hook in `DatasetWriter.save_episode()`, mirroring `#4533`'s pattern); Surface 2 (mid-session live warning in `record_loop`) is a stretch/follow-up.
- [ ] Apply to UMA's "Wild Card - Open candidate" posting — in parallel with the above, not gated on it. Mention the collection-time framing (operator finds out at the rig, not at training time).
- [ ] Ping own stale PRs (`#4241`, `#4592`, `#4326`) — **Monday earliest, not this weekend** (explicit instruction 2026-09-20 — no pinging Maxime, or anyone, on a weekend).
- [ ] Resolve `#2726` (Reachy Mini) rebase conflicts on branch `reachy-mini-rebase` — blocked on own work, not a reviewer, actionable any time.

---

## Open PRs (own) — live status

| PR | Title | State | Gated on | Silent since | Next action |
|---|---|---|---|---|---|
| `#4241` | `ChunkSafetyProcessorStep` | MERGEABLE | Maxime Ellerbach | 2026-08-27 (**3+ weeks**) | Ping Monday. This doc's own domino chain calls this "the credibility unlock." |
| `#4592` | Third-party processor-step plugin discovery | MERGEABLE | — (no reviewer engaged) | 2026-09-09 | Ping Monday. |
| `#4326` | `ActionTokenizerProcessorStep` test coverage | MERGEABLE | — (no reviewer engaged) | 2026-09-09 | Ping Monday — consider re-tagging `jashshah999` (author of `#3123`, the PR this spun off from; engaged once before). |
| `#2726` | Integrate Reachy Mini | `CONFLICTING` | own rebase work | 2026-09-07 | Not blocked on anyone else — fix the conflicts on `reachy-mini-rebase` whenever. |

**Flag, don't let this slide again**: the 2026-09-18 entry in `ARCHIVE.md` said "ping the rest next week." It is now that week. The only reason it hasn't happened yet is the weekend-timing correction above — Monday is the actual deadline, not a soft target.

---

## Job-search targets

| Target | Status | Applied |
|---|---|---|
| UMA — "Wild Card - Open candidate" | Open ~2 months, 100+ applicants already, explicitly open-ended ("something awesome... exceptional builders over perfect pedigrees") | **Not yet** — planned this weekend/early next week |
| UMA — Data & ML Infrastructure Lead | Open ~2 months, **unfilled** — nobody owns data quality there today | **Not yet** |
| Qualia | Open application (per user, 2026-09-20) — not independently researched yet | **Not yet** |
| Mistral — Robotics AI Scientist | Per user, 2026-09-20 — not independently researched yet | **Not yet** |

UMA confirmed Paris-based humanoid-robotics startup, founded by Remi Cadene (LeRobot's own creator) — see `ARCHIVE.md`'s `TODO_AUDIT.md` reference and the 2026-08-28 `ez1540` entry. Qualia and Mistral rows are as stated by the user in conversation, not yet independently verified — check before relying on the "unfilled 2 months" style detail for those two.

---

## Contacts — who owns what, what's owed

| Name | Context | State | Owed |
|---|---|---|---|
| Marine | Met at the Physical AI meetup | Invited user to apply to UMA | No date set — low urgency, no action pending |
| Antoine | LinkedIn outreach | Unanswered (as stated by user, unconfirmed independently) | Check current status before assuming still cold |
| Steven Palma (`imstevenpmwork`) | lerobot maintainer; personal contact from the interview process; personally merged the draccus co-author PRs (`#4260`/`#4265`); publicly reacted ("Great work!") to the Jetson LinkedIn post | Warm, active | Nothing currently owed |
| `pkooij` | lerobot near-team maintainer; personal contact from the interview process; closed `#4400` positively ("great... thanks for the integration") | Warm, active | Nothing currently owed |
| Maxime Ellerbach (`Maximellerbach`) | lerobot maintainer; the one genuinely organic relationship (met via `#4381`→`#4425`→`#4241`); gates `#4241` | Silent since 2026-08-27 | Ping Monday, **not this weekend** |
| CarolinePascal | lerobot maintainer; owns `DATA-14`/`DATA-17` (dataset validation + recording-corruption); `#4533` and `#4549` both open | Not yet contacted | Plan: review `#2936` first to open a channel; frame the frame-quality PR as filling her lane, cite her gist + `#4533` explicitly |

---

## Evidence table — artifact → application req bullet

Paste-ready mapping, so this doesn't get rebuilt from scratch per application.

| Artifact | Maps to |
|---|---|
| `#4599` (Jetson Dockerfile, merged) + HF blog post + LinkedIn post (49k impressions, public reaction from Steven Palma) | "spotting problems, proposing solutions, pushing them through to deployment" / "something awesome" — this already clears the Wild Card bar as-is |
| `#4381` (VLA-JEPA future-frame leak fix, merged) | Judgment on a silently-wrong training objective — caught via paper-vs-code reading, not just reported as a symptom |
| Chunk-safety demo (`#4241` + real SO-101 hardware rollout + published `chunk-safety-demo` repo) | Real hardware validation, not sim/mock — includes a self-found bug in the user's own PR, fixed with a regression test |
| (planned) Frame-quality check PR, collection-time framing | Direct understanding of UMA's actual data-collection operation (teleop volume, unfilled data-quality role), not just the lerobot repo in the abstract |
| `lerobot-policy-turbovla-so101` (published PyPI package) | Owning a real policy architecture end-to-end; portfolio artifact independent of any core merge |

---

## AgentOps req — kept for the Wild Card argument

**Not in my context — paste the saved spec text here.** Per the user: pulled from UMA's job board, since removed, but "the clearest evidence Uma wants that function." Keep the spec text in this file once available; it's cited as the argument for the Wild Card application.

```
(spec text goes here)
```

---

## See also

- `FRAME_QUALITY_LIVE_HOOK_DESIGN.md` — design doc for the frame-quality check (both surfaces).
- `ARCHIVE.md` — the full chronological log everything above was distilled from. Read it for context/history, not for what to do next.
