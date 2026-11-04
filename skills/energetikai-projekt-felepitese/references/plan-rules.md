# Plan rules for `preview_project_changes` / A terv szabályai

These are the rules the EnerCert server checks beyond the published tool schema. The tool
descriptions carry the same text; when they differ, the server's current tool description wins.
Az alábbi szabályokat a szerver ellenőrzi; eltérés esetén az eszköz aktuális leírása az irányadó.

## Plan shape

- `operations`: 1–50 operations. `recalculate`: required boolean; use `true` so the result is
  recalculated when the approved plan is applied.
- Target: give `projectId` and `expectedRevision` together (`expectedRevision` =
  `editRevision` from `get_project_edit_context`), or give neither and start the plan with
  `project.create`. `project.create` is only ever the first operation.
- `opId`: unique within the plan, 1–32 ASCII letters or digits.
- `localRef`: every create has one, unique within the plan, 1–32 ASCII letters or digits. A
  later operation refers to it as `{ "localRef": "wall1" }`; it may only name an **earlier**
  create of the matching kind. Existing rows are referred to as `{ "id": "<ULID>" }`.
- At most 10 `material.create` per plan. `material.create`: `mu` is required unless
  `vapourTight` is true (then neither `mu` nor `muWet`).
- `element.create` data: `length_m` and `height_m`, or `area_m2`.
- Every patch (`*.update`) has at least one field. `system.update`: `patch`, `productId` or both.
  `scenario.update`: `name`, `description` or both.
- `assembly.derive`: `targetElementIds` are distinct element ids of this project. Deriving makes
  the organisation's own copy and relinks only the named elements; a shared assembly is never
  overwritten.
- `acknowledgeIncomplete: true` only on the user's explicit decision to save with
  `missingInputs` (otherwise `canApply` stays false).
- Request size: 64 KiB. Assemblies: at most 30 layers.

## Operations

`project.create` (address-free: `propertyType`, optional `buildingPurpose`, `buildingCategory`,
`yearBuilt`, `heatedAreaM2`), `material.create`, `assembly.create`, `assembly.derive`,
`element.create`, `element.update`, `geometry.update`, `opening.create`, `opening.update`,
`system.create`, `system.update`, `bridge.create`, `bridge.update`, `bridge.delete`,
`bridges.mode`, `ground_floor.update`, and the scenario operations (see the
felujitasi-forgatokonyv skill).

## Systems

- Categories and `systemTypeCode` values:
  - heating: `gas_condensing_boiler`, `gas_conventional_boiler`, `oil_boiler`, `wood_boiler`,
    `district_heating`, `electric_heating`
  - heat_pump: `air_water`, `ground_water`, `air_air`
  - dhw: `gas_water_heater`, `electric_water_heater`, `indirect_cylinder`
  - ventilation: `natural`, `central_mvhr`, `decentral_mvhr` (energy carrier may be omitted)
  - cooling: `split`, `multi_split`, `vrf`
  - pv: `pv_system`, `roof_mounted_pv`
  - solar_thermal: `flat_plate_collector`, `evacuated_tube_collector`
- A missing system is reported by category, for example `system.category:dhw`.
- A linked catalogue product fixes its own technical values; a `systemTypeCode` or
  `energyCarrier` that contradicts it is refused (`product_incompatible`). Do not restate values
  the product already supplies (`ambiguous_selection`), and do not overwrite a value the user
  entered (`attribute_conflict`).

### Heat pump (heat_pump)

`attributes` needs one of: `epsF`; or `erpEtaS`; or — only outside ErP scope (`erpScope` not
`in`) — `hpSourceType` and `flowTempRegime` (`air_air` needs no `flowTempRegime`).

### Domestic hot water (dhw)

Creating DHW requires `systemTypeCode` (`gas_water_heater`, `electric_water_heater`,
`indirect_cylinder`), `energyCarrier`, and `attributes.erpScope`:

- `in`: an ErP-labelled unit; then also `erpEtaS` or `epsHMV`;
- `out` or `unknown`.

The calculation needs one generator value: `efficiency` (or a selected `productId`), `erpEtaS`
or `epsHMV`. Always ask the user for `erpScope`; never assume it.

### Table modes (lossMode / controlMode "table")

- heating / heat_pump `lossMode: "table"` requires `servedAreaM2`, `distributionPlacement`,
  `designRegime`; `storagePresent: true` → also `storagePlacement`, `storageRegime`;
  `solidFuel: true` → `solidFuelType`; `pumpType` → `surfaceType`.
- `controlMode: "table"` requires `surfaceType`, `controlBase`, `emitterKind`, `pipeTopology`,
  `roomControl`, `hydraulicBalancing`; `surfaceType: free` → `radiatorPlacement`;
  `embedded` → `embeddedInsulation`; `emitterKind: radiator` → `designRegime`;
  `pipeTopology: two_pipe` → `emittersOver10`.
- dhw `lossMode: "table"` requires `dhwCirculation`, `dhwDistPlacement`;
  `dhwCirculation: true` → `dhwPumpEEI`; `dhwStoreType` → `dhwStorePlacement`.

## Thermal bridges and ground floor

- `bridge.create`: `kind: linear` — `bridge_type`, `psi_w_mk` in W/(m·K) or
  `"catalogue_typical"`, `length_m`; `kind: point` — `chi_w_k` in W/K, `count`.
- `bridge.update` is for linear bridges only. `bridge.delete` deletes one bridge; say so in the
  diff summary, because the approval page lists it as a deletion.
- `bridges.mode`: `detailed`, or `percentage` with the surcharge (then recorded ψ/χ do not
  count).
- Per-element ζ: `element.update` `zeta_category`.
- Ground floor (S2 §4.4.2): `ground_floor.update` (`wall_thickness_m` d_f, `floor_depth_m` z
  — 0 = near ground level, `soil_type`, `edge_insulation`); the perimeter is `geometry.update`
  `ground_floor_perimeter_m`; R_p comes from the floor layers' `groundRpRole`.
- App-only: an explicit U_T,p, a manual R_p and the 2D thermal-bridge solver.

## What the plan can never do

Delete a project or any data other than thermal bridges and renovation scenarios; enter or read
client, address, cadastral or document data; name projects, elements or systems; change a
material or product that already exists; change the project status; edit a frozen assessment;
issue, sign or upload a certificate; export a PDF, an evidence package or an OÉNY package.
