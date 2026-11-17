<p align="center">
  <img src="assets/owlbear.png" width="160" alt="The MCPblaze owlbear, hugging the logo">
</p>

<h1 align="center">MCPblaze</h1>

<p align="center">
  <strong>Get your API into Claude and ChatGPT.</strong><br>
  Point it at your OpenAPI spec and you get an MCP server with sign-in and per-user rules.
  Then it checks it against what the Claude and ChatGPT directories ask for and writes up
  your submission.
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

Got an OpenAPI spec (a URL or a file)? Run this. Nothing to install:

```bash
npx mcpblaze@latest connector https://ninopizzas.com/openapi.yaml
```

That makes your MCP server, then walks you through whatever's left on the Claude Connectors
Directory checklist. Want the ChatGPT Directory instead? Add `--compliant chatgpt`. You can stop
any time and see what's left with `npx mcpblaze connector <name> --status`.

Just want the MCP server and nothing else?

```bash
npx mcpblaze create --target ./openapi.yaml
```

You get an address like `https://<name>-<tenant>.mcpblaze.com`. No login needed.

### Or do it from your agent

Add MCPblaze's own MCP server:

```bash
claude mcp add --transport http mcpblaze https://mcp.mcpblaze.com
```

Run `/mcp` in Claude Code, pick **mcpblaze** and sign in with GitHub. Then just ask:

> Make an MCP server from https://ninopizzas.com/openapi.yaml and lock reservations to whoever
> made them. Then tell me what's left before I can submit it to the Claude Connectors Directory.

Want to try it first? `npx mcpblaze demo` runs a small booking app on your machine with an MCP
server in front of it.

## Why

The whole point of an MCP server is letting an agent actually do stuff: book a table, cancel an
order, change an address. But most MCP servers talk to the backend with one shared key, so
**anyone using the agent can do anything to any record**. The agent working for Ana can cancel
Ben's reservation and nothing stops it.

Fixing that yourself is 3 separate projects: an OAuth login for the MCP server, a way to know
which person is behind each tool call, and per-record rules that actually check it. MCPblaze does
all 3 from the spec you already have:

1. **An MCP endpoint.** Your API's routes become tools.
2. **Sign-in.** People sign in (GitHub by default, or your own OAuth or OpenID Connect provider),
   so every tool call comes from a real person, not a shared key.
3. **Per-user rules.** Only whoever *created* a record can view, change or delete it. Admins can
   get to everything.

No AI is guessing at your security. The rules come from how your spec is laid out (which routes
create things, which ones touch one thing by id). The CLI shows you each rule as a plain sentence
and you approve it before anything turns on. Through the MCP server, your agent locks everything
it can unless you tell it which ones, and tells you what it locked. You don't have to change your
backend.

## What it looks like

```text
$ npx mcpblaze create --target https://ninopizzas.com/openapi.yaml

Create an MCP server

? MCP server name: nino
→ Your MCP server will live at https://nino-<tenant>.mcpblaze.com

  Uncheck everything to skip.
? Should only creators of a resource in the API be able to amend that resource?
  ◉ restaurants — only the person who created a restaurant, or an admin, can view or change it
  ◉ restaurants/{restaurantId}/reservations — only the person who created a reservation, or an admin, can view, change or delete it
  ◉ restaurants/{restaurantId}/tables — only the person who created a table, or an admin, can change or delete it
? Who is the admin? Their email address (Enter to skip): nino@ninopizzas.com

✔ MCP server created!

  ✓ MCP server nino: your backend and widget use the API key; people and agents use an MCPblaze login (with GitHub)
  ✓ Each person can only change the restaurants, reservations + tables they made (lists stay open: your backend filters them)
  ✓ Backend key (shown once, save it): sk_…
? Connect Claude Code to it now?  Yes
? How should Claude Code authenticate to it?  Sign in with GitHub
  ✓ Claude Code connected — it will ask you to sign in with GitHub the first time you use it
  ✓ admin: nino@ninopizzas.com — active on first sign-in
```

After that:

| The agent, working for Ana, tries to… | What happens |
|---|---|
| book a table (`POST /restaurants/{restaurantId}/reservations`) | **200**, and reservation 42 is saved as Ana's |
| read, change or cancel reservation 42 | **200** |
| read or cancel Ben's reservation 43 | **403**, and the agent is told straight up it's not allowed |
| the same thing, working for Nino (the admin) | **200**, admins can get to anything |

## Getting listed in Claude and ChatGPT

`npx mcpblaze connector <name>` takes your MCP server through the Claude Connectors Directory
checklist:

- your own MCP hostname (one CNAME)
- sign-in on the connector
- the listing answers, pre-filled from your spec
- a reviewer account
- a test run that calls every tool
- the submission packet you paste into each portal

It only asks for what's missing, saves every answer as you go, and tells you what's left at every
step, both in the terminal and on the scorecard page it links you to.

- `npx mcpblaze connector <name> --status` shows the scorecard and the page to finish on.
- `npx mcpblaze check <name> --directory-ready` prints the full readiness report, with a fix for
  every blocker.
- Add `--compliant chatgpt` to get it ready for the ChatGPT Directory instead.

Heads up: passing our checks means your MCP server meets everything we can verify. You still
submit it yourself, and Anthropic and OpenAI make the call.

## Tools

What your agent can do once it's connected to `https://mcp.mcpblaze.com`. Making a server is just
one call: `create_mcp_server` with your spec.

| Tool | What it does |
|---|---|
| `create_mcp_server` | Turns an OpenAPI spec into a hosted MCP server with GitHub sign-in and API keys. You can also just give it your API's URL if it publishes a spec at `/openapi.json`, `/openapi.yaml` or `/swagger.json`. It locks every record family it can to whoever created the record (or just the ones you name), gets it ready for the Claude Connectors Directory (your agent asks if you want ChatGPT too), and gives you back the connector URL, the scorecard page, where it stands and what it locked. |
| `list_mcp_servers` | Lists your MCP servers, 20 at a time: name, connector URL, whether it's on, and how callers get in. |
| `get_mcp_server` | Everything about one MCP server: its URLs, how callers sign in, its record families and which ones are locked, its admins, keys and limits. Plus where it stands on the way to the directories: what's done, what's next and its scorecard page. |
| `update_mcp_server` | Changes one server: display name, the API URL behind it, on or off, its tools (hide, rename, describe), which record families are locked, or which directories it's prepared for. Anything you leave out stays the same. Families and directories are full lists though, so send every family you want locked. Leave one out and it gets unlocked (rules someone wrote by hand stay). A family name that doesn't exist changes nothing, and the answer lists the real ones. To turn the rules off but keep them, send `enforce: false`; `enforce: true` turns them back on. If the switch was flipped from the dashboard or the CLI, that one covers every server on the tenant, so change it there. |
| `delete_mcp_server` | Deletes an MCP server and everything attached to it (keys, rules and logs). Your agent has to check with you first, and there's no undo. |

The rest (DNS, the listing fields, the reviewer account, the packet) gets finished on the
scorecard page or with `npx mcpblaze connector`.

## Connect from Claude Desktop or claude.ai

`https://mcp.mcpblaze.com` uses Streamable HTTP, and you sign in with GitHub through standard MCP
OAuth. In Claude Desktop or claude.ai, go to Customize → Connectors → Add custom connector and
paste `https://mcp.mcpblaze.com`.

## What stays open

Rules cover *one record by its id*. Anyone who's signed in can still reach these, and the CLI
tells you so:

- Lists, like `GET /restaurants`. A list returns whole records, so if that matters to you, filter
  them in your backend or add a rule with `npx mcpblaze rule <name>`.
- Creating, like `POST /restaurants/{restaurantId}/reservations`. That's how ownership gets
  recorded in the first place.
- Action-style routes, like `POST /search` or `POST /reservations/{id}/cancel`.
- Records that existed *before* the rules went on don't have a creator saved, so only admins can
  touch them.

## Good to know

- **Your app runs on localhost?** `npx mcpblaze dev` puts an MCP server in front of it through a
  tunnel.
- **No account needed to start.** MCP servers made while logged out expire after 30 days unless
  you keep them. `npx mcpblaze login` offers to claim them into your account.
- **Not sure where to start?** `npx mcpblaze skills` checks your folder for an API and suggests
  what to do next. Or tell your agent: *"Help me get my app listed in Claude and ChatGPT using
  npx mcpblaze@latest skills."*
- **Everything else:** `mcpblaze` runs the [`apiblaze`](https://www.npmjs.com/package/apiblaze)
  CLI under the MCP name and updates along with it. `npx mcpblaze help --all` shows every command.

## About this repo

This repo holds MCPblaze's docs and its MCP Registry entry (`server.json`). The MCP server itself
is hosted at `https://mcp.mcpblaze.com`, and the CLI is on npm as
[`mcpblaze`](https://www.npmjs.com/package/mcpblaze). MCPblaze is built by
[APIblaze](https://www.apiblaze.com). Questions? support@apiblaze.com.

Everything in this repo is under the [MIT License](LICENSE).
