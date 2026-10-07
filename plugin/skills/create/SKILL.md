---
name: create
description: Build something in Nextora with its AI workshops — a web form, report, dashboard, analytics widget, automation (workflow), email campaign, email template, table (custom entity), pipeline, AI bot, project, goal, list/segment, or a multi-part plan. Use when the user asks to create, set up, build, draft or generate any of these in Nextora or "the CRM" — e.g. "сделай форму заявки для сайта", "собери дашборд по продажам", "настрой рассылку новым лидам", "create an onboarding pipeline".
argument-hint: "[what to build]"
---

# Build it with a Nextora workshop

Nextora builds structures — forms, reports, automations, campaigns — through
**workshops**: a durable server-side builder that asks clarifying questions,
produces a draft, and creates the real object only when the draft is applied.
Your job is to drive one workshop from the user's request to an applied object,
with the user deciding at each point that matters.

Request: $ARGUMENTS

## 0. Is this a workshop job?

- **Workshop:** form, report, dashboard, widget, workflow, campaign, template,
  table, pipeline, bot, project, goal, list. Use kind `plan` when the request
  spans several of them ("set up lead capture: a form, a pipeline and a
  follow-up automation") — a plan splits into child workshops.
- **Not a workshop:** a single record — one lead, contact, deal, task, note or
  meeting. Create it directly with the CRM tools (`crm_create`,
  `calendar_event_create`, …) and stop here.

## 1. Reach the workshop tools

The tools are `creation_discover`, `creation_list`, `creation_start`,
`creation_get`, `creation_continue`, `creation_cancel`, `creation_apply`.

If they are not in your tool list, they live in the Nextora catalog: call
`catalog_search` **with `category: "creation"`** (a plain-language search ranks
other tools first), then call each one through `catalog_execute` with
`{ "name": "<tool>", "arguments": { … } }`.

If the catalog returns `creation_get` but not `creation_start`, this
connection may read workshops but not build. Tell the user to remove and re-add
the Nextora connection in Claude (connections made before the `creation:write`
right existed do not carry it); if that does not help, their workspace admin has
to allow it. Stop.

## 2. Check the kind is allowed

Call `creation_discover` once per conversation (`locale: "ru"` when the user
writes Russian). For the chosen kind:

- `allowed: false` → the user's role may not build this kind. Name the missing
  permission (`requiredPermission`) and stop. Do not try another kind instead.
- Read `semantics` when present: a `project` workshop produces a blueprint, a
  `widget` is an analytics widget (not the website chat widget), a `plan` stores
  no object of its own.

## 3. Do not start a duplicate

`creation_list` with the kind and `statuses: ["briefing", "building", "ready"]`.
If an unfinished workshop has a similar brief, offer to continue it instead of
starting a new one.

## 4. Start

`creation_start` with:

- `request_id` — a fresh UUID v4 you generate **once**. If the call fails in
  transport, retry with the same id and the same payload; that returns the same
  workshop instead of a second one.
- `kind` — from step 0.
- `brief` — the user's request in their own words, plus concrete context from
  this conversation (fields, audience, sources, names, numbers). Do not add
  requirements the user did not ask for.
- `attachments` — only text the user gave you (up to 10, each ≤ 24 000 chars).

## 5. Follow the workshop

Call `creation_get` with the workshop id and act on what it says. Use
`session.generation` as `expected_generation` in every follow-up call.

| `status` | What it means | What you do |
|---|---|---|
| `queued`, `running` | Building | Check again. Between checks, tell the user it is building (the current `steps` are a good progress line). A build takes seconds to a few minutes. If it is still running after several checks, give the user `session.url` and offer to look again later. |
| `waiting_for_user` (or `session.question` is set) | The workshop has a question | Ask the user `question.text` in their language and offer `question.chips` as options. Send the answer with `creation_continue` — `action: "answer"`, `question_key: question.key`, `value`. Answer yourself only when the user already said it plainly in this conversation, and say that you did. |
| `draft` (`session.status: "ready"`) | Draft ready | Summarize what will be created — names, fields, steps, blocks — and anything listed in `session.omitted`. Ask the user to confirm or change it. A change goes back as `creation_continue` with `action: "refine"` and the change in words. |
| `failed` | Build failed | Show `session.errorMessage` in one line and offer a retry: `creation_continue` with `action: "retry"`. Finished steps survive a retry. |
| `completed` | Already applied | Give the result link (`receipt.url`). |
| `cancelled` | Abandoned | Say so; start again only if the user asks. |

A `conflict` error means the workshop moved on: read it again with
`creation_get` and use the new generation. Never start a replacement workshop to
get around a conflict.

## 6. Apply — only after the user says yes

`creation_apply` with the id and `expected_generation`. Then report the real
object: its type (`receipt.entityType`) and its link (`receipt.url`).

- `plan`: apply returns child workshops. Take each child through steps 5–6 on
  its own (read, confirm, apply).
- `project`: the result is a blueprint. If the user wants the project itself,
  find the project creation tool with `catalog_search` (`category: "projects"`).

## Rules

- **Nothing is created until `creation_apply` returns `complete: true` with a
  receipt.** `queued`, `running` and `draft` are not objects — never say "done"
  or "created" before that.
- Give links from the `url` fields (absolute, into the user's workspace). If a
  result carries only `href`, say it is a path inside their workspace.
- To change an object after it is applied, use that object's own tools (found
  through `catalog_search`), or start a new workshop if the change is a rebuild.
- Keep the user in their language. Workshop texts come in `{ ru, en }` — pick
  theirs.
