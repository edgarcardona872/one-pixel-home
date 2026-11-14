# Kickoff prompts — Saturday 9:00 onward

Paste these into Claude Code (Remote Control session, cloud session, or a teammate's terminal). Each prompt \= one branch \+ one PR. Run `pnpm test` before merging. José Luis merges.

## 0\. Repo bootstrap (José Luis, laptop, 9:30 — after Mission 1 confirms the plan)

> Create a Vite \+ React \+ TypeScript (strict) app with Tailwind and Vitest, pnpm. Add MIT LICENSE, the CLAUDE.md and docs/ folder I'm committing, `.github/workflows/ci.yml` running `pnpm test` and `pnpm build` on PRs. Create empty `src/engine/` and `src/ui/` per CLAUDE.md. Commit and push to main. Then: import the repo in Vercel (preview URL per PR).

## 1\. Engine core (Edgar, phone → Remote Control, 10:00)

> Read docs/engine-spec.md fully. Implement in src/engine: constants.ts, sources.ts (S1–S10 with URLs), rng.ts (seeded), orbit.ts, power.ts, comms.ts, coverage.ts with the exact formulas of §4. Write the sanity tests of §7 first, then the code until they pass. No React imports. Open a PR `feat/engine-core`.

## 2\. Catalog \+ evaluateDesign (Edgar, parallel session in its own worktree, 10:30)

> Implement catalog.ts with every value from spec §6 (each with sourceId and verify flag) and evaluate.ts → evaluateDesign(design, scenario, catalog): DesignCheck with all fail/warn rules of §4 as Reason objects. Add the §7 balancing test (brute-force all combinations). PR `feat/engine-evaluate`.

## 3\. UI shell (José Luis \+ Thais, 10:00)

> Build the game shell per spec §1–2: phase stepper (Idea → Concept & Trades → Point Design → Go/No-Go → Simulation → Debrief), 4 role tabs (Science, Space, Launch, Ground), and a resource bar (mass, volume, power, battery, data, budget, timeline) fed by a mocked DesignCheck. Mobile-friendly. PR `feat/ui-shell`.

## 4\. Wire UI to engine (José Luis, after PRs 2 \+ 3\)

> Replace the mock with evaluateDesign(). Each role tab edits its part of MissionDesign. Show reasons under each resource with severity colors and a "source" chip linking to the citation.

## 5\. Simulation \+ score \+ debrief (Edgar, Saturday afternoon)

> Implement simulate.ts and score.ts: day-by-day timeline (seeded events: solar storm −20% power for 3 days, ground-station outage 2 days, licence delay), stars 0–3, breakdown science/efficiency/ collaboration (unanimous Go \+ all sign-offs). closestRealMission picks from a 3-mission table. Every event/reason carries a sourceId. PR `feat/engine-simulate`.

## 6\. Trade study \+ Go/No-Go (Thais \+ José Luis)

> "Save as Design A/B" and a side-by-side comparison using compareDesigns(). Go/No-Go screen where each active role votes; any No-Go shows that role's open fails.

## 7\. Sunday polish

- Tutorial tooltips explaining each concept in one sentence (Belle writes copy).  
- Sources page listing S1–S10 \+ AI-use disclosure.  
- Record the 30-s demo from the Vercel URL (English subtitles).

## Project page text (Edgar, Sunday from phone)

> Using docs/engine-spec.md and the merged code, draft the Space Apps project page fields in English: High-Level Summary, Detailed Description, NASA Data (S1–S10 and how each was used), Other Data/Tools (with licenses), and Use of AI disclosure (Claude Code generated code and docs; humans reviewed and validated formulas). Keep each field concise.