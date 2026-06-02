# Claude Skills

Personal collection of Claude Code skills built for Customer Success workflows at Archive.

## Available skills

### `product-feedback`
Drafts and submits product feedback posts to Slack `#product-feedback` in Archive's canonical format. Accepts Grain share links, Superhuman links, email screenshots, Slack threads, raw call notes, or just your own idea. Auto-detects client-originated vs CSM-originated mode, looks up contact titles in HubSpot, and previews the draft inline before posting.

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/emanuelherrera112-star/claude-skills.git ~/claude-skills-repo
ln -s ~/claude-skills-repo/product-feedback ~/.claude/skills/product-feedback
```

Or copy a single skill:

```bash
cp -r path/to/this/repo/product-feedback ~/.claude/skills/
```

## Requirements

These skills assume you have the following MCP servers configured in Claude Code:
- Slack (for posting and reading threads)
- HubSpot (for contact/company lookups)
- Grain (for fetching call notes and transcripts)
- Superhuman (for fetching email threads)

Without these MCPs, fetch operations will fail silently. Most of the skills will still run on raw text inputs as a fallback.

## Updates

`git pull` from inside the cloned directory picks up new skills or updates to existing ones.
