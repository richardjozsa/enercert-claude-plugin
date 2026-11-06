# Changelog

All notable changes to the EnerCert plugin for Claude. Versions follow `version` in
`.claude-plugin/plugin.json`; raise it with every release, because the directory serves the
version that is live.

## 1.0.1 — 2026-11-06

- Project building: ask for the S3 Table 2.4 airtightness row for ventilation (`infiltrationClass`), the MVHR fan inputs, and the boiler type and placement, which EnerCert 0.19/0.20 require instead of assuming them.

## 1.0.0 — unreleased

First release, prepared for submission to Anthropic's directory.

- Skills: `energetikai-projekt-felepitese` (build a project as one approved plan),
  `felujitasi-forgatokonyv` (renovation scenarios and comparison), `keszultseg-ellenorzes`
  (read-only readiness review).
- Connector reference: the EnerCert MCP server at https://mcp.enercert.hu/mcp (OAuth, no
  bundled credentials).
- Listing fields: icon, documentation, support, privacy policy and terms URLs.
