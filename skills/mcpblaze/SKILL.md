---
name: mcpblaze
description: Makes an MCP server for the API in the current folder with MCPblaze and hands the user its MCP and Scorecard links, then, when asked, gets it ready to be listed in the Claude and ChatGPT directories. Use when the user wants an MCP server for their API, a Claude connector, a ChatGPT app, sign-in or per-user rules for an MCP, or asks what MCPblaze does.
metadata:
  brand: mcpblaze
---

# MCPblaze

Run `npx mcpblaze@latest skills` in the project folder and do exactly what it prints. It checks the folder and prints the one
phase it is in: first the MCP server and its two links for the user, then (only when they ask) getting it ready to be
listed in Claude and ChatGPT.
