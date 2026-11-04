---
name: keszultseg-ellenorzes
description: 'Review what an EnerCert energy-certification project still needs, read-only: readiness checklist, code meanings from the readiness glossary, the next step, the advisory energy result and its freshness, and whether each gap can be fixed through the EnerCert connector or only in the EnerCert app. Use when the user asks what is missing, whether a project is ready, why a result is out of date, or what to do next. HU: készültség, mi hiányzik, kész-e a projekt, következő lépés, készültségi kód jelentése, elavult eredmény, mit kell még pótolni.'
---

# Készültség-ellenőrzés / Readiness review

## Magyar összefoglaló

Csak olvasó áttekintés: semmit nem módosít. A menet: `list_projects` → `get_project_readiness`
→ a kódok jelentése a `get_readiness_glossary` szótárból → a `nextStep` tétel és a blokkoló
tételek magyarázata → `get_energy_summary` (frissesség) → teendőlista, külön jelölve, mit lehet
a kapcsolaton keresztül javaslatként előkészíteni, és mi marad az EnerCert alkalmazásban (cím,
ügyfél, dokumentumok, export, OÉNY-beadás, tanúsítvány). Minden eredmény tájékoztató jellegű,
nem minősül energetikai tanúsítványnak; a felelősség a tanúsítóé. Az eszközök nem kezelnek
ügyfél-, cím- vagy dokumentumadatot.

## Ground rules

- **Read-only.** This skill only reads. Do not call `preview_project_changes` or
  `apply_project_changes` unless the user then asks for a change; that is the
  energetikai-projekt-felepitese skill's job, and it needs approval in the EnerCert app.
- **Advisory only.** Readiness and energy results are advisory (tájékoztató jellegű), not an
  energy certificate. The certifier remains responsible for the project and the certificate.
- **No client or address data.** The tools return none. A missing address or client shows only
  as a code; never ask the user to type the address or client into the chat.
- **Use the glossary's words.** Explain codes with `get_readiness_glossary` rather than your own
  guesses. Read it once per conversation; it is static.

## Workflow

1. **Find the project.** `list_projects`; if several match, ask which one. Ids are accepted in
   any letter case.
2. **Read readiness.** `get_project_readiness`. Note `nextStep` (the first item in workflow order
   that is `incomplete` or `needs_review`), every item's `status` and severity, and the detail
   codes.
3. **Translate the codes.** `get_readiness_glossary` gives, in Hungarian and English, each
   item's label, each status and severity, each detail code's meaning and the in-app place to fix
   it. Blocking = `incomplete` or `needs_review`; warnings never block; info needs nothing.
4. **Check the result.** `get_energy_summary`. If there is no stored result, say so. If `fresh`
   is false, the inputs changed since the stored calculation: the value is out of date and must
   be recalculated before it is used.
5. **Give the next step first**, then the other blocking items, then the warnings, each with
   where it is fixed (below). Keep it short; offer details on request.

## Fixable through the connector, or only in the app?

Through the connector the assistant can only **prepare a plan** that the certifier approves in
the EnerCert app, and only when project changes are enabled for the organisation and the
connection.

| Readiness item / detail code                                                                 | Through the connector (approved plan)                                 | Only in the EnerCert app                                     |
| -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------ |
| `project_data`: `property_missing`, `address_missing`, `client_missing`                      | —                                                                     | Áttekintés › Ingatlan / Ügyfelek: property, address, client  |
| `residential_scope`: `out_of_residential_scope`                                              | —                                                                     | Áttekintés › Ingatlan: property type                         |
| `geometry`                                                                                   | yes (`geometry.update`)                                               | also possible                                                |
| `ground_floor`: `ground_floor_incomplete`, `ground_floor_legacy`, `ground_floor_psi_missing` | S2 parameters, perimeter (`ground_floor.update`, `geometry.update`)   | explicit U_T,p or manual R_p                                 |
| `elements`, `u_values`: `u_values_missing`                                                   | yes (assemblies, elements)                                            | manual U override, labels, notes                             |
| `openings`: `openings_none`, `openings_unhosted`; `plausibility`: `plausibility_warnings`    | yes (`opening.create`, `opening.update`)                              | also possible                                                |
| `heating`, `dhw`, `ventilation`, `pv_solar`                                                  | yes (`system.create`, `system.update`)                                | free-text system types                                       |
| `bridges`: `bridges_detailed_empty`                                                          | yes (`bridge.*`, `bridges.mode`)                                      | the 2D thermal-bridge solver                                 |
| `tiers`: `generic_tiers`                                                                     | pick a manufacturer or sourced catalogue row instead of a generic one | mark a material verified; manufacturer and source references |
| `calc_status`, `result`: `result_missing`, `result_stale`                                    | the next approved plan with `recalculate: true` recalculates          | Eredmény oldal: újraszámolás (recalculate)                   |
| `evidence_export`: `export_pending`                                                          | —                                                                     | Export oldal: documents, evidence package, PDF, OÉNY filing  |
| `result_advisory`, `result_verified`                                                         | information only                                                      | —                                                            |

Never offer to upload documents, export, file to OÉNY or issue a certificate through the
connector; those stay in the app. A project created through the connector has no address and no
client until the certifier enters them in the app, and until then no PDF, OÉNY upload, evidence
package or certificate can be produced from it.

## Example answer shape

- HU: „A következő lépés: **Talajon fekvő padló** – hiányoznak az S2 paraméterek (blokkoló).
  Ezt előkészíthetem javaslatként, Ön az EnerCertben hagyja jóvá. Még hiányzik a cím és az
  ügyfél: ezeket csak az alkalmazásban (Áttekintés › Ingatlan / Ügyfelek) lehet megadni. A tárolt
  eredmény nem friss. Ez tájékoztató jellegű áttekintés, nem energetikai tanúsítvány.”
- EN: "Next step: **Ground floor**, the S2 parameters are missing (blocking). I can prepare them
  as a plan that you approve in EnerCert. The address and client are also missing; enter them in
  the app (Overview › Property / Clients). The stored result is not fresh. This is an advisory
  review, not an energy certificate."

## Refusals

`not_found` means the id is wrong or belongs to another organisation (both look the same):
list again, never guess. `invalid_arguments` / `invalid_cursor`: fix the call. If a tool is
missing, the connection lacks that permission; the user can reconnect.
