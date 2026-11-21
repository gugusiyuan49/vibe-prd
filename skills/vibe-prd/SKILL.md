---
name: vibe-prd
description: Two-way PRD engine for vibe coding. Forward mode turns a one-line idea into a SPEC.md contract (role matrix, Mermaid state machine, 5-category edge-case matrix, data schema and API contract, 5-step checklist) before any code is written. Reverse mode turns an existing HTML or React prototype into a single-file left-UI right-annotation (左图右注) interactive PRD with numbered badges, field dictionaries, Mermaid diagrams, edge cases and tracking events. Use when the user invokes vibe-prd (/vibe-prd, $vibe-prd, @vibe-prd), types /spec or /prd, asks to write a spec or PRD, plan a feature before coding, or document or reverse-engineer an existing page, or says 写SPEC, 先别写代码, 需求文档, 补PRD, 左图右注, 交互说明, 字段字典. Not for ordinary coding requests.
license: MIT
metadata:
  version: "1.0.0"
  repository: "https://github.com/gugusiyuan49/vibe-prd"
---

# vibe-prd · 左图右注

Two engines, one rule: **write the contract before the code, and the docs from the code.**

| Mode | You get | Triggered by |
| --- | --- | --- |
| A · Forward | `SPEC.md`: roles, state machine, edge cases, schema, checklist | `spec <idea>`, `/spec`, "plan this before coding", "写 SPEC" |
| B · Reverse | `PRD_Viewer.html`: the live prototype on the left, numbered annotations on the right | `prd <path>`, `/prd`, "document this page", "左图右注", "补 PRD" |

Files this skill uses (paths are relative to this SKILL.md):

- `templates/spec-template.md`: skeleton for Mode A
- `templates/left-right-prd.html`: self-contained viewer for Mode B. Copy it; never rewrite its code.

## 0. Pick the mode

1. Read the invocation. Claude Code hands over the text after `/vibe-prd` as `ARGUMENTS: ...`; other agents put it in the message. The first word decides: `spec` or `/spec` means Mode A, `prd` or `/prd` means Mode B.
2. No mode word: an idea or feature still to be built means A; a file, URL or "this page" that already exists means B.
3. Still unclear: ask one question ("Spec a new feature, or document an existing page?") and stop.
4. Write every document in the user's language (Chinese in, Chinese out). Code identifiers, Mermaid IDs, file names and event names stay ASCII.

## Mode A · Forward: `spec <idea>` → `SPEC.md`

**Hard rule: no business code in this mode.** Create or edit no file except `SPEC.md` until the user approves it (A6).

**A1. Read the project first.** Skim the README, the dependency manifest, existing schema, types, routes and any earlier SPEC. Reuse the real stack, naming and tables. In an empty folder, choose a mainstream stack and record it as an assumption.

**A2. Close gaps without stalling.** If a missing fact would change the data model or the permissions (who uses it, what the core entity is, whether money moves), ask at most 3 numbered questions in one message and wait. Otherwise choose a sensible default and log it in the Assumptions table, where the user can correct it during review.

**A3. Fill `templates/spec-template.md`.** All five modules are mandatory, in this order:

1. **Product overview & user roles.** Goal, a measurable success metric, non-goals, key scenarios, and a role × capability matrix (✅ allowed, ⛔ denied, 🔶 needs approval).
2. **Core state machine.** A Mermaid `stateDiagram-v2` of the main entity (order, bill, session, task...) with the happy path and every cancel, rollback, timeout and failure path, plus a transition table: from, to, trigger, guard, side effects.
3. **5-category edge-case matrix.** Columns exactly `场景分类 | 触发条件 | 前端 UI/文案表现 | 后端/数据兜底策略` (English docs: `Category | Trigger | Frontend UI & copy | Backend / data fallback`). All five categories, at least 2 rows each:
   - Empty / zero state (first use, no data, filter returns nothing)
   - Boundary / overflow (long text, negatives, extremes, bad formats)
   - Network / API latency and timeout (skeletons, 500s, retries, idempotency, duplicate submit)
   - Concurrency and state conflict (two people edit one record, stale state)
   - Permission and auth guard (no access, expired session, row-level scope)
4. **Data schema & API contract.** SQL DDL (or the project's ORM schema), TypeScript interfaces, an endpoint table, request and response JSON for the main write call, and error codes. Every state in the diagram exists as an enum value in the schema.
5. **Atomic vibe-coding checklist.** At most 5 steps. Each step can be built and verified on its own and ends with an acceptance check someone can run or click ("`npm test recon` passes", "submitting twice creates one order").

**A4. Self-check, then fix (don't report).**

- Every state is reachable and every non-final state has an exit.
- Every edge-case row names concrete UI copy and a concrete backend behaviour. "Handle the error gracefully" is not an answer.
- Every write endpoint states its idempotency (key or natural key) and its concurrency control (version column or lock).
- Every Mermaid block follows the rules in "Mermaid rules" below.

**A5. Save.** Write `SPEC.md` to the project root. If one already exists, write `SPEC.<feature-slug>.md` and say so. Never overwrite silently.

**A6. Hand over, then stop.** Reply in at most 8 lines: the path, the states of the state machine, the 3 riskiest edge cases, and the open questions. End with: "Reply `go` to build Step 1, or tell me what to change." Do not write code until the user says go. After that, build one checklist step at a time, run its acceptance check, and tick its box in the spec.

## Mode B · Reverse: `prd <path>` → `PRD_Viewer.html`

**B1. Load the target.** Never modify the original file.

- `.html`: read the whole file, including inline scripts. Note external CSS, JS and images.
- Component (`.jsx`, `.tsx`, `.vue`, `.svelte`): make a static snapshot first. If the app runs and you have a browser tool, capture `document.documentElement.outerHTML` after the page settles and inline its stylesheets. Otherwise port the component to plain HTML with the same class names (add the Tailwind Play CDN script if the project uses Tailwind) and re-create the key interactions in vanilla JS with the component's mock data.
- No path given: list candidate `.html` files (root, `public/`, `dist/`, `examples/`) and ask which one.

**B2. Scan and number the regions.** Pick 4 to 9 business regions in reading order (top to bottom, left to right): filter or search bar, KPI cards, main table or list, forms, modals and drawers, status tags, primary action groups, charts.

- Number the container, not the leaf: the card around the table, not each cell.
- In the copy you will embed, add `data-prd-badge="N"` (N = 1, 2, 3...) to each region's root element. Change nothing else in the markup.
- Hidden UI (modal, drawer, popover): put N on its panel and the same N on the button that opens it, so the badge shows before it opens.

**B3. Extract each region. Evidence beats guesses.** Read the markup and every function that renders or handles the region, then write one annotation:

- `rules`: what happens on click, input, hover and Enter, with conditions. Cite function names in backticks (`settleRow()`).
- `fields`: every visible column and input: name, type or format (list all enum values), source or formula, required and default.
- `formula`: any number the UI calculates, written as an equation with units, sign and rounding.
- `guardrails` (the AI diagnosis): what the code is missing, checked against the five edge-case categories and the usual vibe-coding gaps: no empty state, no loading state, no debounce or duplicate-submit guard, no confirmation on destructive or financial actions, no length or format validation, no permission check, no version check on edits, a synchronous call that should be an async job, magic numbers. Each guardrail states what you observed, why it matters and the fix ("`exportCsv()` runs on every click and builds the file synchronously; add a 1 s debounce and move exports over 5,000 rows to an async job"). Never report a gap you did not verify in the code.
- Mark anything that is not in the code as inferred: "（推断）" in Chinese docs, "(inferred)" in English.

**B4. Draw at least two diagrams:** a `stateDiagram-v2` for each entity with a status field, a `flowchart TD` for the main flow including its failure branches, and an `erDiagram` for the entities behind the page.

**B5. Edge cases and tracking.** `edgeCases`: all five categories, at least 2 rows each, with `refs` pointing at region numbers. `tracking`: 5 to 12 events named `object_action` in snake_case (`recon_settle_click`), each with its trigger and key properties.

**B6. Compile one file.**

1. Copy `templates/left-right-prd.html` unchanged to the output path: `PRD_Viewer.html` in the same folder as the source, so relative asset paths keep working. A shell `cp` is enough; you never need to read the viewer code.
2. Replace everything between `<!-- VIBE-PRD:PROTOTYPE:BEGIN -->` and `<!-- VIBE-PRD:PROTOTYPE:END -->` with the complete prototype HTML, badges included. Keep its `<style>` and `<script>` tags. Attributes on `<html>` and `<body>` are dropped, so move body classes or styles onto a wrapper `<div>`.
3. Replace everything between `/* VIBE-PRD:DATA:BEGIN */` and `/* VIBE-PRD:DATA:END */` with `window.VIBE_PRD = { ... };` (schema below). It is a JavaScript object literal: use backticks for multi-line Mermaid code, and write `<\/script>` if a string must contain that tag.

For large files, splice with a script instead of editing by hand:

```bash
python3 - "<this skill folder>" prototype.badged.html prd-data.js PRD_Viewer.html <<'PY'
import pathlib, sys
skill, proto, data, out = (pathlib.Path(p) for p in sys.argv[1:5])
html = (skill / "templates" / "left-right-prd.html").read_text(encoding="utf-8")
def put(text, begin, end, body):
    i, j = text.index(begin) + len(begin), text.index(end)
    return text[:i] + "\n" + body.strip() + "\n" + text[j:]
html = put(html, "<!-- VIBE-PRD:PROTOTYPE:BEGIN -->", "<!-- VIBE-PRD:PROTOTYPE:END -->", proto.read_text(encoding="utf-8"))
html = put(html, "/* VIBE-PRD:DATA:BEGIN */", "/* VIBE-PRD:DATA:END */", data.read_text(encoding="utf-8"))
out.write_text(html, encoding="utf-8")
print("wrote", out)
PY
```

Delete the two temporary input files afterwards.

**B7. Verify.**

- Each `data-prd-badge` number has exactly one annotation `id`, and each `id` has a badge.
- If you can open a browser: no console errors, and the console shows `[vibe-prd] N regions, N annotations, 0 issues`. Hovering a red badge scrolls the right panel to its card.
- A broken Mermaid block shows its error inside the State & Architecture tab. Fix the syntax and reload.

**B8. Report in at most 8 lines:** the output path, the regions (N → name), guardrail counts by severity, and "Open it in a browser and hover a red badge."

## The `window.VIBE_PRD` data contract

```js
window.VIBE_PRD = {
  meta: {
    title: "Inventory & ERP Reconciliation Workbench",
    lang: "en",                 // "zh" or "en": language of the viewer's own buttons and labels
    version: "v1.0.0",
    status: "review",           // "draft" | "review" | "approved"
    updated: "2026-11-21",
    source: "raw-prototype.html",
    url: "wms.example.com/inventory"     // shown in the fake address bar
  },
  annotations: [{
    id: 1,                      // = data-prd-badge
    title: "Filter bar & natural-language search",
    type: "Filter Bar",         // Filter Bar | KPI Cards | Data Table | Action Modal | Drawer | Form | Action Group | Chart | Status Tag
    summary: "One sentence: what this region is for.",
    rules: ["**Enter** runs the query", "Reset clears every filter and the AI condition"],
    formula: { label: "Diff", expr: "diff = wms_available + in_transit - erp_qty" },    // optional
    fields: [{ name: "warehouse_id", type: "enum: SH01 | GZ02", source: "GET /warehouses", required: "Optional, default all" }],
    guardrails: [{ level: "high", text: "What is missing, as observed in the code", fix: "Concrete fix" }]   // level: high | medium | low
  }],
  diagrams: [{ title: "Reconciliation status", kind: "state", note: "optional caption", code: `stateDiagram-v2
    [*] --> PENDING` }],        // kind: state | flow | er | sequence
  edgeCases: [{ category: "empty", trigger: "...", ui: "...", fallback: "...", refs: [3] }],
                                // category: empty | boundary | network | concurrency | permission
  tracking: [{ event: "recon_settle_click", trigger: "Click Settle", properties: "sku_id, diff_qty", refs: [3] }]
};
```

Text fields accept `**bold**` and `` `code` `` and nothing else; all other HTML is escaped.

## Mermaid rules (both modes)

- IDs are ASCII without spaces. Put Chinese or long labels in quotes: `state "待对账" as PENDING`, `A["金蝶 ERP（即时库存）"]`.
- `stateDiagram-v2`: write transitions as `PENDING --> CHECKING : 差异 ≠ 0`. Keep labels free of `:`, `;` and `#`.
- `flowchart TD`: quote any label with brackets, parentheses or punctuation. Never use `end` as a node ID. Edge labels go in `-->|label|`.
- `erDiagram`: ASCII UPPER_SNAKE entity names; attributes as `type name PK "comment"` with one-word types (`decimal`, not `decimal(18,2)`); every relationship needs a label: `SKU ||--o{ STOCK : "stocked as"`.
- No HTML inside labels except `<br/>`. Prefer 6 to 15 nodes per diagram; split a bigger one in two.

## Quality bar

- A tester could write a test case from every rule, and a developer could build every field from the dictionary without asking.
- Enums list every value; numbers state their unit and precision (¥, 2 decimals; quantities are integers).
- Guardrails are specific and actionable. "Consider adding validation" does not count.
- Fewer, sharper annotations beat many vague ones.
