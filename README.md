# EnerCert plugin for Claude

![EnerCert](assets/icon.png)

EnerCert is a Hungarian web application for licensed energy-performance certifiers
(jogosultsággal rendelkező energetikai tanúsítók). This plugin adds three skills that teach
Claude how to work with the **EnerCert connector**, and it points at the same EnerCert MCP server
the connector uses, so you see one set of tools.

Az EnerCert jogosultsággal rendelkező energetikai tanúsítóknak készült magyar webalkalmazás. A
bővítmény három készséget ad Claude-nak az **EnerCert-kapcsolat** használatához, és ugyanarra az
EnerCert MCP-szerverre mutat, amelyet a kapcsolat is használ.

## Skills / Készségek

| Skill                            | What it does                                                                                                                                                                                                                                                                                                                        |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `energetikai-projekt-felepitese` | Builds a new address-free project or changes an existing one: looks up materials and products in the EnerCert catalogues, asks you for missing inputs, prepares **one** plan with recalculation, gives you the approval link, applies the plan after you approve it in EnerCert, then summarises the advisory result and readiness. |
| `felujitasi-forgatokonyv`        | Prepares renovation scenarios (insulation, windows, system replacement, added PV, solar thermal or cooling, with optional cost estimates), previews them with recalculation, and compares them with the baseline after approval. Scenarios never change the baseline; payback is an advisory estimate.                              |
| `keszultseg-ellenorzes`          | Read-only review of what a project still needs: readiness items, code meanings, the next step, result freshness, and whether each gap can be prepared through the connector or must be fixed in the EnerCert app.                                                                                                                   |

## How to use it / Használat

1. You need an EnerCert account. To prepare changes, an organisation owner or admin must also
   allow project changes through AI assistants in the EnerCert app; only owners, admins and
   certifiers can author.
2. Add the plugin, then connect the **EnerCert** connector from the plugin's Connectors tab and
   sign in to EnerCert. Select project modification on the consent page if you want Claude to
   prepare changes.
3. Ask, for example: "What is still missing in my latest EnerCert project?", "Build a new
   140 m² house from these inputs and prepare it for my approval", or "Add a renovation scenario
   with 15 cm wall insulation and compare it with the current state".
4. When Claude prepares a change it gives you an approval link. **Nothing is saved until you
   approve the change set on that page in the EnerCert app; a "yes" in the chat is not
   approval.** Claude can delete thermal bridges and renovation scenarios only through such an
   approved plan; it cannot delete projects or any other data.

## Important / Fontos

- Every result is advisory (tájékoztató jellegű) and is not an energy certificate. The
  certifier remains responsible for the inputs and the certificate.
- The tools have no client, address or document fields. Address, client, documents, official
  export, OÉNY filing and certificate issuance stay in the EnerCert app.
- The EnerCert app is in Hungarian.

## Data / Adatok

- This plugin contains only Markdown instructions, three JSON manifests and an icon. It runs no
  code, has no hooks and stores nothing.
- Its only network destination is the EnerCert MCP server at https://mcp.enercert.hu/mcp,
  declared in `.mcp.json`, reached over HTTPS with OAuth 2.1 sign-in to your EnerCert account.
  No credentials are bundled.
- Through that server Claude reads your organisation's project, readiness, energy-result,
  scenario and catalogue data, and sends the change plans you ask for. Project and scenario
  names are shared only if your organisation turns name sharing on.
- What you type into the chat is processed by Anthropic under your Claude account's terms.
- Privacy: https://app.enercert.hu/privacy#kapcsolt-ai-asszisztensek. You can revoke the
  connection at any time in the EnerCert app under Beállítások → Kapcsolt AI-asszisztensek.

## Support / Támogatás

- E-mail: info@enercert.hu (support and security reports)
- Website: https://enercert.hu
- Documentation: https://enercert.hu/tudasbazis/ai-asszisztens-csatlakoztatas/

EnerCert is not affiliated with or endorsed by Anthropic.

## License

Copyright (c) 2026 Józsa Richárd e.v. All rights reserved. See [LICENSE](LICENSE).
