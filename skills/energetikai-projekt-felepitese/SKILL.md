---
name: energetikai-projekt-felepitese
description: 'Build or extend an EnerCert energy-certification project through the EnerCert connector: a new address-free project or technical changes to an existing one (assemblies, elements, windows, heating, heat pump, hot water, ventilation, PV, thermal bridges, ground floor), prepared as one plan that the certifier approves in the EnerCert app. Use when the user asks to create, model, fill in, add to or correct a project in EnerCert. HU: energetikai projekt felépítése, új projekt, rétegrend, szerkezet, nyílászáró, fűtés, hőszivattyú, HMV, szellőzés, hőhíd, talajon fekvő padló felvétele vagy módosítása az EnerCertben.'
---

# Energetikai projekt felépítése / Build an EnerCert project

## Magyar összefoglaló

Az EnerCert-kapcsolattal az asszisztens **javaslatot** készít; semmi nem kerül mentésre, amíg a
tanúsító a javaslatot az EnerCert alkalmazásban (approvalUrl) jóvá nem hagyja. A beszélgetésben
adott „igen” nem jóváhagyás. A menet:

1. Ha egy kód vagy fogalom nem egyértelmű, olvassa el a `get_readiness_glossary` eredményét.
2. Anyagot, rétegrendet és terméket mindig a katalógusban keressen (`search_material_catalog`,
   `search_system_catalog`); azonosítót vagy műszaki értéket soha ne találjon ki.
3. A hiányzó vagy kétértelmű adatot (pl. HMV ErP-kör, energiahordozó, felületek, fűtött
   alapterület) kérdezze meg a felhasználótól, ne találgasson.
4. Mindent **egyetlen** `preview_project_changes` tervbe gyűjtsön, `recalculate: true` mellett.
5. Adja át az approvalUrl-t, és mondja el, hogy a jóváhagyás csak ott érvényes.
6. A jóváhagyás után `apply_project_changes` egy új idempotencia-kulccsal; időtúllépésnél
   **ugyanazzal** a kulccsal próbálja újra.
7. Végül `get_energy_summary` és `get_project_readiness`, és foglalja össze az eredményt.

Minden eredmény tájékoztató jellegű, nem minősül energetikai tanúsítványnak; a felelősség a
tanúsítóé. Az eszközök nem kezelnek ügyfél-, cím- vagy dokumentumadatot: ezeket a tanúsító az
EnerCert alkalmazásban adja meg.

## Ground rules (apply to every step)

- **Advisory only.** Every energy figure is advisory (tájékoztató jellegű) and is not an energy
  certificate. The certifier remains responsible for the inputs and the certificate. Say so
  whenever you report a result.
- **No client or address data.** The tools have no client, address, document or photo fields.
  Never ask for an address, client name, cadastral number (helyrajzi szám) or documents. If the
  user offers them, do not repeat them into a plan; tell them to enter them in the EnerCert app
  (Áttekintés › Ingatlan / Ügyfelek). Project names are generated; do not invent one.
- **Never invent ids or values.** Every `id` you send comes from a tool result in this
  conversation. Every λ, U, efficiency, SPF, ψ or product value comes from the catalogue or from
  the user, with its source. If neither has it, ask.
- **A chat "yes" is not approval.** Only the approval page in the EnerCert app approves a plan.
- **Answer in the user's language.** The EnerCert app itself is in Hungarian; quote its menu
  names as they appear (for example Beállítások → Kapcsolt AI-asszisztensek).

## Before you start

- If `preview_project_changes` is not among your tools, project changes are off for this
  connection. Explain: an organisation owner or admin must allow project changes through AI
  assistants in the EnerCert app, and the user must reconnect the connector with project
  modification selected. Only owners, admins and certifiers can author.
- For an existing project, find it with `list_projects`, then read
  `get_project_edit_context` (page with `section`: summary, assemblies, elements, openings,
  systems, scenarios, bridges). Keep its `editRevision`: it is the plan's `expectedRevision`.
  If `editable` is false, the project is locked (see `blockedCodes`); stop and tell the user to
  resolve it in the app.
- When a readiness code, status or unevaluable reason is unclear, read
  `get_readiness_glossary` once and use its wording.

## Workflow

### 1. Collect the inputs

Ask in one message for everything the plan needs and the user has not given. Typical gaps:

- building: property type, heated floor area (`heatedAreaM2`, also sets the geometry heated
  area), year built if known;
- envelope: each element's category and either length and height or area, which assembly it
  uses, and for windows count, size, U and g (or a catalogue product) and the host element;
- systems: for each system its category, `systemTypeCode` and energy carrier (`energyCarrier`;
  ventilation may omit it), and the generator value (efficiency, SPF, or a catalogue product);
- **ventilation:** always ask for the airtightness row of S3 Table 2.4 and set
  `attributes.infiltrationClass` (airtightness `poor` / `medium` / `good` / `excellent`; one or
  several facades; storeys 1–2 / 3–6 / 7+; sheltered or exposed; for example
  `medium_single_1_2_exposed`), or an explicit `infiltrationAirChangePerH` the user states. Never
  default it: without either the result reports `infiltrationClass` missing. Mechanical
  ventilation (MVHR) also needs `heatRecoveryEfficiency`, `sfpWPerM3s`, `ventDuctType`,
  `ventControlType` and `ventMotorType`;
- **boilers:** ask the boiler type (condensing, low-temperature, constant, combi) and whether it
  stands inside or outside the heated space. Never assume a condensing boiler inside;
- **hot water (DHW):** always ask for `attributes.erpScope` — `in` (an ErP-labelled unit; then
  also `erpEtaS` or `epsHMV`), `out`, or `unknown`. Never default it;
- thermal bridges: detailed (ψ/χ per bridge, or ψ `catalogue_typical`) or percentage mode;
- ground floor (S2 §4.4.2): wall thickness d_f, floor depth z, soil type, edge insulation, the
  ground-floor perimeter (geometry), and ψ_p,f if the user has it.

If an answer is ambiguous (for example "gas heating" could be a condensing or a conventional
boiler; "the usual window" has no U value), ask instead of guessing.

### 2. Look everything up in the catalogues

- Materials: `search_material_catalog` with a `query`, `category` or `materialIds` (a `tier`
  alone is refused). Prefer reviewed catalogue rows; tell the user the tier (generic G,
  manufacturer, user-defined, verified) of what you picked.
- System and window products: `search_system_catalog` with `kind: system` or `kind: opening`.
  Search by category plus a short English query, for example category `heating` + "condensing
  boiler", category `dhw` + "cylinder". A product's `requiresInputs` lists what you must still
  ask the user (systemTypeCode, energyCarrier, and for DHW always erpScope). A product's
  `systemTypeCode` or `energyCarrier` of `null` means unknown, not a default.
- If the catalogue has no match, say so. Propose `material.create` only with values the user
  gives you and their source type (datasheet, standard table, measurement, estimate). A created
  material is saved as user-defined and **not verified**, and the approval page asks the
  certifier to accept it item by item.

### 3. Build ONE plan

Put the whole change into a single `preview_project_changes` call with `recalculate: true`, so
the user approves once and the result is recalculated on apply. Follow the plan rules in
[references/plan-rules.md](references/plan-rules.md); the main ones:

- New project: omit `projectId` and `expectedRevision`, and make `project.create` the **first**
  operation. Existing project: give both `projectId` and `expectedRevision` (= `editRevision`).
- Every operation has a unique `opId`; every create has a unique `localRef` (1–32 ASCII letters
  or digits). Later operations refer to an earlier create as `{ "localRef": "…" }`, of the
  matching kind (an element's `assembly`, an opening's `hostElement`).
- Order: project → materials → assemblies → elements → openings → systems → bridges and ground
  floor → geometry.
- Heat pumps, DHW and table modes have extra required inputs (see the reference file).

### 4. Show the preview and hand over the approval link

From the preview, tell the user:

- the diff in plain words (what is created, what changes from what to what, what is deleted);
- `missingInputs` and `warnings`, and any `defaults` the server applied;
- the `calculationPreview` (advisory classes and values), marked advisory;
- the `approvalUrl`, with this message (in the user's language):
  - HU: „A javaslat még nincs mentve. Nyissa meg a jóváhagyási oldalt az EnerCertben, nézze át
    a tételeket, és ott hagyja jóvá. A beszélgetésben adott »igen« nem jóváhagyás. A javaslat
    kb. 15 percig érvényes. Ha jóváhagyta, szóljon, és végrehajtom.”
  - EN: "Nothing is saved yet. Open the approval page in EnerCert, review the items and approve
    them there. A 'yes' in this chat is not approval. The plan is valid for about 15 minutes.
    Tell me when you have approved it and I will apply it."

If `canApply` is false there is no `approvalUrl`: the plan has `missingInputs` or a blocking
problem. Ask for the missing inputs and preview again. Set `acknowledgeIncomplete: true` only
when the user explicitly decides to save an incomplete model, and tell them what stays missing.

### 5. Apply after the user approved in the app

- Call `apply_project_changes` with the preview's `changeSetId`, `digest` and a **new**
  idempotency key for this plan (16–128 characters of `A–Z a–z 0–9 _ -`; for example `ap_`
  followed by the changeSetId without `cs_`).
- On a timeout, a network error, `verification_failed` or a `retryable` error, call it again with
  the **same** key: the server returns the same receipt and never applies twice.
- Never call apply before the user says they approved, and never on a chat "yes" alone. If apply
  returns `approval_required`, send them back to the `approvalUrl`.

### 6. Read back and summarise

Call `get_energy_summary` and `get_project_readiness`. Report:

- the advisory classes and specific values, and whether `fresh` is true (if false, say the
  stored result is out of date; never present it as current);
- the blocking readiness items that remain, and where each is fixed. A new project always
  still needs its address and client, which the certifier enters in the EnerCert app; PDF, OÉNY
  upload, evidence package and certificate are not possible until then.

End with: the result is advisory, not an energy certificate, and the certifier is responsible.

## Refusals

Errors come back as JSON `{error, reason, opId?, field?, hint, editRevision?, retryable?}`. Use
`field` and `opId` to fix the exact operation. What to do for each code is in
[references/refusal-codes.md](references/refusal-codes.md). The most common:

| Code (reason)                                                        | Do this                                                            |
| -------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `invalid_input` (`required`, `invalid_value`, `erp_scope_required`)  | Fix the named field; ask the user if the value is theirs to give   |
| `invalid_input` (`product_incompatible`, `category_mismatch`)        | Match the product's type and carrier, or drop `productId`          |
| `missing_input`                                                      | Ask the user for it; preview again                                 |
| `conflict` (`revision_mismatch`, `dependency_changed`)               | Re-read `get_project_edit_context`, preview again                  |
| `approval_required` (`not_approved`)                                 | The user approves at `approvalUrl`; do not retry until they say so |
| `expired`                                                            | Preview again and send the new link                                |
| `write_disabled` / `insufficient_scope` / `reauthorization_required` | Explain the setting or ask the user to reconnect                   |
| `not_found`                                                          | The id is wrong or not in this organisation; list again            |
