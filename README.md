# Ajusteur for agents

Ajusteur makes fitted wardrobes, furniture and kitchens in the UAE, made to fit. This repository connects your agent to the engine our workshop builds by. Your agent designs inside what we can make, draws it, gets an honest estimate, and hands you a link that opens ajusteur.com with the design already in it.

- **MCP server:** `https://ajusteur.com/api/mcp`, Streamable HTTP, stateless.
- **Skill:** `design-with-ajusteur` teaches the interview, what to measure and the tool order.
- **Docs:** [docs.ajusteur.com](https://docs.ajusteur.com), also as [llms.txt](https://docs.ajusteur.com/llms.txt).

## 1. Get an API token

Sign in at [ajusteur.com/account/api-tokens](https://ajusteur.com/account/api-tokens) and create one. It starts with `aj_` and is shown once.

```bash
export AJUSTEUR_API_TOKEN="aj_…"
```

## 2. Connect your agent

### Claude Code

The plugin brings the server and the skill together. Claude Code asks for your token and keeps it in your system's credential store.

```bash
claude plugin marketplace add chiekh-a/ajusteur-mcp
claude plugin install ajusteur@ajusteur
```

Inside a session the same two commands start with `/plugin`. To add the server alone, without the skill:

```bash
claude mcp add --transport http ajusteur https://ajusteur.com/api/mcp \
  --header "Authorization: Bearer $AJUSTEUR_API_TOKEN"
```

### Codex

```bash
codex mcp add ajusteur --url https://ajusteur.com/api/mcp \
  --bearer-token-env-var AJUSTEUR_API_TOKEN
npx skills add chiekh-a/ajusteur-mcp -a codex
```

The first command writes this to `~/.codex/config.toml`:

```toml
[mcp_servers.ajusteur]
url = "https://ajusteur.com/api/mcp"
bearer_token_env_var = "AJUSTEUR_API_TOKEN"
```

### Cursor

Put this in `.cursor/mcp.json` in your project, or in `~/.cursor/mcp.json` for every project. Cursor reads the token from your environment.

```json
{
  "mcpServers": {
    "ajusteur": {
      "url": "https://ajusteur.com/api/mcp",
      "headers": { "Authorization": "Bearer ${env:AJUSTEUR_API_TOKEN}" }
    }
  }
}
```

Then add the skill: `npx skills add chiekh-a/ajusteur-mcp -a cursor`.

### VS Code with GitHub Copilot

Put this in `.vscode/mcp.json`. VS Code asks for the token once and stores it.

```json
{
  "inputs": [
    { "type": "promptString", "id": "ajusteur-token", "description": "Ajusteur API token", "password": true }
  ],
  "servers": {
    "ajusteur": {
      "type": "http",
      "url": "https://ajusteur.com/api/mcp",
      "headers": { "Authorization": "Bearer ${input:ajusteur-token}" }
    }
  }
}
```

### Gemini CLI

```bash
gemini mcp add --transport http \
  --header "Authorization: Bearer $AJUSTEUR_API_TOKEN" \
  ajusteur https://ajusteur.com/api/mcp
gemini skills install https://github.com/chiekh-a/ajusteur-mcp
```

### Any other MCP client

The server is one URL and one header. Most clients take a JSON entry like this one. The key names vary a little, so check yours. The [docs](https://docs.ajusteur.com/mcp/connect) list the exact entry for more clients.

```json
{
  "mcpServers": {
    "ajusteur": {
      "type": "http",
      "url": "https://ajusteur.com/api/mcp",
      "headers": { "Authorization": "Bearer aj_…" }
    }
  }
}
```

### The skill on its own

The skill follows the open [Agent Skills](https://agentskills.io) format, so it works in any agent that reads `SKILL.md`.

```bash
npx skills add chiekh-a/ajusteur-mcp
```

## 3. Try it

Ask your agent:

> I have a bedroom recess, 2400 mm wide, 2300 mm high and 600 mm deep. Design a fitted wardrobe with hanging space at both ends and shelves in the middle. Show me a drawing.

The server also offers three prompts, `design_wardrobe`, `design_piece` and `design_kitchen`. Clients that support prompts show them as commands.

## What the tools do

| Tool | What it does |
| --- | --- |
| `families_list`, `families_rules` | What we can design, the limits in millimetres, and the reason behind every rule |
| `materials_list`, `terms_get` | Materials by what they are, and our approved terms |
| `design_storage_create`, `design_storage_resize_bay`, `design_storage_resize_width` | A wardrobe or storage run of hinged bays, fitted or freestanding |
| `design_piece_from_preset`, `design_piece_check` | Any piece made of 18 mm board, from a preset or your own spec |
| `design_kitchen_check` | A straight, L, U or galley kitchen with its worktop |
| `design_describe`, `design_drawing` | The words and the front view to show the customer |
| `estimate_get` | An indicative price in AED, or the named reasons it cannot be priced yet |
| `request_link` | A link that opens ajusteur.com with the design, ready to send to us |

Every tool is documented with its schema, a real request and its real answer at [docs.ajusteur.com/tools](https://docs.ajusteur.com/tools).

## What it will not do

- State a size, a count or a price that a tool did not return.
- Issue a quote, accept one, order or pay. We issue the quote after measuring, and you accept it on ajusteur.com.
- Collect contact details. The request link opens a page where the customer types them in.

Our rates are still being approved. Until a rate is approved, `estimate_get` answers `blocked` and names what is missing. We then price the design by hand.

## Privacy

The server stores no customer data. A request link holds the design, the estimate and the brief, and expires after 30 days. Our privacy policy is at [ajusteur.com/privacy](https://ajusteur.com/privacy).

## Licence

The files in this repository are under the MIT licence. The server and the design engine are Ajusteur's own and are not part of it.
