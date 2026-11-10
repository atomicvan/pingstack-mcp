# Publishing a new version

These steps are for Pingstack maintainers.

## Keep the versions in step

`server.json`, `gemini-extension.json`, `.claude-plugin/plugin.json`,
`.claude-plugin/marketplace.json`, `.cursor-plugin/plugin.json` and
`antigravity/plugin.json` carry the same version as the `version` the server
reports in the app (`Automation::Tools::VERSION` in
`app/services/automation/tools.rb`). Bump all seven together when the tools, the description or the setup changes.
Registry versions can't be changed or reused once published.

## Official MCP Registry

The registry name `io.pingstack/pingstack` belongs to the `pingstack.io`
domain, so publishing needs proof that we control it. Install the publisher
with `brew install mcp-publisher`, then pick one of these.

**HTTP (no DNS access needed).** Generate a key once, with OpenSSL 3
(`brew install openssl@3`; macOS's built-in LibreSSL can't make Ed25519 keys):

```sh
OPENSSL=/opt/homebrew/opt/openssl@3/bin/openssl
$OPENSSL genpkey -algorithm Ed25519 -out key.pem
echo "v=MCPv1; k=ed25519; p=$($OPENSSL pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"
```

Serve that line as plain text at `https://pingstack.io/.well-known/mcp-registry-auth`
(in the sales site, `public/.well-known/mcp-registry-auth`). The registry
doesn't follow redirects, so it has to load directly on the apex domain. Then:

```sh
mcp-publisher login http --domain pingstack.io \
  --private-key "$($OPENSSL pkey -in key.pem -noout -text | grep -A3 'priv:' | tail -n +2 | tr -d ' :\n')"
mcp-publisher validate
mcp-publisher publish
```

**DNS.** Add the same `v=MCPv1; k=ed25519; p=...` line as a TXT record on
`pingstack.io`, wait until `dig TXT pingstack.io +short` shows it, and use
`mcp-publisher login dns --domain pingstack.io --private-key ...` instead.

Keep `key.pem` in the password manager, never in a repository, and leave the
file or TXT record in place for the next version.

Check the listing:

```sh
curl "https://registry.modelcontextprotocol.io/v0.1/servers?search=io.pingstack/pingstack"
```

## Claude Code plugin

`.claude-plugin/marketplace.json` makes this repository a plugin marketplace,
and `.mcp.json` is the plugin's server. Check both with
`claude plugin validate .` before tagging.

## Skills

The `skills` folder holds one folder per skill, each with a `SKILL.md`. Claude
Code and Cursor both find them there without a manifest entry. Keep each
skill's `name` equal to its folder name, in lowercase with hyphens, and keep
the frontmatter to `name` and `description` so both clients accept it. Skills
name the server's tools, so update them when a tool is renamed or removed.
`claude plugin validate .` checks them along with the plugin.

## Gemini CLI

`gemini extensions install https://github.com/atomicvan/pingstack-mcp`
installs straight from the default branch. The repository needs the
`gemini-cli-extension` GitHub topic to appear in the extension gallery.

## Cursor plugin

`.cursor-plugin/plugin.json` makes this repository a Cursor plugin, and
`mcp.json` (not Claude's `.mcp.json`) is its server. The logo is
`assets/icon-512.png`. Cursor Marketplace submissions go through
https://cursor.com/marketplace/publish; the checklist is in
https://cursor.com/docs/reference/plugins.

## Antigravity plugin

The plugin lives in `antigravity/` (`plugin.json`, `mcp_config.json`, its own
`README.md` and `assets/logo.png`), apart from the repository root, because
Antigravity's strict check warns about the Claude and Gemini files there. The
logo is the color mark with transparent padding, so it works on light and dark
themes. Before tagging, check it with
`agy plugin validate --strict antigravity` (no errors and no warnings) and
`agy plugin install ./antigravity`. Marketplace listings go through the
partner intake form linked from https://antigravity.google/docs/build-plugins.
