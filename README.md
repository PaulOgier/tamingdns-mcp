# TamingDNS MCP server

[![mcptry](https://mcptry.com/badge/com.tamingdns/mcp)](https://mcptry.com/servers/com.tamingdns/mcp) [![Glama](https://glama.ai/mcp/connectors/com.tamingdns/mcp/badges/score.svg)](https://glama.ai/mcp/connectors/com.tamingdns/mcp)

A remote MCP server that checks a domain's email authentication and analyses raw email headers. It is the same engine as the tools on [tamingdns.com](https://tamingdns.com).

This repo holds no source code. The server runs at the endpoint below.

- Endpoint: `https://tamingdns.com/mcp`
- Transport: Streamable HTTP, JSON-RPC 2.0 over POST, stateless
- Auth: none. Rate limits apply per IP.
- Registry name: `com.tamingdns/mcp` in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=com.tamingdns)
- Full docs: [tamingdns.com/mcp](https://tamingdns.com/mcp)

## Tools

Every tool is read-only. The checks look up public DNS records and fetch what a domain publishes for MTA-STS, BIMI, mail-client autoconfiguration and its website: the policy file, the logo, the certificates, the autoconfig XML and the site's HTTPS response. The header, bounce and DMARC report tools read what you paste. None of them change anything.

| Tool | What it checks |
|---|---|
| `check_spf` | SPF record, grade, the `all` qualifier through `redirect=` chains |
| `check_dkim` | DKIM key at `<selector>._domainkey.<domain>`: revocation, key length |
| `check_dmarc` | `_dmarc` record and policy |
| `check_mx` | MX hosts, their addresses, PTR and forward-confirmed reverse DNS |
| `check_mta_sts` | MTA-STS TXT record and policy (RFC 8461) |
| `check_tls_rpt` | TLS-RPT record at `_smtp._tls.<domain>` (RFC 8460) |
| `check_bimi` | BIMI record, SVG logo, VMC certificate chain |
| `check_dane` | TLSA records for SMTP DANE at `_25._tcp.<mx-host>` (RFC 7672) |
| `check_dnssec` | DNSSEC chain of trust from the root to the domain |
| `check_m365` | Everything Microsoft 365 expects in DNS: SPF include, DKIM selector1/selector2, DMARC, MX, MTA-STS, Autodiscover, TLS-RPT, BIMI, Teams SIP SRV |
| `check_google_workspace` | Everything Google Workspace expects in DNS: MX, SPF include, the `google` DKIM key, DMARC, MTA-STS, TLS-RPT, BIMI |
| `check_dns_propagation` | One record type at one name, asked of 14 public resolvers, to see whether a change has spread |
| `check_txt` | Every TXT record at a domain, labelled by vendor, with duplicate SPF records, misplaced DKIM keys and stale tokens flagged |
| `check_website` | One hostname's TLS certificate, security headers and HTTPS redirects |
| `check_mcp_keys` | The `v=MCPv1` TXT records the MCP Registry DNS login checks a signature against |
| `analyze_email_headers` | Raw email headers, for spoofing and phishing signs |
| `decode_bounce` | A pasted bounce or SMTP error: the reply and enhanced status codes, what they mean and how to fix it |
| `analyze_dmarc_report` | One DMARC aggregate (RUA) report, pasted as XML text |

`check_dkim` needs `domain` and `selector`. `check_dns_propagation` needs `domain` and takes an optional `type` (A, AAAA, MX, TXT, CNAME, NS, SOA, SRV, CAA, PTR or TLSA; default A). `analyze_email_headers` needs `headers`, `decode_bounce` needs `bounce` and `analyze_dmarc_report` needs `report`. The rest need only `domain`.

`check_m365` and `check_google_workspace` are built from the same results the single checks return, so they never disagree with them. Not sure which provider a domain uses? `check_mx` names it.

There is also one prompt, `audit_email_auth`, which runs the four email-auth checks against a domain and combines the results.

## Connect

[Install in VS Code](https://insiders.vscode.dev/redirect/mcp/install?name=tamingdns&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Ftamingdns.com%2Fmcp%22%7D). One-click links for Cursor, LM Studio and Goose are on [tamingdns.com/mcp](https://tamingdns.com/mcp).

Claude Code:

```
claude mcp add --transport http tamingdns https://tamingdns.com/mcp
```

Cursor, in `.cursor/mcp.json` or `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "tamingdns": { "url": "https://tamingdns.com/mcp" }
  }
}
```

VS Code: run MCP: Add Server, or add this to `.mcp.json` at the workspace root, or to `~/.copilot/mcp-config.json` for every workspace:

```json
{
  "mcpServers": {
    "tamingdns": {
      "type": "http",
      "url": "https://tamingdns.com/mcp",
      "tools": ["*"]
    }
  }
}
```

Windsurf: in Cascade's MCP menu, choose Open MCP config file and add:

```json
{
  "mcpServers": {
    "tamingdns": { "serverUrl": "https://tamingdns.com/mcp" }
  }
}
```

Zed: use Settings > AI > MCP Servers > Add Server > Add Remote Server, or add this to your settings file:

```json
{
  "context_servers": {
    "tamingdns": { "url": "https://tamingdns.com/mcp" }
  }
}
```

LM Studio: in the Program tab, choose Install > Edit `mcp.json` and add the same block as Cursor. LM Studio reads Cursor's format.

Goose, in `~/.config/goose/config.yaml`:

```yaml
extensions:
  tamingdns:
    name: TamingDNS
    type: streamable_http
    uri: https://tamingdns.com/mcp
    enabled: true
    timeout: 300
```

Claude Desktop or claude.ai: open the [TamingDNS listing in the Claude directory](https://claude.ai/directory/tamingdns) and click Connect to Claude.

A client that only speaks stdio:

```
npx mcp-remote https://tamingdns.com/mcp
```

No MCP client? List the tools with curl:

```
curl -sS -X POST https://tamingdns.com/mcp \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Use

Ask your agent things like:

- "Check the DMARC record for example.com"
- "Is example.com set up correctly for Microsoft 365?"
- "I just changed the MX record for example.com. Has it spread yet?"
- "Is the DKIM key at selector1 for example.com long enough?"
- "Here are the headers of an email I got. Is it spoofed?"

When a check cannot run, for example because the DNS resolvers did not answer, the tool returns an `error` with `retriable: true` rather than reporting the record as missing.
