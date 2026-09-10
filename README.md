# claude-uno-plugins

A Claude Code plugin marketplace holding one plugin, `uno`, with the Uno Platform / WinUI XAML
skills. Its purpose is to make those skills available in **cloud sessions**, which do not read
`~/.claude/skills/` on a local machine.

## Install

Add the marketplace and enable the plugin:

```bash
claude plugin marketplace add mtmattei/claude-uno-plugins
claude plugin install uno@mtmattei
```

## Per-repo (the cloud path)

Cloud sessions install plugins declared in the **cloned repo's** `.claude/settings.json`.
Plugins enabled only in user settings do not transfer. Commit this into a project:

```json
{
  "extraKnownMarketplaces": {
    "mtmattei": {
      "source": {
        "source": "github",
        "repo": "mtmattei/claude-uno-plugins"
      }
    }
  },
  "enabledPlugins": {
    "uno@mtmattei": true
  }
}
```

Skills then appear namespaced, for example `/uno:xaml-art-direction`.

## Layout

```
.claude-plugin/marketplace.json     marketplace manifest
plugins/uno/
  .claude-plugin/plugin.json        plugin manifest
  skills/<name>/SKILL.md            one directory per skill
```

## Adding a skill later

1. Copy the skill directory into `plugins/uno/skills/<name>/`.
2. Run `claude plugin validate ./plugins/uno --strict`.
3. Commit and push. Cloud sessions pick it up on their next start.

Skills here may use any Claude Code frontmatter field. That is the difference from skills
uploaded to a claude.ai account, which accept only the Agent Skills spec fields
(`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`).

## Note on runtime-verification skills

`uno-verify` and `demo-ready` drive a locally running app through the Uno App MCP server.
They carry useful knowledge in a cloud session but cannot launch or inspect an app there.
