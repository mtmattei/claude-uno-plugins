# HANDOFF — Cloud-portable Claude Code skills
Updated: 2026-09-09

## Where we are

**Complete.** The goal was making `~/.claude/skills/` available in Claude Code cloud sessions.
71 local skills were sorted into buckets, 68 were uploaded to the claude.ai account (confirmed
working), and this repo was built as a plugin marketplace for local install and sharing.

The headline finding: **repo-declared plugins do not install in cloud sessions.** Committing
`extraKnownMarketplaces` + `enabledPlugins` to a cloned repo's `.claude/settings.json` has no
effect. A cloud session on `mtmattei/Bill-tracker` showed `/root/.claude/plugins/synced/<id>/`
**empty** while the sibling `skills/synced/<id>/` held the account skills. Plugins reach cloud
through account sync, not repo settings. That settings.json was reverted from Bill-tracker.

So the working delivery path for cloud is **account-uploaded skills**. This repo remains the
local install (`/uno:mvux` etc.) and the shareable artifact.

## Last verified state

- **Build**: n/a (no code). `claude plugin validate .` and `claude plugin validate ./plugins/uno --strict` both pass
- **Runtime**: marketplace verified end to end locally — added over HTTPS, 26 skills cached, `uno@mtmattei` installed and enabled, then removed to avoid duplicating the local `~/.claude/skills/` copies
- **Cloud**: 68 account skills confirmed present. .NET SDK **10.0.112** confirmed installed via cloud environment setup script
- **Git**: `main` @ `6adca2f`, clean, pushed to `github.com/mtmattei/claude-uno-plugins` (public, MIT)
- **Bill-tracker**: `master` @ `b21bb62`, the inert settings.json reverted, repo clean

## Next actions (in order)

1. Nothing blocking. The system works as delivered.
2. When adding a skill later, pick the bucket: cloud-wide → stage under `C:\Users\Platform006\out\`, zip it, upload at claude.ai. Local namespaced → `plugins/uno/skills/<name>/`, validate, push. One repo only → that repo's `.claude/skills/`.
3. Optional: add a pointer line to `~/.claude/rules/uno-scaffolding.md` noting its copy lives in `plugins/uno/skills/uno-scaffolding/references/`, so edits get mirrored.
4. Optional: re-test repo-declared plugins in cloud after a Claude Code release, in case the behavior changes.

## Open questions

- Is there a cap on account skills? 68 is a lot and no docs mention a limit.
- The 51 KB scaffolding rules file now exists in two places and can drift.
- `demo-ready`, `uno-verify`, `uno-app-ui-testing` assume a locally running app driven through the Uno App MCP. They carry knowledge in cloud but cannot drive an app there.

## Gotchas found this session (all cost real time)

1. **Repo-declared plugins are inert in cloud.** See above.
2. **The `owner/repo` marketplace shorthand resolves to SSH.** Fails `Permission denied (publickey)` on a machine with no SSH keys, even when HTTPS works via Windows Credential Manager. Use `{"source": "git", "url": "https://github.com/..."}`.
3. **`apt-get update` exits 100 in the cloud base image.** It ships `deadsnakes` and `ondrej` PPAs on `ppa.launchpadcontent.net`, which egress blocks with 403. A non-zero setup script aborts session start, so this fails before reaching the install. Disable those source files first and make `update` non-fatal.
4. **Account uploads accept only Agent Skills spec frontmatter** (`name, description, license, compatibility, metadata, allowed-tools`). Anything else is a hard error, not an ignored field.
5. **`zip` is absent from Git Bash here.** Use PowerShell `Compress-Archive`. The claude.ai skills UI accepts a per-skill `.zip`.

## Relaunch

```powershell
cd C:\Users\Platform006\claude-uno-plugins
claude plugin validate . ; claude plugin validate .\plugins\uno --strict

# install locally (creates duplicates alongside ~/.claude/skills, so usually skip)
claude plugin marketplace add https://github.com/mtmattei/claude-uno-plugins.git
claude plugin install uno@mtmattei
```

Staged upload sources: `C:\Users\Platform006\out\account-skills\` (42),
`account-skills-uno\` (26), `zips\` (68 ready to drag).

Related memory: `project-claude-cloud-skills`, `reference-cloud-env-dotnet`.
