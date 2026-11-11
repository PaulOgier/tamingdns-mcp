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

Every tool is read-only. The checks look up public DNS records and fetch the files a domain publishes for MTA-STS, BIMI and mail-client autoconfiguration: the policy file, the logo, the certificate and the autoconfig XML. The header tool reads the headers you paste. None of them change anything.

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
| `analyze_email_headers` | Raw email headers, for spoofing and phishing signs |

`check_dkim` needs `domain` and `selector`. `check_dns_propagation` needs `domain` and takes an optional `type` (A, AAAA, MX, TXT, CNAME, NS, SOA, SRV, CAA, PTR or TLSA; default A). `analyze_email_headers` needs `headers`. The rest need only `domain`.

`check_m365` and `check_google_workspace` are built from the same results the single checks return, so they never disagree with them. Not sure which provider a domain uses? `check_mx` names it.

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
- "Is example.com set up correctly for Microsoft 365?"
- "I just changed the MX record for example.com. Has it spread yet?"
- "Is the DKIM key at selector1 for example.com long enough?"
- "Here are the headers of an email I got. Is it spoofed?"

When a check cannot run, for example because the DNS resolvers did not answer, the tool returns an `error` with `retriable: true` rather than reporting the record as missing.
