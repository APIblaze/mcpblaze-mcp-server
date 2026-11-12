<p align="center">
  <img src="assets/owlbear.png" width="160" alt="The MCPblaze owlbear, hugging the logo">
</p>

<h1 align="center">MCPblaze</h1>

<p align="center">
  <strong>Get your API listed in Claude and ChatGPT.</strong><br>
  One command turns your OpenAPI spec into an MCP server with sign-in and per-user rules,
  checks it against the published requirements of the Claude Connectors Directory and the
  ChatGPT Directory, and writes your submission.
</p>

<p align="center">
  <a href="https://mcpblaze.com">Website</a> ·
  <a href="https://www.npmjs.com/package/mcpblaze">npm</a> ·
  <a href="https://www.apiblaze.com/docs">Docs</a>
</p>

<p align="center">
  <img src="assets/demo.gif" width="800" alt="Demo: one npx mcpblaze connector command makes an MCP server that is ready to submit to Claude and ChatGPT; then an AI agent signed in as Ana books a table but is denied when it tries to cancel Ben's booking">
</p>

## Quick start

**In your terminal**, run the MCPblaze CLI's `connector` command on your OpenAPI spec (a URL or
a local file). `npx` runs it, so there is nothing to install:

```bash
npx mcpblaze@latest connector https://ninopizzas.com/openapi.yaml
```

It creates the MCP server, then walks you through every open step of the Claude Connectors
Directory checklist (add `--compliant chatgpt` for the ChatGPT Directory) and links the scorecard
page where you can finish. Stop any time: `npx mcpblaze connector <name> --status` shows what is
left.

Only want the MCP server? `npx mcpblaze create --target ./openapi.yaml` makes one at
`https://<name>-<tenant>.mcpblaze.com/1.0.0/dev`. No login needed.

**In your agent**, add MCPblaze's own MCP server:

```bash
claude mcp add --transport http mcpblaze https://mcp.mcpblaze.com
```

Run `/mcp` in Claude Code and pick **mcpblaze** to sign in with GitHub. Then ask:

> Make an MCP server from https://ninopizzas.com/openapi.yaml and lock reservations to whoever
> made them. Then tell me what is left before I can submit it to the Claude Connectors Directory.

Want to see it first? `npx mcpblaze demo` runs a small booking app on your machine with an MCP
server in front of it.

## Why

An MCP server exists to let an agent *do things*: book the table, cancel the order, change the
address. Most MCP servers reach the backend with one shared credential, so **every user of the
agent can take every action on every record**. The agent acting for Alice can cancel Bob's
reservation, and nothing in the stack knows the difference.

Fixing that by hand is three projects: an OAuth login for the MCP server, a way to know *which
person* is behind each tool call, and per-record rules that check it. MCPblaze does all three
from the spec you already have:

1. **An MCP endpoint**: your API's routes as tools.
2. **Sign-in**: callers sign in (GitHub by default, or your own OAuth or OpenID Connect
   provider), so every tool call carries a real person, not a shared key.
3. **Per-user rules**: whoever *created* a record is the only one who can view, change or
   delete it. Admins can override.

No AI guesses your security. The rules come from the shape of your spec (which routes create
things, which routes touch one thing by id). The CLI shows each one as a sentence for you to
approve before anything turns on; through the MCP server, your agent locks every family it can
unless you name some, and says what it locked. Your backend does not change.

## What it looks like

```text
$ npx mcpblaze create --target https://ninopizzas.com/openapi.yaml

Create an MCP server

? MCP server name: nino
→ Your MCP server will live at https://nino-<tenant>.mcpblaze.com/1.0.0/dev

? How should callers sign in?  Sign in with an account (OAuth)
? Which sign-in?  Hosted sign-in with GitHub
? Do you want to prevent unauthorized users from taking unauthorized actions?  Yes

  Reading your OpenAPI spec…
? Lock these to the person who created them (or an admin):
  ◉ restaurants — only the person who created a restaurant, or an admin, can view or change it
  ◉ restaurants/{restaurantId}/reservations — only the person who created a reservation, or an admin, can view, change or delete it
  ◉ restaurants/{restaurantId}/tables — only the person who created a table, or an admin, can change or delete it

✔ MCP server created!
  ✓ https://nino-ninodefaulttenant.mcpblaze.com/1.0.0/dev — callers sign in with GitHub
  ✓ restaurants, reservations + tables locked to their creator

? Connect an agent to it now?  Claude Code (detected)
  ✓ Claude Code connected — it will ask you to sign in with GitHub the first time you use it
```

From then on:

| The agent, acting for Alice, tries to… | Result |
|---|---|
| book a table (`POST /restaurants/{restaurantId}/reservations`) | **200**, and reservation 42 is recorded as Alice's |
| read, change or cancel reservation 42 | **200** |
| read or cancel Bob's reservation 43 | **403**: the agent is told plainly it isn't allowed |
| the same, when Alice is an admin | **200**: admins can act on anything |

## Getting listed in Claude and ChatGPT

`npx mcpblaze connector <name>` takes an MCP server through the Claude Connectors Directory
checklist: your own
MCP hostname (one CNAME), sign-in on the connector, the listing answers pre-filled from your
spec, a reviewer account, a proof run that calls every tool, and the submission packet you paste
into each portal. It asks only for what is missing, saves every answer as you give it, and says
what is left at every step, in the terminal and on the scorecard page it links.

- `npx mcpblaze connector <name> --status` shows the scorecard and the page to finish on.
- `npx mcpblaze check <name> --directory-ready` prints the full readiness report: every blocker,
  with its fix.
- Add `--compliant chatgpt` to prepare it for the ChatGPT Directory instead.

Passing the checks means the MCP server meets the requirements we can verify. You submit.
Anthropic and OpenAI decide.

## Tools

What your agent can do once it is connected to `https://mcp.mcpblaze.com`. Making a server is one call:
`create_mcp_server` with your spec.

| Tool | What it does |
|---|---|
| `create_mcp_server` | Turns an OpenAPI spec, or the URL of an API that publishes one at `/openapi.json`, `/openapi.yaml` or `/swagger.json`, into a hosted MCP server with GitHub sign-in and API keys. It locks every record family it can to the person who created the record (or only the ones you name), prepares the server for the Claude Connectors Directory (your agent asks whether to add the ChatGPT Directory), and returns its connector URL, its scorecard page, where it stands and what it locked. |
| `list_mcp_servers` | Lists your MCP servers: name, URLs, status, and how callers get in. |
| `get_mcp_server` | Describes one MCP server in full (its URLs, how callers get in, its record families and which are locked, the directories it is prepared for, its admins) and where it stands on the way to the directories: steps done, the next step and its scorecard page. |
| `update_mcp_server` | Changes a server: its display name, target URL or on/off switch; its tools (hide, rename, describe); which directories it is prepared for; or which record families are locked. Send every family you want locked: one you leave out is unlocked, open to anyone who can call the server (a rule someone wrote by hand stays as it is), and a name that is not one of the server's families changes nothing (the answer lists the right ones). To switch every rule off and keep them, send `enforce` false on its own; `enforce` true on its own switches them back on (a switch set with the dashboard or the CLI is shared by every server on its tenant, and is changed there). |
| `delete_mcp_server` | Deletes an MCP server and everything attached to it. It needs an explicit confirmation, and it cannot be undone. |

The details (DNS, the listing fields, the reviewer account, the packet) are finished on the
scorecard page or with `npx mcpblaze connector`.

## Connect from Claude Desktop or claude.ai

`https://mcp.mcpblaze.com` uses Streamable HTTP and signs you in with GitHub through standard MCP
OAuth. In Claude Desktop or claude.ai: Customize → Connectors → Add custom connector, then paste
`https://mcp.mcpblaze.com`.

## What stays open

Rules cover *one record by id*. These stay reachable by any signed-in caller, and the CLI says so
in its output:

- Lists such as `GET /restaurants`. A list returns whole records, so if that matters to you, add
  a rule with `npx mcpblaze rule <name>`.
- Creating, such as `POST /restaurants/{restaurantId}/reservations`. Creating is how ownership
  gets recorded.
- Action-style routes such as `POST /search` or `POST /reservations/{id}/cancel`.
- Records that existed *before* the rules went on have no recorded creator, so only admins can
  touch them.

## Good to know

- **Your app runs on localhost?** `npx mcpblaze dev` puts an MCP server in front of it through a
  tunnel.
- **No account needed to start.** MCP servers made while logged out expire after 30 days unless
  you keep them: `npx mcpblaze login` offers to claim them into your account.
- **Not sure where to start?** `npx mcpblaze skills` checks your folder for an API and suggests the
  next step. Or tell your agent: *"Help me get my app listed in Claude and ChatGPT using
  npx mcpblaze@latest skills."*
- **Everything else:** `mcpblaze` runs the [`apiblaze`](https://www.npmjs.com/package/apiblaze)
  CLI under the MCP name and updates with it. `npx mcpblaze help --all` shows every command.

## About this repository

This repository holds MCPblaze's documentation and its MCP Registry entry (`server.json`). The MCP server itself
is hosted at `https://mcp.mcpblaze.com`, and the CLI is published on npm as
[`mcpblaze`](https://www.npmjs.com/package/mcpblaze). MCPblaze is built by
[APIblaze](https://www.apiblaze.com). Questions: support@apiblaze.com.

The contents of this repository are released under the [MIT License](LICENSE).
