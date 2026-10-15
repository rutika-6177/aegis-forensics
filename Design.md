# AegisForensics — Design System

**Status:** Draft v0.2
**Visual direction:** Minimalist forensic workspace with a restrained Cyber HUD aesthetic
**Decision policy:** Font choices are initial proposals; confirm licensing/availability and final pairing before production UI.

## Confirmed owner decisions (2026-10-15)

- Platform: web application.
- Backend: C++17.
- Storage: local-first hybrid, with optional Supabase synchronization.
- Audience: academic project.
- Scope: full planned feature set.
- Deployment: local development and hosted deployment are intended, with hosted security gates.
- Learning style: step-by-step explanations suitable for a learner.
- AI workflow: multiple AI tools; keep these documents as the shared source of truth.
- GitHub target: public `https://github.com/rutika-6177/aegis-forensics`; existence and current contents are not yet verified.
- Cloud sync: metadata only; image bytes remain local.
- Graph visualization: Cytoscape.js.
- Initial local mode: no login; do not expose a hosted API until authentication/access controls and deployment security are resolved.
- Frontend framework: **OPEN** — owner selected “other” and wants to discuss options before choosing.


## 1. Design goals

- Feel precise, calm, technical, and credible—not like a game dashboard.
- Make evidence status and data relationships easy to scan.
- Use neon accents sparingly against a deep charcoal/navy background.
- Keep dense data tables readable and controls predictable.
- Preserve accessibility and usability when animations are disabled.

## 2. Theme

Dark theme is the default. A light theme is **[OPEN DECISION]** and is not required for the first milestone.

### Color tokens

| Token | Value | Purpose |
|---|---|---|
| `--bg` | `#080D14` | Main background |
| `--surface` | `#101925` | Panels/cards |
| `--surface-raised` | `#172333` | Elevated menus and dialogs |
| `--border` | `#27374A` | Subtle dividers |
| `--text-primary` | `#E8F0F8` | Main text |
| `--text-secondary` | `#9BAEC2` | Secondary labels |
| `--accent-cyan` | `#42D9F5` | Primary interactive accent |
| `--accent-emerald` | `#43D9A3` | Success/verified processing |
| `--accent-amber` | `#F4BC5C` | Warning/review needed |
| `--accent-crimson` | `#F16C79` | Errors/destructive actions |
| `--accent-blue` | `#6EA8FE` | Informational state |
| `--focus-ring` | `#FFFFFF` | Visible keyboard focus, with offset |

Do not rely on color alone: pair each status color with text, iconography, or a shape.

## 3. Typography

Initial font pairing proposal:
- **UI/body:** Inter, or a system sans-serif fallback.
- **Display/headings:** Athelas or a restrained serif alternative, only if it remains legible and the font license/hosting is acceptable.
- **Technical values:** system monospace stack for hashes, IDs, timestamps, and code-like values.

**[OPEN DECISION]** Confirm whether the project should use Inter + Athelas, Inter alone, or another pairing. Avoid loading multiple large font files until the design is approved. Use local/system fallbacks when offline behavior matters.

Suggested hierarchy:
- Page title: 28–32 px, semibold.
- Section title: 18–22 px, semibold.
- Body: 14–16 px.
- Table body: 13–14 px, with comfortable row height.
- Labels/meta: 11–12 px, never too faint for essential information.
- Hashes: monospace, allow wrapping or copy action rather than forcing a wide table.

## 4. Layout and spacing

- Desktop-first dashboard, responsive down to tablet and mobile widths.
- Desktop sidebar approximately 220–250 px; collapses to icon rail or drawer at narrower widths.
- Use a consistent 4 px spacing scale (4, 8, 12, 16, 24, 32, 40).
- Prefer 8–12 px corner radii; avoid excessive pill shapes.
- Panels use a subtle border and minimal shadow; avoid heavy blur that reduces contrast.
- Data tables prioritize column visibility, horizontal scrolling or responsive column hiding, and stable row alignment.
- Keep upload feedback and processing status near the upload action.

## 5. Navigation and page structure

Proposed primary navigation:
1. Overview
2. Evidence Vault
3. Duplicate Review
4. Relationship Graph
5. Cases / Workspaces
6. Exports
7. Settings

**[OPEN DECISION]** Case/workspace support and authentication requirements determine whether “Cases” is a primary navigation item in v1.

## 6. Overview dashboard

- Compact top bar with app name, active case/workspace, backend status, and settings.
- Summary metrics: evidence records, unique stored objects, exact duplicate references, open near-duplicate candidates.
- Upload dropzone with browse button and clear supported-format/size information.
- Recent activity/audit list.
- Avoid fabricated metrics: all displayed values must come from API responses or be clearly labeled as placeholders in development.

## 7. Evidence Vault

- Search box with clear action.
- Filters for case, exact duplicate, candidate match, processing status, and review status.
- Table columns: filename, evidence ID, ingested time, size, hash status, duplicate status, review status, actions.
- Long hashes should be truncated visually with copy/full-value affordance.
- Use stable sorting and visible sort indicators.
- Empty state explains how to add test evidence; error state includes retry when appropriate.

## 8. Duplicate review

- Side-by-side previews where practical.
- Show exact-match status separately from dHash distance.
- Display the threshold and distance as technical evidence, not a similarity probability.
- Actions: Confirm relationship, Reject candidate, Leave unresolved.
- Require confirmation for destructive or consequential actions.
- Explain that dHash may miss transformations and may generate false positives.

## 9. Relationship graph

- Distinguish node categories and edge types through labels, line styles, and accessible colors.
- Include selected-node detail panel.
- Provide pan, zoom, fit-to-view, reset, filters, and a text/list alternative for graph information.
- Show BFS/DFS traversal sequence as a list as well as animation.
- Force-directed positions are a visualization aid, not a factual measure of relationship strength.
- Respect reduced-motion settings.

## 10. Interaction and motion

- Use short transitions (roughly 120–220 ms) for hover, focus, and panel transitions.
- Particle trails and animated metrics are optional and must not distract from evidence.
- No continuous high-intensity background effects.
- No audio autoplay. If sound effects are ever added, default them off and provide a persistent toggle.
- Support `prefers-reduced-motion: reduce`.
- Do not animate a processing status in a way that implies progress if no measurable progress is available.

## 11. Components

Build reusable components for:
- App shell / sidebar / top bar.
- Metric card.
- Status badge with icon + text.
- Upload dropzone.
- Evidence table and filter bar.
- Hash value with copy affordance.
- Duplicate candidate comparison panel.
- Graph canvas and node details.
- Toast/alert and confirmation dialog.
- Empty/loading/error states.
- Export menu.
- Audit activity row.

## 12. Accessibility and responsive behavior

- Use semantic buttons, labels, headings, and table structures.
- Ensure keyboard operation and visible focus.
- Maintain readable contrast; test accent colors against dark surfaces.
- Do not use color as the sole signal.
- Provide accessible labels for icon-only controls.
- Respect reduced motion and browser zoom.
- At small widths, prioritize the evidence list and key status fields; use detail views for secondary metadata.

## 13. Content language

Prefer precise terms:
- “Exact duplicate” for byte-identical files after the implementation's validation policy.
- “Near-duplicate candidate” for perceptual matches awaiting review.
- “Reviewer-confirmed relationship” for a human decision.
- “Stored object” for the managed bytes.
- “Evidence record” for database metadata.

Avoid unsupported terms:
- “100% visually identical” based on dHash alone.
- “Proves the suspect's guilt.”
- “Tamper-proof” or “court-ready” without a verified assurance process.
- “Zero storage” when metadata and storage-system overhead still exist.

## 14. Design acceptance checklist

- Dark Cyber HUD is restrained, legible, and not overly decorative.
- Every important status has text/icon in addition to color.
- Loading, empty, error, and success states exist.
- Upload limits and supported formats are visible.
- Hashes can be copied and inspected fully.
- Graph has a non-animated accessible alternative.
- Reduced-motion mode works.
- No real evidence data or secrets appear in mock screenshots.
