# One Pixel Home — NASA Space Apps 2026 (Space Mission Design Game)

## What we are building

A browser game where 1–4 players design, launch and simulate an Earth-observation CubeSat mission. Every consequence must be explained ("why it failed / succeeded") and traceable to a source. Challenge: [https\://www\.spaceappschallenge.org/2026/challenges/space-mission-design-game/](https://www.spaceappschallenge.org/2026/challenges/space-mission-design-game/) Full design spec: `docs/engine-spec.md` (read it before touching `src/engine`).

## Stack

- Vite \+ React \+ TypeScript (strict), Tailwind CSS, Vitest. Package manager: pnpm.  
- Deploy: Vercel (GitHub integration, preview URL per PR).  
- No backend. Co-op \= up to 4 roles on ONE shared screen (tabs), no networking.

## Layout

- `src/engine/` — pure TypeScript, NO React/DOM imports. Deterministic (seeded RNG).  
  - `constants.ts` (physical constants with source ids), `catalog.ts` (components/launch/stations), `orbit.ts`, `power.ts`, `comms.ts`, `coverage.ts`, `budget.ts`, `evaluate.ts`, `simulate.ts`, `score.ts`, `rng.ts`, `sources.ts`.  
  - Tests in `src/engine/__tests__/`. Every formula gets a sanity test (see spec §7).  
- `src/ui/` — React components. Roles: Science, Space, Launch, Ground (+ Mission Lead view).  
- `docs/` — spec, prompts, sources.

## Rules (non-negotiable)

- UI text, code comments and docs in **English**.  
- Every number shown to the player comes from `catalog.ts`/`constants.ts` and carries a `sourceId` (`NASA-*`, `PHYS`, or `GAME`). Never invent a NASA citation. Unknown → tag `VERIFY`.  
- Every failure/success reason is a `Reason { code, message, sourceId }` — no bare strings.  
- No NASA logos, insignia, flags or mission patches anywhere (Space Apps AI/branding rules).  
- AI-generated images must carry a visible "AI-generated" watermark.  
- Keep the engine free of UI concerns; UI reads `evaluateDesign()` / `simulateMission()` only.  
- Small PRs, one concern each. Branch names: `feat/<area>-<thing>`. Conventional commits.

## GitHub

- Use the `gh` CLI for all GitHub operations (`gh pr create`, `gh pr view`, `gh pr merge`, `gh pr close`). It is installed and authenticated on this machine.  
- There is no GitHub MCP server here — do not search for `mcp__github__*` tools.  
- Repo: `edgarcardona872/one-pixel-home`. Default branch: `main`.

## Commands

- `pnpm dev` — local dev server  
- `pnpm test` — Vitest (must pass before every PR)  
- `pnpm build` — type-check \+ production build