# CLAUDE.md

Notes for whoever (human or Claude) picks this repo up next — including on a different machine.

## What this repo is

An interactive wireframe for Radiant's **case-creation form**, used to gather feedback from the
team and the PM. It is a demo artifact, not production code: no build, no dependencies, no tests.
Each wireframe is **one self-contained `.html` file** you open in a browser.

The design idea it explores is **"derive-and-hide"**: any required value the system can work out
from something already entered is dropped from the form and set behind the scenes. Case type, for
example, comes from the chosen analysis and shows as a badge instead of a field.

Two versions exist, identical except for where the clinical-signs (HPO phenotype) picker sits:

| File | Version | Picker placement |
|---|---|---|
| `case-create-signs-inline.html` | **B — inline** | version A's picker, unpacked into the form: instruction, picked terms, search row, suggestions; only the HPO tree opens a modal |
| `case-create-signs-modal.html` | A — modal | one button opens a picker dialog; the tree opens a second modal on top |

`README.md` is the demo-facing description (pros/cons, demo tips). Keep it in sync when behaviour
changes.

## Current work — read this first

**Vincent is reviewing version B (`case-create-signs-inline.html`).** Version A is not being
edited right now; the two have drifted apart as a result.

The review lives in **`revue-maquette-inline.md`**. Vincent wants it **short**: a one- or two-line
bullet per change under « Changements effectués », and a « Commentaires » section holding **only
his own comments** — not mine. Technical caveats, assumptions and open questions go here in
CLAUDE.md instead, never in that file.

Working rules Vincent set, which still hold unless he says otherwise:

- **Notes go in `revue-maquette-inline.md` first.** Only edit the wireframe when he asks for it
  explicitly. Several entries are recorded but deliberately not applied (R1, for one).
- **His notes are French-only.** No English translation to maintain for them.
- There is an idea — not built — to surface his notes inside the wireframe as a second tab in the
  existing `.notes-legend` block, under the **Codes** toggle. Nothing has been implemented.
- Vincent writes in French. Answer in French.

## Data sources

- **`analysis_catalog_qlin.csv`** — the real analysis catalog (37 analyses, tenant `qlin`). It is
  the source for `var ANALYSES` in the wireframe. Columns: `code`, `name` (French, **with the act
  number as a prefix**), `primary_condition`, `primary_condition_label_en`,
  `primary_condition_label_fr`, `condition_code_system`, `analysis_type_code`.
- **The two label columns were added on 2026-09-08**, resolved from the EBI OLS API
  (`ontologies/mondo` and `ontologies/hp`); the French side is a translation, not an ontology
  source, and needs a clinician's review.
- The full HPO ontology (~18,690 terms) is **inlined** in each HTML file. That is what makes the
  files ~1.6 MB and ~20,500 lines.

## Working on these files

They are single 1.6 MB HTML files. A few habits that make that bearable:

- **Never read a whole file.** Use `grep -n` to locate, then `sed -n 'A,Bp'` to read the region.
- **Edit with a Python script** (`python3 - <<'PY'`) doing an exact string replace with an
  `assert s.count(old)==1` guard. `sed -i` on this content is a trap.
- Structure, in order: CSS in one `<style>`, the form markup, then one big `<script>` holding data
  (`ANALYSES`, `OPTIONS`, `HPO_*`, `SUGGESTIONS_BY_ANALYSIS`), the i18n dictionaries, and the
  behaviour. Line numbers shift constantly — grep for a symbol, don't trust remembered numbers.
- **Syntax-check after editing the script block:**
  ```bash
  sed -n '/<script>/,/<\/script>/p' case-create-signs-inline.html | sed '1d;$d' > /tmp/check.js
  node --check /tmp/check.js
  ```

### Conventions inside the wireframe

- **Bilingual, French by default** (`var lang = 'fr'`). Every user-visible string is a key in the
  `en` and `fr` dictionaries, referenced from markup by `data-i18n` / `data-i18n-html` /
  `data-i18n-ph`. Adding visible text means adding both keys.
- **Selects are not `<select>`**. They are `div.ctrl.select[data-sel]` driven by `openMenu()`. The
  canonical value lives in `dataset.value`; the visible text is the translated label.
- **Reviewer annotations** — the `field_code` hints, the numbered footnotes (`note.1`…`note.8`) and
  the `.notes-legend` block — are toggled by the **Codes** button (`#docs-toggle`, which flips
  `body.hide-docs`). They are hidden by default: that is the clean view users see.
- Comments in the file explain *why* a thing is the way it is. Match that when adding code.

### Testing — there is no test runner, use headless Chrome

`google-chrome` is installed. Copy the file to a scratch directory, inject a `<script>` before
`</body>` that drives the DOM and dumps `PASS`/`FAIL` lines into a `<pre id="TESTOUT">`, then:

```bash
google-chrome --headless --disable-gpu --no-sandbox \
  --virtual-time-budget=8000 --dump-dom test.html
```

and grep the `TESTOUT` block out of the dump. `--screenshot=out.png` on the same command gives a
visual check. This was used to verify the searchable analysis menu (19 assertions) and the
condition-derivation rules — do the same rather than claiming something works untested.

## Decisions already made (don't re-litigate)

- **Analysis menu is searchable.** `openMenu()` grows a filter box once a list passes
  `MENU_SEARCH_MIN` (8 entries). It matches `name` **and** `code`, **anywhere in the string** —
  analysis names start with the act number, so a prefix-only match would never find "muscul" —
  accent- and case-insensitively, and highlights the run it matched.
- **The real 37-analysis catalog replaced the 4 fake ones**, in CSV order.
- **The primary condition is derived only from a MONDO code.** An HPO code or a blank leaves the
  field empty for the user. 34 of 37 derive; RHAB (HPO), RAPIDE and GENOR (blank) do not. The raw
  catalog code is kept in `conditionCode` either way.
- **Priority is never derived.** Prenatal used to force STAT and a fetal demise used to undo it;
  both rules were dropped (2026-09-08) — the user always picks. The rail still shows the field
  with Routine as its default.
- **Case type (germline/somatic) comes from `analysis_type_code`** — the real catalog confirms one
  type per analysis, which was an open assumption in footnote 1.
- **Every dropdown is clearable** back to its placeholder — required fields and the two that ship
  with a default (priority, id type) included — through the `↺ clear` row `withClear()` prepends
  whenever a select is `filled`. Clearing the analysis drops the derived condition suggestions;
  clearing the issuing site stops relatives inheriting it.
- **The field once called "issuing site" is "Établissement du patient" / "Patient organization"**
  (renamed 2026-09-08 — it is FHIR's `managingOrganization`, not HL7v2's sending facility). The
  internal key stays `issuing`; only the visible strings changed. **No default value.**
- **The identifier leads section 2**, labelled « Identifiant (NDD, code de l'étude, …) ». The
  id-type dropdown (MRN / Other) was removed on 2026-09-08, proband and family row alike, so the
  existing-patient lookup keys on organization + identifier: it fires whenever that pair is
  complete, whichever half moved last, and re-fires when either changes; until then it says which
  field is missing. It is mocked (`PATIENT_DB`, one record behind a 700 ms
  delay): MRN 1234 at Sainte-Justine prefills health number, names, sex and date of birth;
  anything else reports "new patient" and takes back only the values the lookup itself wrote.
- **Clinical signs follow version A's picker layout** (2026-09-08): instruction line, then the
  observed terms already picked (each with its onset), then the search row (free-text HPO search +
  "Browse the HPO tree"), then the analysis suggestions, then not-observed. A term picked anywhere
  rises into the list above the search box and is dropped from the suggestion and search lists, so
  it is never shown twice.
- **Suggestion lists**: `EPI4` was renamed to the real code `EPIL`. `CARDIO` and `TSOL` are **left
  orphaned and unused** rather than reassigned to a real analysis — that is a clinical call, not a
  technical one.

## Open questions

Full detail in `revue-maquette-inline.md`; the ones that will block work:

1. **MONDO labels now come from EBI OLS**, fetched on Vincent's go-ahead (2026-09-08). Confirm
   that source is acceptable, and get the French translations reviewed.
2. **The catalog has no English names.** In EN the form currently shows the French name.
3. **Category is not in the catalog**; Postnatal is assumed for all 37.
4. **Suggested phenotypes are one placeholder list shown for every analysis** (`SUGGESTIONS_DEFAULT`,
   set 2026-09-08), except RAPIDE and GENOR which get none. Real per-analysis lists are still a
   clinical call nobody has made; the drafted ones sit unread in `SUGGESTIONS_DRAFTS`.
5. **French HPO terms are largely machine-translated** and need a French clinician's review.
6. Whether the search should also apply to **issuing site / ordering site** — plugging in the real
   Quebec establishment list would trip the 8-entry threshold on its own.
7. **Two apparent duplicates in the catalog**: NPC and NEUTP both read « Neutropénie congénitale » ;
   HLEB and HLH both carry act number 55412. Data-entry error, or a real distinction?
8. **Switching analysis does not clear an already-derived condition** — going from MMG to RAPIDE
   (no derived condition) leaves « Maladie neuromusculaire » in the field. Original behaviour,
   untouched. Note that *clearing* the analysis does now clear the condition (asked 2026-09-08),
   so only the switch case is left inconsistent.
