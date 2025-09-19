# MCP Publish Action

Generic GitHub Action to publish your MCP server to the MCP Registry. The action constructs server.json behind the scenes using the official schema and publishes it via the MCP Publisher CLI with GitHub OIDC.

Schema reference: docs/reference/server-json/server.schema.json in the MCP Registry repository.

## Minimal Usage
Trigger on tags and grant `id-token: write`. The action accepts only the schema-required fields and transport/package essentials.

```yaml
name: Publish MCP Server

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - name: Publish to MCP Registry (PyPI, stdio)
        uses: OtherVibes/mcp-publish-action@v1
        with:
          name: io.github.owner/repo
          description: "MCP server providing weather data and forecasts"
          version: "1.0.0"
          registry_type: pypi
          identifier: my-server
          transport_type: stdio
          # Optional: website_url: https://example.com
          # Optional: registry_base_url: https://pypi.org
```

Notes
- **Version handling**: Use semantic versioning (e.g., `"1.0.0"`, `"2.1.0-alpha"`). For tag-based releases, you can use `${{ github.ref_name }}` but ensure your tags are proper versions (not branch names).
- **Repository metadata**: Auto-populated using the current GitHub repo URL and `source: github`.
- For remote transports, use:
  - transport_type: streamable-http and provide transport_url
  - transport_type: sse and provide transport_url
- Authentication: uses GitHub Actions OIDC tokens (github-oidc method) which works automatically with the id-token: write permission

## Inputs

| Input | Required | Type | Default | Description |
|-------|----------|------|---------|-------------|
| `name` | ✅ | string | | Server name in reverse-DNS format (must contain exactly one forward slash separating namespace from server name) |
| `description` | ✅ | string | | Clear human-readable explanation of server functionality (should focus on capabilities, not implementation details) |
| `version` | ✅ | string | | Version string for this server (should follow semantic versioning, e.g., "1.0.2", "2.1.0-alpha") |
| `registry_type` | ✅ | choice | | Registry type indicating how to download packages<br/>**Options:** `pypi`, `npm`, `oci`, `nuget`, `mcpb` |
| `identifier` | ✅ | string | | Package identifier - either a package name (for registries) or URL (for direct downloads) |
| `transport_type` | ✅ | choice | | Transport type for the package<br/>**Options:** `stdio`, `streamable-http`, `sse` |
| `transport_url` | | string | | Transport URL (required for streamable-http and sse transports) |
| `website_url` | | string | | Optional URL to the server homepage, documentation, or project website |
| `registry_base_url` | | string | | Base URL of the package registry |
| `package_version` | | string | | Package version (must be a specific version, defaults to top-level version) |
| `file_sha256` | | string | | SHA-256 hash of the package file for integrity verification (required for MCPB packages) |
| `status` | | choice | `active` | Server lifecycle status<br/>**Options:** `active`, `deprecated`, `deleted` |
| `registry_url` | | string | `https://registry.modelcontextprotocol.io` | Registry base URL for publishing and verification |
| `verify` | | boolean | `true` | Validates the published version is visible in the registry |

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `version` | string | Published version |
| `server_name` | string | Fully qualified server identifier |
| `server_json` | string | The JSON used for publication |

## Requirements
- Ubuntu runner with curl and tar; jq will be installed if missing
- Workflow permission id-token: write for GitHub OIDC
