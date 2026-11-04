---
name: felujitasi-forgatokonyv
description: 'Prepare, compare and tidy renovation scenarios on an EnerCert project through the EnerCert connector: insulation, window replacement, heating or hot-water system replacement, added PV, solar thermal or cooling, each with an optional cost estimate, previewed with recalculation and approved by the certifier in the EnerCert app; then compare each scenario with the baseline. Use when the user asks what a renovation would achieve, wants a renovation package or variant, or asks to compare or remove scenarios. HU: felújítási forgatókönyv, korszerűsítési változat, hőszigetelés, ablakcsere, kazáncsere, hőszivattyú, napelem, megtérülés, összehasonlítás az alapállapottal.'
---

# Felújítási forgatókönyv / Renovation scenarios

## Magyar összefoglaló

A felújítási forgatókönyv az alapállapotra rakott módosításcsomag: **soha nem változtatja meg az
alapállapotot** és a tanúsítvány bemeneteit, az alkalmazásban „AI-javaslat” jelöléssel látszik.
A menet: a projekt beolvasása (`get_project_edit_context`, `section: scenarios` is) → a
felhasználóval egyeztetett intézkedések katalógusból vagy a felhasználó adataiból → egyetlen
`preview_project_changes` terv `recalculate: true` mellett, amely forgatókönyvenként az
alapállapothoz mért előzetes eredményt is visszaadja → jóváhagyás az EnerCert alkalmazásban
(approvalUrl; a beszélgetésben adott „igen” nem jóváhagyás) → `apply_project_changes` →
`list_scenarios` és `compare_scenarios`. A megtérülési idő tájékoztató becslés. Minden eredmény
tájékoztató jellegű, nem minősül energetikai tanúsítványnak; a felelősség a tanúsítóé. Az
eszközök nem kezelnek ügyfél-, cím- vagy dokumentumadatot.

## Ground rules

- **Scenarios never change the baseline.** They are override sets on top of the stored model.
  Say this when you propose one, so the user knows the certificate inputs stay as they are.
- **Advisory only.** Scenario results, savings and payback are advisory (tájékoztató jellegű),
  not an energy certificate and not an energy audit. The certifier remains responsible.
  **Payback is an advisory estimate**: always report it with that label, and never present it
  as a guaranteed saving or a financial recommendation.
- **No client or address data.** Never ask for or repeat an address, client name or documents.
- **Never invent values.** A new U, g, efficiency, SPF or SEER comes from the catalogue or from
  the user with its source. A cost estimate (`costEstimateHuf`) comes from the user; if they have
  none, leave it out and say payback cannot be estimated without it.
- **A chat "yes" is not approval.** Scenario plans also need approval in the EnerCert app.

## Workflow

### 1. Read the project

- Find it with `list_projects`; read `get_project_edit_context` with `section: summary`, then
  `elements`, `openings`, `systems` as needed, and `section: scenarios` for existing scenarios
  and their measures. Keep `editRevision` for `expectedRevision`.
- Read the current result with `get_energy_summary`. If it is not `fresh`, say so: a scenario
  comparison against an out-of-date baseline is not meaningful until the baseline is
  recalculated (the build skill's recalculating plan, or the app).

### 2. Agree the measures with the user

One measure sets one field of one target. Ask which elements, openings or systems each measure
applies to, and for the value and its source:

| Target (`targetKind`) | Field (`field`)                                                                                                                                                                          | Typical renovation                      |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| `element`             | `assembly_id` (a catalogue, organisation or this plan's new assembly), `u_value_override`, `zeta_override`                                                                               | wall, roof or floor insulation          |
| `opening`             | `opening_product_id` (catalogue window), or `u_value_w_m2k`, `g_value`, `frame_ratio`                                                                                                    | window replacement, e.g. triple glazing |
| `system`              | `energy_carrier`, `system_efficiency_total`, `system_eta_total`, `spf`, `eps_f`, `seer`, `heat_recovery_efficiency`, `fan_power_w`, `annual_yield_kwh_yr`, `peak_power_kwp`, `vent_type` | boiler or heat-pump replacement, MVHR   |
| `system` (no target)  | `system_create` with `category` `pv`, `solar_thermal` or `cooling`                                                                                                                       | added PV, solar thermal or cooling      |

Each measure may carry `costEstimateHuf`. A scenario takes at most 30 measures here.

### 3. One plan, with recalculation

- Use `scenario.create` (with `measures`, optional `name` and one-line `description`) for a new
  package; `scenario.measure.add` / `.update` / `.remove` and `scenario.update` to change an
  existing one; `scenario.delete` to remove one.
- If an insulation measure needs a new assembly, create it in the same plan
  (`assembly.create` with a `localRef`) and point `assembly_id` at `{ "localRef": "…" }`.
- Call `preview_project_changes` once with `recalculate: true`. The preview then returns
  `scenarioResults`: the baseline and, for up to 3 scenarios, the advisory class and values
  with deltas against the baseline. A scenario can be `unevaluable` (see the readiness glossary's
  `unevaluableReasons`) or `not_evaluated`.
- **Deletion:** `scenario.delete` is the only way the assistant can delete a scenario, and it
  happens only when the user approves that plan in the app. Deleting a scenario that was created
  in the app (not by an assistant) makes the plan `riskTier: standard`, and the approval page
  warns about it. Say this before you include a deletion.

### 4. Approval, apply, compare

1. Show the diff, the warnings and the advisory `scenarioResults`, then the `approvalUrl` with
   the reminder that only the approval page in EnerCert approves (about 15 minutes validity).
2. After the user says they approved it there, call `apply_project_changes` with the
   `changeSetId`, `digest` and a new idempotency key; retry a timeout with the **same** key.
3. Call `list_scenarios`, then `compare_scenarios` with up to 3 `scenarioIds`.
4. Summarise per scenario: change in primary energy (kWh/m²a) and CO₂ (kg/m²a), class change,
   the cost estimate, and the payback in years **labelled advisory**. Note any measure that could
   not be evaluated and why.

### 5. Wording for the summary

- HU: „Ez tájékoztató jellegű összevetés, nem energetikai tanúsítvány és nem energetikai
  auditálás. Az alapállapot nem változott. A megtérülési idő a megadott költségbecslésből
  számolt tájékoztató érték.”
- EN: "This is an advisory comparison, not an energy certificate or an energy audit. The
  baseline has not changed. The payback is an advisory estimate from the cost you gave."

## Common refusals

- `invalid_input` / `measure_exists`: the scenario already has that target and field; use
  `scenario.measure.update`. `measure_missing`: nothing to update or remove there.
  `measure_ambiguous` or `too_many_measures`: edit that scenario in the app.
- `invalid_input` / `scenario_empty_update`: `scenario.update` needs `name`, `description` or both.
- `insufficient_scope` / `scenarios_scope_required`: the user reconnects the connector.
- `conflict`: re-read `get_project_edit_context` and preview again. `expired`: preview again.
- `approval_required`: the user has not approved in the app yet.
- `compare_scenarios` `over_cap`: compare at most 3 scenarios per call.
