# Discovery & Registration Mechanisms Analysis

## MCP OAuth 2.1 Discovery (Well-Known Protected Resource Metadata)
**How servers are discovered**: MCP clients call the server’s OAuth Protected Resource Metadata endpoint at `/.well-known/oauth-protected-resource/mcp`, receive a 401 with a `WWW-Authenticate` header pointing to the authorization server, and then use the metadata for discovery. 【F:OAuth 2.1 for MCP Servers/README.md†L12-L53】【F:OAuth 2.1 for MCP Servers/server.py†L27-L48】【F:OAuth 2.1 for MCP Servers/auth.py†L34-L61】
**Configuration/registration flow**: Clients can use Dynamic Client Registration to register automatically. During registration they provide name, type, redirect URLs, and scopes; authorization servers issue `client_id` and `client_secret`. 【F:OAuth 2.1 for MCP Servers/README.md†L25-L44】【F:OAuth 2.1 for MCP Servers/README.md†L83-L84】
**Version management approach**: Not specified in the repository materials. 【F:OAuth 2.1 for MCP Servers/README.md†L1-L90】
**Limitations or pain points mentioned**: None explicitly called out. 【F:OAuth 2.1 for MCP Servers/README.md†L1-L90】

## MCP Authorization Server Metadata Configuration (Environment-Driven)
**How servers are discovered**: The well-known metadata endpoint advertises `authorization_servers`, `bearer_methods_supported`, resource documentation, and scopes. 【F:OAuth 2.1 for MCP Servers/server.py†L27-L48】
**Configuration/registration flow**: Authorization server URLs and resource metadata are configured via environment variables, enabling deployment-time configuration for discovery and authorization. 【F:OAuth 2.1 for MCP Servers/config.py†L1-L20】【F:OAuth 2.1 for MCP Servers/server.py†L35-L43】
**Version management approach**: Not specified in the repository materials. 【F:OAuth 2.1 for MCP Servers/server.py†L27-L48】
**Limitations or pain points mentioned**: None explicitly called out. 【F:OAuth 2.1 for MCP Servers/server.py†L27-L48】

# Cosmos SDK Smart Contract Registry Specification

## 1. Server Registration Fields (On-Chain)
Each server is a smart contract registry entry with the following fields:

- **server_id** (string): Unique identifier (e.g., DNS name + path or DID).
- **owner** (bech32 address): Entity that controls updates and stake.
- **endpoints** (array): List of reachable base URLs.
- **auth** (enum): `oauth2`, `api_key`, `none`, `custom`.
- **discovery_url** (string): Well-known discovery endpoint (e.g., `/.well-known/oauth-protected-resource/mcp`).
- **authorization_servers** (array of strings): OAuth authorization server URLs (if applicable).
- **scopes_supported** (array of strings): Advertised scopes for OAuth flows.
- **resource_docs_url** (string): URL for public resource documentation.
- **metadata_cid** (string): IPFS CID to off-chain metadata JSON.
- **version** (string): Semver-style server version (e.g., `1.2.3`).
- **protocol_version** (string): MCP protocol version (e.g., `mcp/1`).
- **status** (enum): `active`, `deprecated`, `suspended`.
- **stake** (coin): Staked amount to register or maintain listing.
- **created_at** (timestamp): Registration creation time.
- **updated_at** (timestamp): Last update time.

## 2. IPFS Metadata Structure
Metadata is stored as a JSON document pinned to IPFS, referenced by `metadata_cid`.

```json
{
  "name": "Finance News MCP",
  "description": "Sentiment analysis and finance news tools.",
  "homepage": "https://example.com",
  "contact": {
    "email": "support@example.com",
    "twitter": "@example",
    "discord": "https://discord.gg/example"
  },
  "endpoints": ["https://api.example.com"],
  "discovery": {
    "well_known": "/.well-known/oauth-protected-resource/mcp"
  },
  "auth": {
    "type": "oauth2",
    "authorization_servers": ["https://auth.example.com"],
    "scopes": ["mcp:tools:news:read"],
    "bearer_methods_supported": ["header"]
  },
  "capabilities": {
    "tools": ["news_sentiment"],
    "resources": ["/news"],
    "exec": ["workflow:sentiment"]
  },
  "protocol_version": "mcp/1",
  "server_version": "1.2.3",
  "license": "Apache-2.0",
  "tags": ["finance", "news", "sentiment"],
  "security": {
    "pkce_required": true,
    "short_lived_tokens": true,
    "redirect_uri_strict": true
  }
}
```

## 3. Reputation Scoring Algorithm
A deterministic score computed off-chain and periodically submitted on-chain:

- **Inputs**:
  - `uptime_ratio` (0–1) from monitoring.
  - `latency_p95_ms` from monitoring.
  - `auth_failure_rate` (0–1) based on invalid auth attempts.
  - `user_ratings` (1–5) aggregated from verified feedback.
  - `incident_count` (integer) in last 90 days.

- **Score formula**:
  ```
  score = 100 * uptime_ratio
        + 10 * (5 - (latency_p95_ms / 1000))
        + 5 * user_ratings
        - 50 * auth_failure_rate
        - 10 * incident_count
  ```
  - Clamp to [0, 100].
  - Weight adjustments can be governed by on-chain params.

- **Submission**:
  - Validators or authorized oracles submit `score` with evidence hashes.
  - Registry stores last N score snapshots with timestamps.

## 4. Stake Requirements
- **Registration stake**: Minimum stake `S_min` required to create a registry entry.
- **Maintenance stake**: Entries must maintain `S_maintain` to remain `active`.
- **Slashing**:
  - If `incident_count` exceeds a governance threshold or evidence of malicious behavior is proven, a percentage of stake is slashed.
- **Unbonding**:
  - Owners may unbond and withdraw stake after a cooling-off period `T_unbond` (e.g., 14 days), during which status is `deprecated`.

## 5. Discovery Query Patterns
Registry supports indexed queries for clients and integrators:

- **By capability**: `capabilities.tools CONTAINS "news_sentiment"`.
- **By auth type**: `auth == "oauth2"`.
- **By scope**: `scopes_supported CONTAINS "mcp:tools:news:read"`.
- **By protocol_version**: `protocol_version == "mcp/1"`.
- **By status**: `status == "active"`.
- **By minimum reputation**: `score >= 80`.
- **By region/tag**: `tags CONTAINS "finance"`.

## 6. Version Management
- **Server version** is a semver field stored on-chain and in IPFS metadata.
- **Protocol version** indicates MCP compatibility.
- **Deprecation flow**: setting `status = deprecated` plus `sunset_date` in metadata; clients should warn and migrate.

## 7. Limitations & Considerations
- **Discovery metadata** must be accessible over HTTPS.
- **Dynamic registration** should be constrained by allowlists or proof-of-ownership challenges.
- **Registry read access** must be public; write access requires stake and authorization.
