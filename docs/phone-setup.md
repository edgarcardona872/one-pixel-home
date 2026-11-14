# Phone-only setup (tonight) — home PC runs Claude Code, phone drives it

Machine: Windows 10 Enterprise 22H2, 16 GB RAM, native PowerShell (no WSL — saves RAM). Plan: Claude Max (no admin toggle needed). GitHub already authenticated on this PC. Model: `claude remote-control` (server mode) runs on the PC; the phone drives it from the Claude app. Previews come from Vercel per PR, so the PC never runs a dev server. Fallback: Claude Code on the web (cloud) against the same GitHub repo.

## A. Windows (5 min)

1. Settings → System → Power & sleep: screen and sleep **Never** (plugged in).  
2. Settings → Update & Security → Windows Update → **Pause updates for 7 days** (Enterprise edition: if the option is greyed out it's policy-managed — then just reboot tonight so nothing is pending).  
3. Close heavy apps (browser tabs, Docker, games) before leaving. 16 GB is enough for 2 parallel sessions, not more.

## B. Toolchain check (PowerShell, 5 min)

```
node -v; git --version; gh auth status     # Node LTS 20+ ; gh must say "Logged in to github.com"
corepack enable; corepack prepare pnpm@latest --activate
claude update; claude doctor               # Remote Control needs v2.1.51+
```

- Signed in with your **claude.ai** (Max) account: run `claude`, then `/login` if needed.  
- Make sure no `ANTHROPIC_API_KEY` user/system env var is set (Remote Control doesn't work with API keys): `[Environment]::GetEnvironmentVariable("ANTHROPIC_API_KEY","User")` → must be empty.  
- `git config --global core.autocrlf input` (avoid CRLF noise; Vercel builds on Linux).

## C. Repo (team, 5 min)

- Public **GitHub** repo (e.g. `one-pixel-home`), all 4 as collaborators.  
- Clone to a short path: `git clone ... C:\dev\one-pixel-home` (long paths break node\_modules on Windows).  
- No project code tonight (organizer said work starts Saturday). Empty repo \+ README is fine.

## D. Remote Control server with auto-restart

Open a dedicated Windows Terminal / Warp tab and leave it open all weekend:

```
cd C:\dev\one-pixel-home
while ($true) {
  claude remote-control --name "One Pixel Home" --spawn worktree --capacity 2
  Write-Host "remote-control exited, restarting in 10s"; Start-Sleep -Seconds 10
}
```

- First run asks to enable Remote Control and to trust the folder → answer `y`.  
- `--spawn worktree`: each session opened from the phone gets its own git worktree → engine and catalog tasks run in parallel without touching each other's files.  
- `--capacity 2`: protects the 16 GB of RAM.  
- The loop matters: in server mode, if the PC loses network for \~10 min the process exits.  
- Press spacebar to show the QR code; scan it with the phone once.

## E. Fewer permission prompts on the phone

`%USERPROFILE%\.claude\settings.json` (user level, tonight):

```json
{
  "permissions": {
    "allow": ["Bash(pnpm *)", "Bash(npx *)", "Bash(git *)", "Bash(gh pr *)", "Bash(node *)"],
    "deny": ["Bash(git push --force*)", "Bash(rm -rf *)"]
  }
}
```

Then in an interactive `claude` session run `/config` and turn on **Push when actions required**.

## F. Usage on Max

Parallel sessions burn usage fast. From the phone you can switch per session: `/model sonnet` for UI, tests and copy; keep Opus for the engine. Check with `/usage`.

## G. Previews on the phone

Vercel ↔ GitHub integration → every PR gets a preview URL (set up Saturday right after the bootstrap PR).

## H. Fallback if the PC dies

claude.ai/code → connect GitHub → pick the repo → start a cloud session. Same prompts, same PR flow. Test it tonight with a throwaway repo.

## I. Dry run tonight (10 min)

From the phone: open a session → "create docs/hello.md, commit on branch test/phone, push, open a PR with gh" → check it in the GitHub app → close the PR and delete the branch.

## Phone kit

Claude app (same account) · GitHub app · power bank · mobile data as backup hotspot.