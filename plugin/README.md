# Nextora for Claude

Work in your [Nextora](https://nextora.co) CRM from Claude. This plugin connects
Claude to your Nextora workspace and teaches it how to work there well.

## What's inside

- **The Nextora connector** (`.mcp.json`): a remote MCP server at
  `https://mcp.nextora.co/mcp`. It is the only thing the plugin connects to.
  Nothing runs on your machine — there are no scripts, hooks or local servers.
- **Two skills**:
  - `crm` — find, read and change leads, contacts, companies, deals and tasks
    safely: resolve before acting, preview multi-record changes, show your day,
    a record, the pipeline, the numbers, the inbox or a table import as live
    Nextora cards.
  - `create` — build forms, reports, dashboards, automations, campaigns,
    pipelines, bots, goals, lists and more with Nextora's AI workshops; the
    draft is shown as the thing itself and nothing is created until you accept.

## Connecting

On first use Claude opens Nextora's sign-in: you enter your workspace address
(for example `acme`), sign in to Nextora as usual and approve the access. Your
password is typed only in Nextora. Claude then works as you — with your role's
permissions and your workspace's access rules.

## What is sent where

- Claude sends tool calls (the request you made, such as "show my pipeline") to
  `mcp.nextora.co`, which relays them to your own workspace and returns the
  result. The relay stores no CRM data; its tokens are encrypted and expire.
- Your data stays in your Nextora workspace. Nextora does not read your Claude
  conversations.
- Privacy policy: https://nextora.co/privacy · Terms: https://nextora.co/terms

## Documentation and support

- Full documentation: https://mcp.nextora.co/docs
- Support: https://github.com/kilhomeJS/nextora-plugins/issues

## License

Proprietary — see `LICENSE`. The plugin may be installed and used with a
Nextora workspace; it may not be copied, modified or redistributed.
