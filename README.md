# TamingDNS MCP server

[![mcptry](https://mcptry.com/badge/com.tamingdns/mcp)](https://mcptry.com/servers/com.tamingdns/mcp)

A remote MCP server that checks a domain's email authentication and analyses raw email headers. It is the same engine as the tools on [tamingdns.com](https://tamingdns.com).

This repo holds no source code. The server runs at the endpoint below.

- Endpoint: `https://tamingdns.com/mcp`
- Transport: Streamable HTTP, JSON-RPC 2.0 over POST, stateless
- Auth: none. Rate limits apply per IP.
- Registry name: `com.tamingdns/mcp` in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=com.tamingdns)
- Full docs: [tamingdns.com/mcp](https://tamingdns.com/mcp)

## Tools

Every tool is read-only. The checks look up public DNS records and fetch the files a domain publishes for MTA-STS and BIMI: the policy file, the logo and the certificate. The header tool reads the headers you paste. None of them change anything.

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
| `analyze_email_headers` | Raw email headers, for spoofing and phishing signs |

`check_dkim` needs `domain` and `selector`. `analyze_email_headers` needs `headers`. The rest need only `domain`.

There is also one prompt, `audit_email_auth`, which runs the four email-auth checks against a domain and combines the results.

## Connect

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
- "Is the DKIM key at selector1 for example.com long enough?"
- "Here are the headers of an email I got. Is it spoofed?"

When a check cannot run, for example because the DNS resolvers did not answer, the tool returns an `error` with `retriable: true` rather than reporting the record as missing.
