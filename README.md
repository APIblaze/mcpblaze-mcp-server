<p align="center">
  <img src="assets/owlbear.png" width="160" alt="The MCPblaze owlbear, hugging the logo">
</p>

<h1 align="center">MCPblaze</h1>

<p align="center">
  <strong>Serverless MCP with OAuth and more.</strong><br>
  Point it at your OpenAPI spec and you get a hosted MCP server with sign-in and per-user rules.
  Then it gets you ready for the Claude and ChatGPT directories.
</p>

<p align="center">
  <a href="https://mcpblaze.com">Website</a> ·
  <a href="https://www.npmjs.com/package/mcpblaze">npm</a> ·
  <a href="https://dashboard.apiblaze.com">Dashboard</a>
</p>

## Quick start

Got an OpenAPI spec (a URL or a file)? Run this. Nothing to install, no login needed:

```bash
npx mcpblaze@latest create --target https://ninopizzas.com/openapi.yaml
```

You get your MCP server's address and a Scorecard link (sign in there to see it). That's step 1 of 4:

| Step | Command | What you get |
|---|---|---|
| 1 | `npx mcpblaze create --target <spec>` | your MCP server and its Scorecard |
| 2 | `npx mcpblaze build <name>` | one tool built around what people ask for |
| 3 | `npx mcpblaze tools <name>` | keep only the tools people would ask for |
| 4 | `npx mcpblaze ready <name>` | the directory checklist, then the packet you submit |

`build` chains your API's calls into one tool for what people want most, like "all of a diner's
reservations, from just an email". No AI involved, and every call still runs as the signed-in person.

App only on your machine? Use `npx mcpblaze dev` for step 1. In a hurry? `npx mcpblaze ready <spec>`
does step 1 and jumps to step 4 (that part needs an account). `connector`, its old name, still works.

<p align="center">
  <img src="assets/demo.gif" width="800" alt="Demo: an OpenAPI spec becomes an MCP server ready to submit; then Ana's agent books a table but can't cancel Ben's booking">
</p>

### Or do it from your agent

```bash
claude mcp add --transport http mcpblaze https://mcp.mcpblaze.com
```

Run `/mcp` in Claude Code, pick **mcpblaze** and sign in with GitHub. Then just ask:

> Make an MCP server from https://ninopizzas.com/openapi.yaml and lock reservations to whoever
> made them. Then tell me what's left before I can submit it to the Claude Connectors Directory.

In Claude Desktop or claude.ai: Customize → Connectors → Add custom connector, then paste
`https://mcp.mcpblaze.com`. Other MCP apps may need a one-time OK from us first (support@apiblaze.com).

## Why

An MCP server lets an agent actually do stuff: book a table, cancel an order. But an MCP server that
calls your backend with one shared key lets **anyone using the agent do anything to any record**.
The agent working for Ana can cancel Ben's reservation.

MCPblaze fixes that from the spec you already have:

1. **An MCP server.** Your API's routes become tools.
2. **Sign-in.** GitHub by default, or your own OAuth app. Every signed-in call comes from a real person.
3. **Per-user rules.** Only whoever *created* a record can open, change or delete it. Admins can get
   to everything. Lists stay open (see below).

No AI guesses at your security. The rules come from how your spec is laid out, and each one reads
as a plain sentence. Your backend only changes if you want lists filtered.

## What it looks like

A logged-in run, trimmed a bit:

```text
$ npx mcpblaze create --target https://ninopizzas.com/openapi.yaml

? MCP server name: nino
? Should only creators of a resource in the API be able to amend that resource?
❯◉ restaurants — only the person who created a restaurant, or an admin, can view or change it
 ◉ restaurants/{restaurantId}/reservations — only the person who created a reservation, or an admin, can view, change or delete it
 ◉ restaurants/{restaurantId}/tables — only the person who created a table, or an admin, can change or delete it
? Who is the admin? Their email address (Enter to skip): nino@ninopizzas.com
✔ MCP server created!
  MCP server:  https://nino-<tenant>.mcpblaze.com
  Scorecard:   https://dashboard.apiblaze.com/directory/nino/1.0.0?…

  ✓ Each person can only change the restaurants, reservations + tables they made (lists stay open: your backend filters them)
  ✓ Backend key (shown once, save it): sk_…
```

Logged out, it picks the name for you (or pass `--name`). With the rules on:

| The agent, working for Ana, tries to… | What happens |
|---|---|
| book a table | **works**, saved as Ana's reservation 42 |
| open, change or cancel reservation 42 | **works** |
| open or cancel Ben's reservation 43 | **403**, and it's told it's not allowed |
| the same, working for Nino (the admin) | **works**, admins can get to anything |

## Getting listed in Claude and ChatGPT

`npx mcpblaze ready <name>` walks you through the Claude Connectors Directory checklist: your own MCP
hostname (one CNAME), tool descriptions, sign-in and a reviewer account, the listing answers, a test
run, one sign-in from your own Claude, then the packet you submit.

It switches your server to listing mode right away (short errors, long answers cut) and may hide
utility routes like `/health` itself. Then it asks only for what's missing. Just looking? `--status`
shows what's left. You'll need an account, your app on a public https address (not a `dev` tunnel)
and a privacy policy page.

- `npx mcpblaze check <name> --directory-ready` prints the full report, with a fix for every blocker.
- Add `--compliant chatgpt` for the ChatGPT Directory instead, or `--compliant all` for both.

Passing our checks means you meet everything we can verify. You submit it, and Anthropic and OpenAI
decide.

## Make your tools easy for AI apps

`npx mcpblaze mcp scorecard <name>` checks how well AI apps can use your tools and what to fix first.
It's our own check, not Claude's or ChatGPT's, though some findings (like tool names over 64
characters) also block a listing.

- `--fix short-errors` sends a failed call back as one short line, not a dump.
- `--test` asks a Claude model and a GPT model which tool they'd pick for your test cases (you write
  those on the Scorecard page). Needs an account. Free tries first, then credits.

Want your own wording? `npx mcpblaze mcp ops set <name> <tool> --input <input> --describe "…"`. Or
do all of it on the **MCP server** tab in the dashboard.

## Sign-in

By default people sign in with GitHub (our GitHub app) and your backend uses an API key. The key
`create` gives you expires after 14 to 30 days.

Want your own Google, GitHub, Microsoft, Auth0 or OpenID Connect app? Add
`https://callback.mcpblaze.com` to its callback URLs (a GitHub app takes only one, so use a new
one), then tell your agent or run `npx mcpblaze tenant manage`. Don't paste the client secret in a
chat: your agent gives you a one-time link for it. Switching later through your agent keeps our
GitHub sign-in working until someone signs in with your app.

## Tools at mcp.mcpblaze.com

| Tool | What it does |
|---|---|
| `create_mcp_server` | Turns your OpenAPI spec (or your API's URL, if it serves its spec at `/openapi.json`, `/openapi.yaml` or `/swagger.json`) into a hosted MCP server with sign-in, API keys and per-user rules, and starts its Claude directory prep. Gives back its address, the Scorecard page and what it locked. |
| `list_mcp_servers` | Your MCP servers, 20 at a time: name, address, on or off, how people get in. |
| `get_mcp_server` | One server in full, including sign-in, locked record types, where it stands for the directories, and how well AI apps can use its tools. |
| `update_mcp_server` | Changes the display name, API URL, on or off, the spec (new routes become tools and get locked), tool text and hints, locked record types, directories or sign-in. What you leave out stays. |
| `delete_mcp_server` | Deletes a server and everything attached to it (keys, rules and logs). Checks with you first. No undo. |

Heads up: `families` and `directories` are whole lists, so a record type you leave out gets unlocked.
`enforce: false` turns the rules off but keeps them (if you turned them on in the CLI or the
dashboard, turn them off there).

## What stays open

Rules cover *one record by its id*. Anyone signed in can still reach:

- lists like `GET /restaurants` (filter them in your backend)
- creating, like `POST /restaurants/{restaurantId}/reservations` (that's how the creator gets saved)
- action routes like `POST /search` or `POST /orders/{id}/refund`
- records your spec has no create route for, like `PATCH /profiles/{id}` with no `POST /profiles`
- record types we can't lock safely (the CLI checklist and `get_mcp_server` say which and why)

Records made before the rules (or while they were off) have no creator, so only admins can touch
them. New spec via `update_mcp_server`? New routes get locked. Via `npx mcpblaze spec set` or the
dashboard? They come in open: run `npx mcpblaze rule <name>` in a terminal and tick them.

## Good to know

- **Up to 5 MCP servers on one API.** `npx mcpblaze mcp servers add nino --name "Back office"` makes
  `ninomcp2`, with its own tools (none at first, or `--copy` the first one's), address and listing.
  Same API, sign-in, users and rules. Needs an account. That's why a name can't end in `mcp`.
- **No account needed to start.** Logged out, your MCP servers share one workspace. It's deleted
  after 3 days if none of them gets a call, or after 30 days with no calls. Log in within 30 days
  (`npx mcpblaze login`) to keep it.
- **Not sure where to start?** Run `npx mcpblaze skills`, or tell your agent: *"Help me get my app
  listed in Claude and ChatGPT using npx mcpblaze@latest skills."*
- **Everything else:** `npx mcpblaze --help` for the main commands, `npx mcpblaze help --all` for the
  rest.

## About this repo

MCPblaze's docs and its `server.json` for the official MCP Registry. Built by
[APIblaze](https://www.apiblaze.com). Questions? support@apiblaze.com.

Everything in this repo is under the [MIT License](LICENSE).
