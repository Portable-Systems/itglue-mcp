# IT Glue MCP Server

A Model Context Protocol (MCP) server that provides Claude with access to IT Glue documentation and asset management.

> **Important:** This repository began as a fork of another project but has since been substantially modified for our own internal workflows and may differ significantly from the original. It is published as-is for reference and reuse, without any guarantee that it will suit other environments. Review the code, security implications, and configuration carefully, and test it thoroughly before use. You use this software at your own risk.

## One-Click Deployment

[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/wyre-technology/itglue-mcp/tree/main)

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/wyre-technology/itglue-mcp)

## Installation

```bash
npm install @wyre-technology/itglue-mcp
```

Or use the Docker image:

```bash
docker pull ghcr.io/wyre-technology/itglue-mcp:latest
```

## Configuration

The server accepts credentials via environment variables:

| Variable | Description | Required |
|----------|-------------|----------|
| `ITGLUE_API_KEY` | Your IT Glue API key (format: ITG.xxx) | Yes |
| `ITGLUE_REGION` | API region: `us`, `eu`, or `au` (default: `us`) | No |

Alternative: The MCP Gateway can inject credentials via `X_API_KEY` header.

## Available Tools

Tools named `search_*` retain their public names for compatibility, but IT Glue's index endpoints do not provide useful fuzzy name search. They return up to 50 records per page. Filter the returned records locally by name, and pass `page_number` only when the response metadata contains a `nextPage`. The page size is intentionally server-controlled to avoid oversized tool results.

### Organizations

- **search_organizations** - List an organization index page, optionally narrowed by type, status, or PSA ID
- **get_organization** - Get a specific organization by ID

### Configurations (Devices/Assets)

- **search_configurations** - List a configuration index page, optionally narrowed by organization, type, status, serial number, RMM ID, or PSA ID
- **get_configuration** - Get a specific configuration by ID

### Passwords

- **search_passwords** - List a password-entry index page (metadata only, with no actual password values)
- **get_password** - Get a specific password entry; metadata is returned by default, and `show_password: true` explicitly includes the password value

### Documents

- **search_documents** - List standard-document previews for an organization. Omitting `document_folder_id` lists documents outside the root folder; use `0` for the root folder or a positive ID for one exact folder
- **read_document_html** - Read all sections of one standard document as combined HTML

### Flexible Assets

- **list_flexible_asset_types** - List flexible asset types and discover the ID required by `search_flexible_assets`
- **search_flexible_assets** - List a flexible-asset index page for one required `flexible_asset_type_id`; flexible assets are separate from standard documents

### Utility

- **itglue_health_check** - Verify connectivity to IT Glue API

## Usage with Claude Code

Add to your `.mcp.json`:

```json
{
  "mcpServers": {
    "itglue": {
      "command": "npx",
      "args": ["@wyre-technology/itglue-mcp"],
      "env": {
        "ITGLUE_API_KEY": "${ITGLUE_API_KEY}",
        "ITGLUE_REGION": "us"
      }
    }
  }
}
```

Or with Docker:

```json
{
  "mcpServers": {
    "itglue": {
      "command": "docker",
      "args": ["run", "--rm", "-i", "-e", "ITGLUE_API_KEY", "ghcr.io/wyre-technology/itglue-mcp:latest"],
      "env": {
        "ITGLUE_API_KEY": "${ITGLUE_API_KEY}"
      }
    }
  }
}
```

## Example Queries

Once configured, you can ask Claude:

- "Search for organizations containing 'Acme' in IT Glue"
- "Get the configuration details for device ID 12345"
- "Find all passwords for organization ID 100"
- "Search for flexible assets of type 54321"

## Security Notes

- Password search results do not include actual password values for security
- Use `get_password` with explicit ID to retrieve password values
- Store your API key securely using environment variables or a secrets manager
- The API key should have appropriate read permissions in IT Glue

## License

Apache-2.0

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.
