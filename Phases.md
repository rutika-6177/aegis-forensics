# AegisForensics — Implementation Phases

**Status:** Draft v0.3  
**Approach:** One phase at a time; do not begin the next phase until the current exit criteria are reviewed.  
**Schedule:** No dates committed until toolchain and weekly availability are confirmed.

## Confirmed owner decisions (2026-10-15)

- Platform: web application.
- Backend: C++17.
- Storage: local-first hybrid, with optional Supabase synchronization.
- Audience: academic project.
- Scope: full planned feature set.
- Deployment: local development and hosted deployment are intended, with hosted security gates.
- Learning style: step-by-step explanations suitable for a learner.
- AI workflow: multiple AI tools; keep these documents as the shared source of truth.
- GitHub repository: public `https://github.com/rutika-6177/aegis-forensics`; verified on 2026-10-15. Documentation commit `ca30437` was pushed to `main`.
- Cloud sync: metadata only; image bytes remain local.
- Graph visualization: Cytoscape.js.
- Initial local mode: no login; do not expose a hosted API until authentication/access controls and deployment security are resolved.
- Frontend: React + TypeScript selected on 2026-10-15; setup and tooling will be introduced step by step.


## Phase 0 — Decisions, environment, and repository

### Tasks
- Verify the existing local Git repository and `origin` remote. The React + TypeScript framework choice and public GitHub repository are already confirmed; deployment target remains open.
- Confirm Windows compiler/toolchain and install Git, VS Code, CMake, Visual Studio C++ build tools, and vcpkg as needed.
- Keep the existing local project folder and repository history; do not create a second repository or reinitialize Git.
- Add initial docs, README, `.gitignore`, `.editorconfig`, license decision, and third-party notices placeholder.
- Check Node.js/npm and the C++ toolchain; confirm the frontend build tool before scaffolding. Establish a minimal CMake build and React + TypeScript development command.

### Exit criteria
- Fresh local clone builds/runs the “hello service” and frontend shell.
- Git remote points to `https://github.com/rutika-6177/aegis-forensics.git`; documentation commit is pushed and the working tree is understood.
- No credentials or evidence are committed.
- All decisions required for Phase 1 are recorded.

## Phase 1 — API skeleton and Cyber HUD shell

### Tasks
- Implement C++17 service startup/shutdown, `/api/v1/health`, JSON error format, and safe loopback binding.
- Build responsive app shell: sidebar/navigation, top status area, metrics placeholders, evidence table empty state, upload zone, and notifications.
- Define and document API contracts.
- Add basic frontend-to-backend health check.

### Exit criteria
- Frontend loads and displays API health state.
- Invalid API routes and malformed input return controlled errors.
- Keyboard focus, contrast, and reduced-motion behavior are checked.
- Build and smoke-test instructions work from a clean clone.

## Phase 2 — Local database and evidence ingestion

### Tasks
- Create SQLite schema/migrations for cases, evidence, relationships, and audit events.
- Implement safe local evidence directory and generated object keys.
- Implement upload validation, limits, temporary-file lifecycle, and processing states.
- Add metadata registry listing/search/filtering.

### Exit criteria
- Valid test image ingests and appears in the registry.
- Unsupported, malformed, oversized, and interrupted uploads fail safely.
- Original bytes remain unchanged.
- Database transactions and failure paths have tests.

## Phase 3 — Cryptographic hashing and exact duplicates

### Tasks
- Implement SHA-256 using OpenSSL EVP; MD5 optional and explicitly labeled.
- Index SHA-256 in SQLite and add exact-duplicate lookup.
- Store one managed object for byte-identical duplicates and reference it from additional ingestion records.
- Add audit events and UI duplicate badges.

### Exit criteria
- Identical fixture files produce the same SHA-256.
- Changed bytes produce a different digest in the tested fixtures.
- Duplicate ingestion does not create a second managed evidence object.
- Storage and database failure cases are tested.
- UI distinguishes exact duplicate from near-duplicate candidate.

## Phase 4 — dHash and candidate review

### Tasks
- Integrate image decoding.
- Implement grayscale conversion, 9×8 resize, fixed bit ordering, 64-bit dHash, and Hamming distance.
- Search candidates with threshold `<= 6`.
- Add candidate review actions and audit events.

### Exit criteria
- Deterministic tests verify dHash bit ordering and distance.
- Fixture set includes identical images, resized/re-encoded images, dissimilar images, and known failure cases.
- Candidate match is never automatically represented as a confirmed fact.
- Size/pixel-count protections are tested.

## Phase 5 — DSA structures and graph API

### Tasks
- Implement educational separate-chaining hash table with bucket count configurable and initial demo value 17.
- Implement typed evidence graph and deterministic BFS/DFS.
- Add graph and traversal endpoints.
- Test collision handling and graph edge cases (empty graph, isolated node, cycle, disconnected components).

### Exit criteria
- Hash table, BFS, and DFS tests pass.
- Traversal ordering is documented and deterministic.
- Graph relationship types are distinguishable.
- Algorithm module is independently testable without the UI.

## Phase 6 — Graph visualization and polished registry

### Tasks
- Use the selected Cytoscape.js library; validate integration with a small prototype.
- Render graph nodes/edges, pan/zoom, selection, detail panel, and filters.
- Add search, sorting, empty/loading/error states, and relationship labels.
- Add restrained micro-interactions, optional sound only if explicitly enabled, and reduced-motion support.

### Exit criteria
- Graph reflects API data accurately.
- Visual proximity is not presented as proof of factual association.
- Keyboard and screen-size checks are complete.
- No frontend action claims success before API confirmation.

## Phase 7 — CSV/JSON exports and case dossier

### Tasks
- Implement CSV and versioned JSON export.
- Include export time, case metadata, evidence hashes, review status, and relationship data.
- Add export UI and safe filenames.
- Add round-trip/schema validation tests where relevant.

### Exit criteria
- Export files parse correctly and match registry data.
- Exports do not include image bytes or secrets.
- Export event is audited.
- User-facing copy clarifies that reports are academic summaries, not certified forensic reports.

## Phase 8 — Optional Supabase synchronization

### Tasks
- Metadata-only cloud sync is selected; image bytes remain local.
- Design database migrations, authentication, row-level security, storage policies, and secret handling.
- Implement sync adapter, idempotency keys, retry state, conflict rules, and explicit user controls.
- Test cloud-disabled local mode and failure/retry behavior.

### Exit criteria
- Core functionality works without cloud configuration.
- Sync failures are visible and do not masquerade as success.
- No service-role secret reaches the browser.
- Restore/backup claims are supported by a tested procedure.
- This phase is skipped if cloud sync is not approved.

## Phase 9 — Hardening, deployment, and academic presentation

### Tasks
- Add automated tests and CI.
- Test clean build on the supported toolchain; add Linux build only if portability is in scope.
- Review upload security, filesystem boundaries, SQL, secrets, dependency licenses, and CORS.
- Deploy a demo using synthetic/public-domain test images.
- Write architecture/algorithm explanation, limitations, benchmark method, screenshots, and demo script.

### Exit criteria
- Fresh-clone build and tests pass or any known gaps are clearly documented.
- Hosted demo does not expose a local workstation or unrestricted evidence storage.
- No secrets or real sensitive evidence are in repository history.
- Academic presentation distinguishes measured results from assumptions.
- Final repository has coherent commits, README, setup steps, and third-party notices.

## Cross-phase workflow

For every phase:
1. Review the phase goals and unresolved decisions.
2. Implement the smallest vertical slice.
3. Run build and tests and report exact results.
4. Inspect `git diff` and `git status`.
5. Update docs with what actually changed.
6. Ask the owner to approve the phase exit before continuing.

## Memory file policy

Do **not** create `Memory.md` during initial documentation. Create it once implementation starts, after Phase 0 decisions and first code changes. It should summarize actual repository state, current phase, commands verified, decisions, unresolved questions, tests run, and next task. It must not replace the canonical requirements or architecture documents.