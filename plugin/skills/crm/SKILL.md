---
name: crm
description: Find, review and change Nextora CRM data safely — leads, contacts, companies, deals, tasks, pipelines, notes, calendar. Use for questions about the pipeline, forecasts or workload ("сколько сделок в работе", "что горит на этой неделе") and whenever the user asks to update, move, assign, bulk-edit, merge or clean up records in Nextora.
---

# Working with Nextora CRM data

Every Nextora tool runs as the connected person, under their role and
row-level security: you see and change exactly what they could in the app.
These habits keep changes correct and visible to them.

## Read

- **Resolve before acting.** Records are keyed by UUID. Find the record first
  (`crm_search`, or `crm_list` with `query`), then act on its id. Never invent
  an id, a status or a stage.
- **Introspect, don't guess.** Before filtering by status or touching a custom
  field, call `describe_entity` — it returns the statuses actually in use, the
  pipeline stages and the custom-field keys. A wrong status filters to an empty
  list, not an error.
- **Count, don't list,** for "how many" questions: `crm_count`. For "how is the
  pipeline", start from `crm_analytics`, `deal_forecast` or `pipeline_board`.
- **Deals:** `status` is the outcome (`in_progress`, `won`, `lost`, `on_hold`);
  `stage_id` is the column on the board, resolved with `deal_stages_list`. Move
  a deal with `deal_move_stage`.
- **More tools than you see.** The connection starts with a small toolset.
  Anything else — forms, projects, knowledge base, ERP, settings — is found with
  `catalog_search` (pass `category` when you know the area: `crm`, `pipelines`,
  `projects`, `comms`, `marketing`, `knowledge`, `analytics`, `files`, `erp`,
  `creation`) and run through `catalog_execute` (tools that only read) or
  `catalog_execute_write` (tools that change data — Claude asks first).

## Show it as a card

In claude.ai and the Claude apps Nextora draws live cards. When the person
wants to SEE their work rather than read a summary, call the card tool and keep
your text short — the card carries the detail:

- "what's on today", "утренняя сводка", "что горит" → `nextora_my_day`
- a specific deal, lead, contact or company → resolve its id, then
  `nextora_show_record`
- "how's the pipeline", "покажи воронку" → `nextora_show_pipeline`
- "how are we doing", "итоги месяца" → `nextora_show_metrics` (the person
  switches week / month / quarter / year in the card)
- "find …", "найди …" → `nextora_search`
- adding a lead, contact, company, deal or task they'd want to check →
  `nextora_new_record`, pre-filled from the conversation
- "кто ждёт ответа", "что во входящих" → `nextora_inbox`; one conversation →
  `nextora_show_conversation`. Replies are SENT in Nextora from the person's
  own channel: draft the text for them, never say you sent it.
- a table the person attached (CSV/XLSX) → `nextora_import` with the table as
  CSV text, header row first. It only checks; the person picks fields and what
  to do with duplicates and starts the import in the card. For files over
  ~1.5 MB point them to the importer in Nextora.

The person can act inside a card — tick tasks off, move a deal to another
stage, drag deals on the board. Each change made there reaches you as context
("the user moved the deal … to …"): take it as done, do not repeat it, and do
not call the card tool again just to refresh. Use `crm_get` / `crm_query` for
your own lookups mid-task; the card tools are for showing.

## Change

- **Several records at once → show it first.** Propose the change with
  `nextora_review_changes` (each record: old → new). The person unticks what to
  leave, applies it in the card, and can put it back. That tool writes nothing;
  wait for the card to tell you what was applied, and never apply it again.

- **Preview first** whenever a change touches more than one record or moves a
  deal: run the write with `dry_run: true`, show the user what will change and
  which automations will fire, and wait for a yes.
- **Destructive tools** (`crm_delete`, `bulk_delete`, `entity_record_delete`)
  answer without `confirm: true` with a preview only. Pass `confirm: true` only
  after the user explicitly agreed to that exact deletion. Deleted records go
  to the trash and can be restored with `crm_restore` — unless `hard: true` is
  passed, which deletes permanently. Never pass `hard: true` unless the user
  asked for permanent deletion in those words.
- **Assign** with `owner: "$me"` for the user themselves, or an id from
  `team_members_list`.
- **Repeat safely.** Tools that ask for an `operation_id` use it to make a retry
  return the same result — generate one per intended change and reuse it on a
  retry.
- **Say what really happened.** Report the result the tool returned — saved,
  partially saved, queued, waiting for the user — not what you intended. A
  queued job is not a finished one; check its status before claiming the result.

## Build instead of edit

Creating a form, report, dashboard, automation, campaign, pipeline, bot or
similar structure is a workshop job — use the `create` skill
(`/nextora:create`).
