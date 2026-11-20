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

**1. Connect it**

```bash
claude mcp add --transport http mcpblaze https://mcp.mcpblaze.com
```

Run `/mcp` in Claude Code, pick **mcpblaze** and sign in with GitHub.

In Claude Desktop or claude.ai: Customize → Connectors → Add custom connector, then paste
`https://mcp.mcpblaze.com`. Other MCP apps may need a one-time OK from us first (support@apiblaze.com).

**2. Ask for an MCP server**

> Make an MCP server from https://ninopizzas.com/openapi.yaml and lock reservations to whoever
> made them. Then tell me what's left before I can submit it to the Claude Connectors Directory.

You get back its address, a Scorecard link and what it locked. Your agent will ask if you want it
ready for ChatGPT too. No spec file? Just give it your API's URL, as long as the spec is at
`/openapi.json`, `/openapi.yaml` or `/swagger.json`.

**3. Finish on the Scorecard page**

Click the Scorecard link and sign in. It opens your MCP server in the dashboard:

- **MCP config:** how well AI apps can use your tools, what to fix first, Edit tools and the Tool builder.
- **Listing on Claude and ChatGPT:** the directory checklist, one step at a time.

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

With the rules on:

| The agent, working for Ana, tries to… | What happens |
|---|---|
| book a table | **works**, saved as Ana's reservation 42 |
| open, change or cancel reservation 42 | **works** |
| open or cancel Ben's reservation 43 | **403**, and it's told it's not allowed |
| the same, working for Nino (the admin) | **works**, admins can get to anything |

<p align="center">
  <img src="assets/demo.gif" width="800" alt="Demo: an OpenAPI spec becomes an MCP server ready to submit; then Ana's agent books a table but can't cancel Ben's booking">
</p>

## Tools

| Tool | What it does |
|---|---|
| `create_mcp_server` | Turns your OpenAPI spec (or your API's URL) into a hosted MCP server with sign-in, API keys and per-user rules, and starts its Claude directory prep. Gives back its address, the Scorecard link and what it locked. |
| `list_mcp_servers` | Your MCP servers, 20 at a time: name, address, on or off, how people get in. |
| `get_mcp_server` | One server in full, including sign-in, locked record types, where it stands for the directories, and how well AI apps can use its tools. |
| `update_mcp_server` | Changes the display name, API URL, on or off, the spec (new routes become tools and get locked), tool text and hints, locked record types, directories or sign-in. What you leave out stays. |
| `delete_mcp_server` | Deletes a server and everything attached to it (keys, rules and logs). Checks with you first. No undo. |

Heads up: `families` and `directories` are whole lists, so a record type you leave out gets unlocked.
`enforce: false` turns the rules off but keeps them (if you turned them on in the CLI or the
dashboard, turn them off there). These tools work on a project's first MCP server.

## Getting listed in Claude and ChatGPT

The checklist covers your own MCP hostname (one CNAME), tool descriptions, sign-in and a reviewer
account, the listing answers, a test run, one sign-in from your own Claude, then the packet you
submit.

You'll need an account, your app on a public https address (not a tunnel) and a privacy policy page.

Passing our checks means you meet everything we can verify. You submit it, and Anthropic and OpenAI
decide.

## Make your tools easy for AI apps

On the Scorecard page, **How well AI apps can use it** checks your tools and tells you what to fix
first. It's our own check, not Claude's or ChatGPT's, but a few things it flags (like tool names over
64 characters) also block a listing.

- **Short, clear errors** sends a failed call back to the AI as one short line, not a dump.
- **Test with AI models** asks a Claude model and a GPT model which tool they'd pick for your test
  cases. Free tries first, then credits.

Your agent can help too: `get_mcp_server` shows the same checks, and `update_mcp_server` can rewrite
tool and input text.

## Sign-in

By default people sign in with GitHub (our GitHub app) and your backend uses an API key. The first
backend key you get expires after 14 to 30 days.

Want your own Google, GitHub, Microsoft, Auth0 or OpenID Connect app? Add
`https://callback.mcpblaze.com` to its callback URLs (a GitHub app takes only one, so use a new
one), then tell your agent. Don't paste the client secret in the chat: your agent gives you a
one-time link for it. Switching later keeps our GitHub sign-in working until someone signs in with
your app.

## What stays open

Rules cover *one record by its id*. Anyone signed in can still reach:

- lists like `GET /restaurants` (filter them in your backend)
- creating, like `POST /restaurants/{restaurantId}/reservations` (that's how the creator gets saved)
- action routes like `POST /search` or `POST /orders/{id}/refund`
- records your spec has no create route for, like `PATCH /profiles/{id}` with no `POST /profiles`
- record types we can't lock safely (`get_mcp_server` tells you which and why)

Records made before the rules (or while they were off) have no creator, so only admins can touch
them. Got a new version of your spec? Send it through `update_mcp_server` and the new routes get
locked too.

## More than one MCP server?

A project can have up to 5, each with its own tools, address and listing. They share the API,
sign-in, users and rules. Add one in the dashboard (MCP config → Advanced) or with
`npx mcpblaze mcp servers add <name> --name "Back office"`. Needs an account. That's also why a
project name can't end in `mcp`.

## Prefer a terminal?

```bash
npx mcpblaze@latest create --target https://ninopizzas.com/openapi.yaml
```

Nothing to install, no login needed. You get the address and the Scorecard link, same as above.
Logged out, it picks the name for you (or pass `--name`).

- `npx mcpblaze ready <name>` does the listing checklist in the terminal. It needs an account, and
  it switches your server to listing mode right away (short errors, long answers cut). Just looking?
  Add `--status`. Add `--compliant chatgpt` for ChatGPT, or `--compliant all` for both. Its old name,
  `connector`, still works.
- `npx mcpblaze dev` does the same as `create` for an app running on your machine.
- `npx mcpblaze --help` shows the rest.

Logged out, your MCP servers share one workspace. It's deleted after 3 days if none of them gets a
call, or after 30 days with no calls. Log in within 30 days (`npx mcpblaze login`) to keep it.

## About this repo

MCPblaze's docs and its `server.json` for the official MCP Registry. Built by
[APIblaze](https://www.apiblaze.com). Questions? support@apiblaze.com.

Everything in this repo is under the [MIT License](LICENSE).
