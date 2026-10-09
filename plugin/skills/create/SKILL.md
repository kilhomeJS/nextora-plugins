---
name: create
description: Build something in Nextora — a web form, report, dashboard, automation (workflow, any size), email template or campaign, table (custom entity), pipeline, AI bot, project, goal, list/segment — either directly with the build tools and then prove it works (dry run on a real record, real run with its step log, report numbers, a picture of the email or page, a test submission, a test chat), or through an AI workshop that drafts it for the user to accept. Use when the user asks to create, set up, build, automate, draft or generate any of these in Nextora or "the CRM" — e.g. "сделай форму заявки для сайта", "собери автоматизацию по новым лидам", "сделай отчёт по воронке", "create an onboarding pipeline".
argument-hint: "[what to build]"
---

# Build it in Nextora — and prove it works

Request: $ARGUMENTS

Two ways to build. Pick one before you start:

- **Build it yourself** (path A) — you write the object with the build tools,
  then check it with the matching check tool until it works. The default when
  the user wants *you* to build, edit or debug something, and the only way for
  large or intricate automations.
- **A workshop** (path B) — Nextora's AI builder asks its questions, drafts the
  object and the user accepts it. Use it when the user wants a draft to look
  at and accept, or says "через мастерскую".

A single record (one lead, deal, task, note, meeting) is neither: create it
with the CRM tools (`crm_create`, `calendar_event_create`, …).

## A. Build it yourself, then prove it

Not done until its check passed. Say what you checked — and what you could
not (a real email to a customer is never sent just to test).

**Automation (workflow), any size**
1. `workflow_node_catalog` — the step types, their config keys (required ones
   marked), the outputs a branching step's edges must name, the triggers per
   record type and their `trigger_config`, the variable syntax, the limits.
   Resolve every id you will write (pipelines, stages, users, chat channels,
   templates) with the CRM tools first — never invent one.
2. Write the graph: nodes `{ node_key, type, label, config }`, edges
   `{ source, target, source_handle? }` by node_key. Steps that do not branch
   leave `source_handle` empty; branching steps name an output on every edge.
   No single path longer than 50 steps — fan out instead.
3. `workflow_test` with `graph` (not saved yet): read `validation.errors` and
   fix every one; read each run's steps and `not_reached`. Run it on a real
   record (`record: { entity, id }`) to see the branch that record takes and
   what each step would send, resolved.
4. `workflow_create` (header; `entity_type` is derived from the trigger when it
   is unambiguous), then `workflow_save_graph` with the whole graph — or only
   `workflow_save_graph` for an existing workflow, passing the `revision` you
   read so a concurrent edit is not overwritten.
5. `workflow_set_active` when the user wants it live.
6. Real run, when the user wants proof: `workflow_run` for a manual workflow
   (yours, or you are an admin), or create a test record that matches the
   trigger. Then `workflow_runs({ workflow_id })` → `workflow_runs({
   execution_id })` shows every step with its output and error. A failed step
   names itself — fix the graph and run again.

**Report** — `report_sources` → `report_sources({ source })` for its fields →
`report_run` with the query until the numbers answer the question (a wrong
field is refused with the real ones) → `report_save` (up to 6 blocks; every
block is run first) → give the link.

**Email template** — `email_template_save` with blocks (only what matters: the
builder fills the rest; a `theme`) → `email_preview` and LOOK at the picture:
raw `{{…}}` means a variable that does not exist, broken images, empty blocks,
a missing unsubscribe → fix and save again.

**Form** — create it through the catalog (`catalog_search` "create form"), then
publish it (the update operation with `is_active: true`) → `page_preview` at
`/f/<id>`, 390 and 1280 wide → `form_test_submit` with answers by field label →
`form_submissions_list`, and `workflow_runs` for automations on its submissions.

**Bot** — create it and its flow through the catalog → `bot_test_chat` with
the questions its customers will ask; read the reply, its steps and tool calls.

**Custom table, dashboard, goal, pipeline, knowledge page, meeting type** —
their create/update operations are in `catalog_search`; read the object back
(and for a table, add a record and list it) to check.

## B. Build it with a Nextora workshop

Workshops are a durable server-side builder that asks clarifying questions,
produces a draft, and creates the real object only when the draft is applied.
Drive one from the user's request to an applied object, with the user deciding
at each point that matters.

### 0. Is this a workshop job?

- **Workshop:** form, report, dashboard, widget, workflow, campaign, template,
  table, pipeline, bot, project, goal, list. Use kind `plan` when the request
  spans several of them ("set up lead capture: a form, a pipeline and a
  follow-up automation") — a plan splits into child workshops.
- **Not a workshop:** a single record — one lead, contact, deal, task, note or
  meeting. Create it directly with the CRM tools (`crm_create`,
  `calendar_event_create`, …) and stop here.

### 1. Reach the workshop tools

The tools are `creation_discover`, `creation_list`, `creation_start`,
`creation_get`, `creation_continue`, `creation_cancel`, `creation_apply`.

If they are not in your tool list, they live in the Nextora catalog: call
`catalog_search` **with `category: "creation"`** (a plain-language search ranks
other tools first), then call each one with `{ "name": "<tool>", "arguments": { … } }`
through `catalog_execute` when it only reads (`creation_discover`, `creation_list`,
`creation_get`) or `catalog_execute_write` when it changes something
(`creation_start`, `creation_continue`, `creation_cancel`, `creation_apply`).

If the catalog returns `creation_get` but not `creation_start`, this
connection may read workshops but not build. Tell the user to remove and re-add
the Nextora connection in Claude (connections made before the `creation:write`
right existed do not carry it); if that does not help, their workspace admin has
to allow it. Stop.

### 2. Check the kind is allowed

Call `creation_discover` once per conversation (`locale: "ru"` when the user
writes Russian). For the chosen kind:

- `allowed: false` → the user's role may not build this kind. Name the missing
  permission (`requiredPermission`) and stop. Do not try another kind instead.
- Read `semantics` when present: a `project` workshop produces a blueprint, a
  `widget` is an analytics widget (not the website chat widget), a `plan` stores
  no object of its own.

### 3. Do not start a duplicate

`creation_list` with the kind and `statuses: ["briefing", "building", "ready"]`.
If an unfinished workshop has a similar brief, offer to continue it instead of
starting a new one.

### 4. Start

`creation_start` with:

- `request_id` — a fresh UUID v4 you generate **once**. If the call fails in
  transport, retry with the same id and the same payload; that returns the same
  workshop instead of a second one.
- `kind` — from step 0.
- `brief` — the user's request in their own words, plus concrete context from
  this conversation (fields, audience, sources, names, numbers). Do not add
  requirements the user did not ask for.
- `attachments` — only text the user gave you (up to 10, each ≤ 24 000 chars).

### 5. Follow the workshop

**In claude.ai and Claude Desktop, `creation_start` opens a live Nextora card**
that follows the workshop by itself: it shows the progress, asks the
workshop's question with answer buttons, shows the draft as the thing itself
(a report's numbers, a form's questions, an automation's steps, a campaign's
letters exactly as they will be mailed) and has **Принять / Accept**. So keep
your own summary of the draft to a line or two. There, do not poll — tell the person they can answer and accept in
the card or in the chat, and read the workshop with `creation_get` only when
you need its state (they ask, or before you act on it). When the card tells
you the person answered or accepted something, take it as done: do not answer
the same question again and never apply the same workshop twice.

Without a card (Claude Code and other text clients), follow it yourself:

Call `creation_get` with the workshop id and act on what it says. Use
`session.generation` as `expected_generation` in every follow-up call.

| `status` | What it means | What you do |
|---|---|---|
| `queued`, `running` | Building | Check again, no more often than every ten seconds or so. Between checks, tell the user it is building (the current `steps` are a good progress line). A build takes seconds to a few minutes. If it is still running after several checks, give the user `session.url` and offer to look again later. |
| `waiting_for_user` (or `session.question` is set) | The workshop has a question | Ask the user `question.text` in their language and offer `question.chips` as options. Send the answer with `creation_continue` — `action: "answer"`, `question_key: question.key`, `value`. Answer yourself only when the user already said it plainly in this conversation, and say that you did. |
| `draft` (`session.status: "ready"`) | Draft ready | Summarize what will be created — names, fields, steps, blocks — and anything listed in `session.omitted`. Ask the user to confirm or change it. A change goes back as `creation_continue` with `action: "refine"` and the change in words. |
| `failed` | Build failed | Show `session.errorMessage` in one line and offer a retry: `creation_continue` with `action: "retry"`. Finished steps survive a retry. |
| `completed` | Already applied | Give the result link (`receipt.url`). |
| `cancelled` | Abandoned | Say so; start again only if the user asks. |

A `conflict` error means the workshop moved on: read it again with
`creation_get` and use the new generation. Never start a replacement workshop to
get around a conflict.

### 6. Apply — only after the user says yes

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
