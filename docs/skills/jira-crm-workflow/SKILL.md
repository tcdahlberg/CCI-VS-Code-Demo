---
name: jira-crm-workflow
description: "The CRM team's Jira process (ECRMSF project on the ECRM board) and the acli (Atlassian CLI) usage patterns for working with it — inactive-sprint intake workflow (Needs BA Review -> Needs CRM Review -> Needs Prioritization -> active sprint, plus On Hold and Not Recommended/Feasible), sprint IDs, and CLI command syntax discovered by trial (create/subtask/link/search/sprint lookup). Activate whenever the user mentions Jira, ECRMSF, acli, work items, subtasks, or the CRM review/prioritization process, so the process and CLI quirks don't need to be re-explained each session."
metadata:
  author: UST-Agentic-Coding-Skills
  version: "1.0"
allowed-tools: Bash Read Write
---

# Jira CRM Workflow (ECRMSF / acli)

## When to Use This Skill

Activate whenever the user asks to view, create, update, comment on, or move a Jira issue — especially anything in the `ECRMSF` project (Enterprise CRM - Salesforce, board id `97`, board name `ECRM`) — or mentions `acli`, work items, subtasks, sprints, or the CRM review/prioritization process.

---

## Use `acli`, Not the Jira MCP

Prefer the Atlassian CLI (`acli`) over Jira MCP tools for all Jira work (view, search, create, update, comment). It's already authenticated via OAuth and avoids an MCP-specific bug (see below).

```bash
acli jira auth status              # verify auth first — shows site/email/auth type, no need to ask the user
acli jira workitem view ECRMSF-1234
acli jira --help                   # subcommands: board, dashboard, field, filter, project, sprint, workitem
acli jira workitem [subcommand] --help   # ALWAYS check flags before constructing a command — don't guess
```

Only fall back to the Jira MCP if `acli` is unavailable (`which acli` fails) or a specific operation genuinely isn't supported by the CLI.

### `addCommentToJiraIssue` (MCP) Markdown Bug — only relevant if falling back to MCP

The Atlassian MCP's `addCommentToJiraIssue`, called with `contentFormat: "markdown"`, does not reliably convert `\n\n` into real ADF paragraph breaks — it can store the literal two-character sequence `\n` as visible text instead of a line break. `editJiraIssue`'s `description` field does NOT have this bug with the same format. Fix: for comments, skip markdown and pass `contentFormat: "adf"` with a hand-built ADF JSON doc (`paragraph`/`bulletList`/`text` nodes with `marks`). Re-calling with the same `commentId` updates in place rather than duplicating.

Separately, some Jira textarea custom fields reject ADF `text` nodes combining two marks on one run, or containing an em dash — generic `INVALID_INPUT` with no field detail. Bisect to a single-paragraph, single-mark payload with plain hyphens before assuming the field rejects rich text entirely.

---

## The CRM Team's Inactive-Sprint Intake Process

This team tracks issue triage/prioritization state using **inactive sprints** on the ECRM board (board id `97`), not the issue `status` field. Don't search `status = 'X'` for these names — they aren't statuses, they're sprints (`state: "future"` in the Agile API, confusingly, since they never start/end).

**Flow:** `Needs BA Review` → `Needs CRM Review` → `Needs Prioritization` → an active dated sprint (e.g. `ECRM SEP 2026`). A separate BA team reviews their own stories in `Needs BA Review`, then hands off into the CRM team's `Needs CRM Review` bucket. The CRM team reviews items there, then moves them to `Needs Prioritization`, where a stakeholder steering committee decides what enters the next active/dated sprint.

Two additional terminal-ish buckets: `On Hold` (waiting on external resource/availability) and `Not Recommended/Feasible` (rejected — performance impact or system limitation).

### Known sprint IDs (board 97 / ECRM) — confirmed via `acli jira board list-sprints --id 97 --state active,closed,future --json`

| Sprint name | id | Goal (paraphrased) |
|---|---|---|
| Needs BA Review | 125 | Not enough info to scope; needs BA review |
| Needs CRM Review | 124 | Ready for CRM team to review, create subtasks, estimate, assign |
| Needs Prioritization | 126 | Waiting for stakeholders to prioritize into the sprint backlog |
| On Hold | 144 | Pending external resource work/availability |
| Not Recommended/Feasible | 148 | Rejected — performance impact or system limitation |
| ECRM Student Intern Backlog | 145 | (future-state bucket, not part of the main flow) |

Dated active/closed sprints follow the pattern `ECRM <MON> <YYYY>` (e.g. `ECRM SEP 2026`, id `395` as of 2026-09). Re-run the `list-sprints` command above to get current IDs — don't hardcode the dated-sprint id, only the named-bucket ids above are stable.

If the user names a bucket not in this table, re-run `list-sprints` rather than guessing — new buckets may be added.

### ⚠️ Known gap: `acli` cannot move an issue into a sprint

Confirmed by exhausting the CLI surface (2026-09-22): `acli jira sprint update` only edits sprint metadata (name/goal/dates/state), not membership. `acli jira workitem edit` has no `--sprint` flag, and `--from-json`/`--generate-json` for `edit` only expose `assignee`, `description`, `issues`, `labelsToAdd/Remove`, `summary`, `type` — no custom-field/sprint support. `workitem create --generate-json` exposes `additionalAttributes` for custom fields, but that only helps at creation time, not for moving an existing issue, and the exact Sprint custom-field id for this instance was not resolved (the standard Jira Agile field, often `customfield_10020`, wasn't present in this instance's field dump on a sample issue with no sprint set — would need `?expand=editmeta` or a describe call to confirm, which the raw `acli --json` output doesn't surface by default).

The underlying Jira Agile REST endpoint is `POST /rest/agile/1.0/sprint/{sprintId}/issue` with `{"issues": ["KEY"]}`, but `acli` has no passthrough for arbitrary REST calls and does not expose its underlying OAuth token for reuse — going around it would mean extracting a credential, which conflicts with this workspace's security rules (never handle/expose credentials, see agent-onboarding).

**Current workaround: ask the user to drag the issue into the target sprint bucket in the Jira UI** after creating/updating it via `acli`. Don't attempt an OAuth-token workaround without the user's explicit sign-off. If `acli` ships sprint-membership support in a future version, check `acli jira workitem edit --help` and `acli jira sprint --help` again before assuming this gap still exists.

---

## `acli` Command Patterns Learned By Trial

These flag/argument quirks were not obvious from `--help` alone and cost extra round-trips to discover — check here before re-guessing.

- **`workitem create --from-file` vs `--summary`/`--description-file` are mutually exclusive as a group.** Passing `--from-file` together with `--summary` errors: `if any flags in the group [from-file summary] are set none of the others can be`. Use `--summary` + `--description-file` (or `--description`) together instead — that combination works fine and is more predictable for scripted use.
- **Subtask creation needs `--parent` as the issue KEY, not the numeric issue id.** `--parent "61119"` (the numeric id returned in `create`'s JSON `id` field) fails with `Please select valid parent issue.` — use `--parent "ECRMSF-5907"` (the `key` field) instead.
- **`workitem edit --description-file` works even though it's not listed in `edit --help`'s flag list.** The help text only shows `--description` and `--description-file` is undocumented there but functions correctly — useful for updating a long description after creation without re-typing `--description` inline.
- **`workitem search --jql` errors loudly (and usefully) on an invalid field value** — e.g. searching `status = 'Needs CRM Review'` returns `the value 'needs crm review' does not exist for the field 'status'`, which was the actual signal that this was a sprint, not a status. Treat that specific error shape as a hint to check `board list-sprints` next.
- **`search --json` returns a bare JSON array of issues**, not an object with an `issues` key — despite the human-readable interactive output implying a paginated result object. Parse accordingly (`for i in data:`, not `data['issues']`).
- **`workitem view --json` returns the full raw Jira REST issue payload** (`changelog`, `editmeta`, `fields`, `names`, `schema`, `transitions`, etc.) but most of those top-level keys are `null` unless the request includes the right `expand` params, which `acli` doesn't appear to set by default. Don't rely on `editmeta`/`names`/`schema` from this output to discover custom-field IDs — they'll likely be empty.
- **Link types available on this instance** (via `acli jira workitem link type`): `Blocks`, `Cloners`, `Discovery - Connected`, `Duplicate`, `Gantt End to End/Start`, `Polaris datapoint/merge/work item link`, `Problem/Incident`, `Relates`. Use `Relates` for general "these are connected" links between a new remediation issue and the issues it follows up on.
- **Board/project discovery path:** `acli jira board search --project ECRMSF --json` → gives board `id`. Then `acli jira board list-sprints --id <boardId> --state active,closed,future --json` → full sprint list including the named inactive buckets. Without `--state active,closed,future`, `list-sprints` can return an empty result — always pass all three states explicitly when trying to find a bucket that isn't the current active sprint.

---

## Example: Creating a Remediation Story With Subtasks (worked pattern)

```bash
# 1. Create the parent story
acli jira workitem create --project "ECRMSF" --type "Task" \
  --summary "..." --description-file /tmp/desc.txt --json

# 2. Link it to related issues (use Relates unless a stronger relationship applies)
acli jira workitem link create --out ECRMSF-NEW --in ECRMSF-OLD --type Relates --yes

# 3. Create subtasks under it — --parent MUST be the issue KEY
acli jira workitem create --project "ECRMSF" --type "Subtask" --parent "ECRMSF-NEW" \
  --summary "..." --description-file /tmp/sub_desc.txt --json

# 4. Verify subtask parenting via search (the parent's own `subtasks` field in `view`
#    can lag briefly right after creation — don't trust a `subtasks: []` immediately after)
acli jira workitem search --jql "parent = ECRMSF-NEW" --fields "key,summary" --json
```

Clean up any `/tmp/*.txt` description scratch files used for `--description-file` after the calls succeed — per workspace convention, use `ai-logs/` instead of `/tmp/` if the description content itself is worth preserving for audit purposes; `/tmp/` is fine for pure throwaway staging text with no sensitive content.
