# Nextora for Claude

Work in your [Nextora](https://nextora.co) workspace from Claude: find and
update leads, deals and tasks, ask about your pipeline, and build forms,
reports, dashboards, automations, campaigns and bots with Nextora's AI
workshops.

## Install

**Claude Code**

```text
/plugin marketplace add kilhomeJS/nextora-plugins
/plugin install nextora@nextora
```

Then run `/mcp`, pick **nextora** and choose **Authenticate**. A page asks for
your workspace address — `acme` or `acme.nextora.co` is enough. Nextora asks
you to sign in and shows what Claude will be able to do; approve, and you are
connected.

**claude.ai and Claude Desktop** — Settings → Connectors → Add custom connector
→ `https://mcp.nextora.co/mcp`. The same sign-in and approval follow.

## What Claude can do

Exactly what you can do in Nextora — Claude works under your account, your role
and your workspace's access rules, and never sees anything you cannot.

| Skill | What it is for |
|---|---|
| `/nextora:create` | Build something with an AI workshop: a form, report, dashboard, automation, campaign, template, pipeline, bot, project, goal or list. Claude answers the workshop's questions with you, shows the draft, and creates it only after you confirm. |
| `/nextora:crm` | Work with CRM data safely: find before changing, preview bulk edits, confirm before deleting. |

You can also just ask — "how many deals are in negotiation?", "make a lead form
for the website" — and Claude picks the right one.

## Disconnect

Remove the connection in Claude: `/mcp` in Claude Code, or Settings →
Connectors in claude.ai and Claude Desktop.

## Support

Questions and problems: [open an issue](https://github.com/kilhomeJS/nextora-plugins/issues).
Privacy: [nextora.co/privacy](https://nextora.co/privacy) ·
Terms: [nextora.co/terms](https://nextora.co/terms)

## License

Proprietary — see [LICENSE](LICENSE). You may install and use this plugin with
your Nextora workspace; copying, modifying or redistributing it is not
permitted.
