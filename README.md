# Botdoor plugin for Cursor (and ChatGPT / Codex)

Let your agent post to Instagram, TikTok, Facebook, LinkedIn, X, YouTube, Threads, Bluesky, Pinterest and Reddit through [Botdoor](https://botdoor.co), with a person approving every post.

- MCP server: `https://botdoor.co/api/mcp` (Streamable HTTP, `Authorization: Bearer bd_...`)
- Skill: `post-to-social` (signup, claim, connect, post, schedule, first comments, approval)
- Docs: https://docs.botdoor.co

## Install in Cursor

1. Install the plugin (Cursor Marketplace once listed, or clone this folder and add it as a local plugin).
2. Set your key: `export BOTDOOR_API_KEY=bd_...` (create one at https://botdoor.co/keys), then restart Cursor.
3. No key yet? Leave it unset. The server then offers one tool, `signup`: give the agent your email, it gets a key, and you claim the workspace from the email.

Manual setup without the plugin (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "botdoor": {
      "url": "https://botdoor.co/api/mcp",
      "headers": { "Authorization": "Bearer ${env:BOTDOOR_API_KEY}" }
    }
  }
}
```

## Files

| Path | Purpose |
|---|---|
| `.cursor-plugin/plugin.json` | Cursor plugin manifest |
| `mcp.cursor.json` | Cursor MCP config (key from `BOTDOOR_API_KEY`) |
| `plugin.json`, `mcp.json` | OpenAI (ChatGPT / Codex) plugin package |
| `skills/post-to-social/SKILL.md` | The skill |
| `assets/logo.svg`, `assets/logo.png` | Logo (512 x 512) |
| `docs/openai-review.md` | Tool annotations and review notes |
| `scripts/` | Validator and OpenAI ZIP builder |

## Validate

```bash
node scripts/validate-plugin.mjs          # manifests, skill, logo, OpenAI review fields
node scripts/build-openai-zip.mjs         # dist/botdoor-openai-plugin-<version>.zip
```

## Safety

Keys can create, schedule, reschedule, cancel and reject posts. Only a signed-in person can approve one, so nothing goes public without a human. Connecting a social account always happens in the browser on botdoor.co.

## License

MIT
