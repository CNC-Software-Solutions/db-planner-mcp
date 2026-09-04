# Installing the DB Planner MCP server

This guide is written for an AI agent (Cline, Claude Code, Cursor and similar) that has been asked to set up DB Planner as an MCP server. Read it fully before doing anything.

## The one fact that matters

DB Planner is a **hosted** MCP server. There is nothing to download, build, clone or run. The endpoint is:

```
https://api.dbplanner.io/mcp
```

Transport is streamable HTTP. Authentication is OAuth 2.1 and happens in the user's browser on the first tool call. Do not ask the user for an API key, token or password, and do not put credentials in any configuration file. None exist for this server.

## Steps

1. Add the server to the client's MCP configuration. The `url` is the same for every client. The `type` value and the file location are not, so use the client's own shape:

   | Client | File | Entry |
   |---|---|---|
   | Claude Code | `.mcp.json` in the project | `{ "type": "http", "url": "https://api.dbplanner.io/mcp" }` |
   | Cursor | `.cursor/mcp.json` | `{ "url": "https://api.dbplanner.io/mcp" }` |
   | Cline | its `mcp.json` | `{ "type": "streamableHttp", "url": "https://api.dbplanner.io/mcp" }` |
   | Other | per its docs | the client's remote-server shape with that URL |

   Each entry goes under `mcpServers` with the key `db-planner`, for example:

   ```json
   {
     "mcpServers": {
       "db-planner": {
         "type": "http",
         "url": "https://api.dbplanner.io/mcp"
       }
     }
   }
   ```

   For Claude Code you can run this instead of editing the file:

   ```bash
   claude mcp add --transport http db-planner https://api.dbplanner.io/mcp
   ```

2. Restart or reload the client so it reads the new configuration.

3. Call the `whoami` tool. The client will open a browser window to the DB Planner consent screen. The consent decision belongs to the user, not to you: tell them the screen has opened, that they need to sign in (or create a free account at https://console.dbplanner.io), and that the screen has a checkbox controlling whether the connection may make changes. Explain that leaving it checked lets you edit their projects and unchecking it limits you to reading them, then wait for them to decide. Never attempt to interact with the consent screen yourself.

4. Once `whoami` returns the user's profile, the server is connected. Continue with `list_orgs` and `list_projects` to find the project the user wants to work in. Every other tool takes a project id.

## If something goes wrong

- **The browser never opens.** The client does not support OAuth for remote MCP servers. Claude Code, Cursor, Claude.ai, Claude Desktop and the MCP Inspector do. If the client only offers header-based authentication for remote servers, it cannot connect to this one, and there is no token to paste; tell the user which client to use instead.
- **A 401 before sign-in.** Expected. That response carries the OAuth discovery pointer the client uses to start the sign-in flow.
- **A tool says the token only grants read access.** The user chose a read-only connection on the consent screen. That was their decision, so do not work around it. Tell them the action needs write access and that granting it means removing and re-adding the server so the consent screen appears again, then leave the choice to them.
- **`resources/list` or `prompts/list` return "method not found".** Expected. The server offers tools only.

## Do not

- Do not clone this repository into the user's project. It contains documentation only.
- Do not install any package named `db-planner-mcp` from npm; there is none.
- Do not add an `Authorization` header or any `env` block. Identity comes entirely from the OAuth token the client obtains.
