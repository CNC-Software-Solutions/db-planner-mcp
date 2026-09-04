# DB Planner MCP server

Hosted MCP server for [DB Planner](https://dbplanner.io) that lets any MCP client edit database schemas (DBML/SQL), Mermaid diagrams and board layouts. OAuth sign-in, nothing to install.

```
https://api.dbplanner.io/mcp
```

This repository is documentation. The server is built into the DB Planner API and there is no code to run: add the URL to your client, sign in once in the browser, and your agent has 40 tools over your projects.

## What is DB Planner

DB Planner is a collaborative database design tool. You write schemas in DBML or import SQL dumps (PostgreSQL, MySQL and SQL Server), sketch diagrams in Mermaid, and lay both out on a shared board that renders relationships, notes and drawings in real time. The [template marketplace](https://dbplanner.io/templates) is where published schemas are shared.

## Connect

### Claude Code

```bash
claude mcp add --transport http db-planner https://api.dbplanner.io/mcp
```

### Cursor and other clients with a JSON config

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

The `url` is the same everywhere; the `type` value is not. Claude Code's `.mcp.json` takes `http` as above, Cursor's `.cursor/mcp.json` needs no `type` at all, and Cline's `mcp.json` takes `streamableHttp`. Check your client's documentation for its remote-server shape.

### Claude.ai and Claude Desktop

Settings, then Connectors, then Add custom connector. Paste the URL. Leave authentication on "Always required" and the OAuth client on "register one automatically". No headers.

### MCP Inspector

```bash
npx @modelcontextprotocol/inspector
```

Click Add Servers, then fill in the form: server ID `db-planner`, transport `streamable-http`, URL `https://api.dbplanner.io/mcp`. Add it, switch it on, and the consent screen opens in the browser.

On every client the first tool call opens the DB Planner consent screen. Sign in, pick the scopes, and you are connected. No password or API key is stored in the client configuration. An agent setting this up on your behalf should read [`llms-install.md`](llms-install.md).

## Authentication

The endpoint is protected by OAuth 2.1 with PKCE. Discovery is standard, so clients configure themselves from the URL alone:

- `https://api.dbplanner.io/.well-known/openid-configuration`
- `https://api.dbplanner.io/.well-known/oauth-protected-resource`
- Dynamic client registration at `https://api.dbplanner.io/connect/register`, for both public clients (`token_endpoint_auth_method=none`) and confidential ones (`client_secret_post`, `client_secret_basic`)

Two scopes are the whole permission model, and the split is enforced on the server, not in the client:

| Scope | Grants |
|---|---|
| `dbplanner:read` | Every read tool. Always granted, because the endpoint requires it. |
| `dbplanner:write` | The mutating tools, checked per tool. Also covers publishing to the public marketplace. |

MCP clients ask for both. Read is always granted, and write is a checkbox on the consent screen, on by default. Turning it off means the token is issued without write even though the client requested it, so a read-only connection is a real guarantee rather than a client-side courtesy.

Clients also request `offline_access`. That is a session mechanism rather than a permission: it yields a refresh token so the connection survives token expiry, and it is granted whenever asked.

A tool never sees more than the signed-in user can see in the app: organization and project roles are resolved from the token's subject and enforced by the same services the web app uses.

## Tools

Every tool carries a title, a read-only or destructive hint, an `openWorldHint` of false, a description on every parameter and an output schema, so clients can run reads without confirmation prompts and read structured results instead of parsing text.

| Group | Read | Write |
|---|---|---|
| Session | `whoami`, `get_public_profiles` | |
| Organizations | `list_orgs`, `get_org` | `create_org`, `rename_org`, `delete_org`, `upsert_org_member`, `remove_org_member` |
| Projects | `list_projects`, `project_file_counts` | `create_project`, `update_project`, `delete_project`, `upsert_project_member`, `remove_project_member` |
| Documents | `list_documents`, `list_diagrams` | `create_document`, `update_document`, `delete_document`, `reorder_documents`, `set_table_color` |
| Board | `get_board`, `get_board_image`, `list_tables`, `list_drawings` | `update_board`, `draw_stroke`, `draw_arrow`, `erase_drawings`, `add_board_image` |
| Marketplace | `browse_templates`, `get_template`, `template_categories`, `my_listings` | `use_template`, `publish_listing`, `update_listing`, `unpublish_listing` |

A few things worth knowing before an agent calls them:

- **Design lives in documents, never on the board.** Tables, columns, references, enums, classes and flowchart steps are derived by parsing a document, so the design changes through `create_document` and `update_document` and the board follows. There is no tool that creates a table directly.
- **`get_board_image` renders the board to a PNG** so the model can look at a layout rather than infer it from coordinates. Pass `format='svg'` for the vector source as text.
- **`update_board` takes a partial delta.** Send only the card ids you are changing; a null value deletes one. It is the same merge contract the app uses, so an agent moving one card does not clobber a collaborator's layout.
- **Publishing is creator-only, and no tool takes a price.** Only the person who created a project may list it, and a listing published through MCP is always free. Pricing requires Stripe onboarding that nobody can do on someone's behalf.

## Where it is listed

- [Official MCP Registry](https://registry.modelcontextprotocol.io/) as `io.dbplanner/db-planner`
- [Smithery](https://smithery.ai/servers/dbplanner/db-planner)
- [Glama](https://glama.ai/mcp/servers)
- [mcp.so](https://mcp.so)
- [cursor.directory](https://cursor.directory)

## Support

Questions and problems: [support@dbplanner.io](mailto:support@dbplanner.io). Product documentation is at [dbplanner.io/features](https://dbplanner.io/features).

## License

This documentation is released under the [MIT License](LICENSE). DB Planner itself is a hosted product; see [dbplanner.io/pricing](https://dbplanner.io/pricing).
