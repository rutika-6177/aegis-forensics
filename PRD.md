# AegisForensics — Product Requirements Document

**Status:** Draft v0.3  
**Project type:** Academic software engineering / digital-forensics demonstration  
**Last updated:** 2026-10-15  
**Decision policy:** Items marked **[OPEN DECISION]** require confirmation before implementation.

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


## 1. Product summary

AegisForensics is a web-based, local-first digital evidence image triage application. It helps an academic user or investigator-in-training identify exact duplicate image files, surface visually similar images, inspect candidate relationships, and organize findings into a searchable evidence registry.

The initial project is a controlled academic demonstration, not a certified forensic product. It must not claim that perceptual similarity proves common origin, that a hash proves who created a file, or that the application is suitable for evidentiary use without independent validation.

## 2. Problem statement

Evidence collections may contain repeated image files and transformed variants. Reviewing each file manually wastes time and makes relationships difficult to see. AegisForensics aims to reduce repetitive review by combining:
- Cryptographic file fingerprints for byte-identical files.
- Perceptual fingerprints for candidate visual near-duplicates.
- Explicitly reviewed evidence relationships and graph visualization.
- A searchable registry and exportable case summaries.

Any duplicate-rate statement used in a presentation must be sourced and contextualized; it is not an assumed universal statistic.

## 3. Target users

### Primary audience
- Engineering student building and presenting an academic project.
- Faculty or evaluators reviewing the system's algorithms, architecture, UI, and testing.

### Secondary audience
- Investigators-in-training and digital-forensics learners using synthetic or lawfully obtained test images.

## 4. Goals

1. Demonstrate SHA-256 and optional MD5 file hashing.
2. Demonstrate a 64-bit difference hash (dHash) for candidate near-duplicate detection.
3. Demonstrate core data structures: a separate-chaining hash table with a configurable bucket count (initial demonstration target: 17), an evidence relationship graph, and BFS/DFS traversal.
4. Provide a polished, accessible Cyber HUD web interface.
5. Maintain a local evidence registry and local image store, with optional cloud synchronization after local behavior is reliable.
6. Produce reproducible tests, setup instructions, and a credible Git history suitable for an academic review.

## 5. Non-goals

- Automated determination that a person committed a crime.
- Facial recognition, identity inference, or suspect profiling.
- Automated legal conclusions or claims of evidentiary admissibility.
- Silent modification, recompression, or deletion of source evidence.
- Processing arbitrary file types in the first release.
- Multi-tenant enterprise case management in the initial academic release.
- Cloud upload of evidence by default.

## 6. Functional requirements

### FR-01 — Case workspace
- Create and select a local case/workspace.
- Each evidence record belongs to a case.
- Selected: no login for local academic mode. Hosted deployment authentication remains a required security decision before exposing the API publicly.

### FR-02 — Image ingestion
- Accept a defined allowlist of image formats, initially JPEG, PNG, and WebP if the chosen decoder supports them.
- Enforce a configurable maximum upload size and image dimensions/pixel count.
- Validate file content, not only extension or browser-supplied MIME type.
- Assign a stable evidence ID and record ingestion time.
- Never overwrite an existing evidence file because a filename matches.

### FR-03 — Exact duplicate detection
- Compute SHA-256 over original file bytes.
- MD5 may be computed for demonstration/legacy comparison, but must not be treated as the primary integrity control.
- A matching SHA-256 identifies a byte-identical file with overwhelming practical confidence; verify exact bytes when required by the implementation's assurance level.
- For exact duplicates, avoid storing a second copy when the application can safely reference the existing stored object. Record the new ingestion event and duplicate pointer.
- Do not claim “zero additional bytes” absolutely: metadata, audit records, filesystem allocation, and storage-provider behavior may consume space.

### FR-04 — Perceptual near-duplicate candidates
- Decode the image safely, convert to grayscale, resize to 9×8, and compare each adjacent horizontal pixel pair to produce 64 bits.
- Compute Hamming distance between dHashes.
- Initial candidate threshold: distance <= 6, configurable and documented.
- A match is a **candidate**, not a confirmed fact. Cropping, overlays, watermarks, rotations, and heavy transformations may defeat basic dHash; false positives are possible.
- The UI must allow a reviewer to confirm, reject, or leave a candidate unresolved.

### FR-05 — Evidence registry
- Display ID, original filename, case, ingestion time, byte size, MIME/format detection, SHA-256, MD5 (if enabled), dHash, storage state, and review status.
- Search and filter by filename, case, hash, duplicate category, and review status.
- Support stable sorting and pagination/virtualization as needed.

### FR-06 — Relationship model
- Represent records as graph nodes.
- Support typed relationships such as `exact_duplicate_of`, `candidate_near_duplicate_of`, and `reviewer_confirmed_related_to`.
- Record the actor/source and timestamp for reviewer decisions.
- Do not automatically infer a parent/original from upload order alone. **[OPEN DECISION]** Define the rule for selecting a canonical representative within a duplicate group.

### FR-07 — DSA demonstrations
- Provide separate-chaining hash table implementation for the academic DSA demonstration; bucket count 17 is a teaching/configuration value, not a claim of optimal production sizing.
- Provide BFS (level-order) and DFS (pre-order) over a documented graph.
- Traversal output must be deterministic for tests (define neighbor ordering).
- UI animation is optional to correctness; traversal results must be testable without animation.

### FR-08 — Relationship visualization
- Show nodes and typed edges, with labels and filters.
- Provide pan, zoom, selection, and node detail panel.
- Avoid presenting force-directed proximity as proof of factual relationship.
- Cytoscape.js is selected. Build a small proof-of-concept to validate the integration.

### FR-09 — Export
- Export case metadata and registry rows as CSV and JSON.
- Exports must identify the export time and schema/version.
- Never include image bytes in CSV/JSON exports.
- Include hash values and relationship/review status where available.
- Warn that exported reports are summaries, not a validated forensic acquisition format.

### FR-10 — Audit trail
- Record ingestion, duplicate decisions, relationship edits, exports, and cloud sync attempts.
- Audit entries should be append-only at the application level; this alone is not tamper-proof storage.
- Store timestamps in UTC and display them in local time.

### FR-11 — Local-first behavior and cloud sync
- Core ingest, hashing, registry, matching, graph, and exports must work offline.
- Local database and evidence files are the default source of truth in the initial release.
- Supabase synchronization is optional and must be explicitly configured.
- Cloud sync policy selected: metadata only. Original image bytes remain local; revisit only with explicit owner approval.

### FR-12 — Status and errors
- Show upload, hashing, decoding, storage, and sync progress where practical.
- Provide actionable error messages without exposing secrets or internal paths unnecessarily.
- Failed processing must not leave a falsely “complete” evidence record.

## 7. Non-functional requirements

- **Integrity:** original bytes remain unchanged; hashes are reproducible.
- **Security:** validate uploads, restrict local paths, use parameterized SQL, keep secrets out of source control, and do not disable TLS verification.
- **Privacy:** no evidence is sent to the cloud unless configured and disclosed.
- **Reliability:** recover gracefully from interrupted ingestion and database errors.
- **Performance:** establish benchmarks using a documented synthetic dataset; do not promise unsupported throughput.
- **Accessibility:** keyboard-visible focus, sufficient contrast, semantic controls, and reduced-motion support.
- **Portability:** local development on Windows; deployment target and Linux support need validation.
- **Maintainability:** modular C++ code, reproducible build, tests, pinned dependencies, and clear setup documentation.

## 8. Acceptance criteria for the first complete academic release

1. Two byte-identical files produce the same SHA-256 and are linked as exact duplicates without creating a second managed image copy.
2. A fixture set demonstrates dHash generation and threshold behavior, including known limitations.
3. Unit tests cover hash-table collisions, BFS, DFS, dHash bit generation, Hamming distance, and database operations.
4. Registry search, filters, sorting, and CSV/JSON exports work against local data.
5. A reviewer can confirm or reject a near-duplicate candidate and the decision is audited.
6. The app operates with networking disabled when cloud sync is not enabled.
7. The UI distinguishes exact duplicates from perceptual candidates.
8. Fresh-clone setup and test instructions work on the documented toolchain.
9. No secrets, real sensitive evidence, or generated build artifacts are committed.
10. A clean repository contains a license/third-party notices appropriate to selected dependencies.

## 9. Success metrics

- All core functional acceptance tests pass.
- A fresh clone can be built by following the README.
- Test data and benchmark procedure are reproducible.
- A demonstration can show ingest → hash → candidate review → graph → export.
- No known data-loss, path-traversal, or secret-leakage defects remain in the reviewed academic release.

## 10. Open decisions register

| ID | Decision | Current status |
|---|---|---|
| O-01 | Frontend framework | **RESOLVED** — React + TypeScript selected on 2026-10-15; build tool to be confirmed during setup |
| O-02 | Authentication for local academic mode | Resolved: no login locally; hosted access requires security review |
| O-03 | Cloud policy | Resolved: metadata-only sync; image bytes stay local |
| O-04 | Deployment target/provider and public demo data | **OPEN** |
| O-05 | Graph library | **RESOLVED** — Cytoscape.js selected |
| O-06 | Canonical representative rule for exact-duplicate groups | **OPEN** |
| O-07 | Maximum upload size and supported image formats | **OPEN** |
| O-08 | Repository state | **RESOLVED** — public `rutika-6177/aegis-forensics` verified; documentation commit `ca30437` pushed to `main` on 2026-10-15 |
| O-09 | Whether case/workspace separation is required in v1 | **OPEN** |

## 11. Repository and dependency starting points

The application repository is the existing public repository `https://github.com/rutika-6177/aegis-forensics`; the initial documentation commit `ca30437` is pushed to `main`. It is separate from all upstream dependency repositories. Proposed upstream projects (subject to license/version review and pinning):

- `https://github.com/yhirose/cpp-httplib` — HTTP server/client.
- `https://github.com/nothings/stb` — `stb_image` image decoding header.
- `https://github.com/nlohmann/json` — JSON handling.
- `https://github.com/openssl/openssl` — SHA-256/MD5 via EVP APIs and TLS support as configured.
- `https://github.com/sqlite/sqlite` — SQLite engine/source; a package-manager build may be more practical.
- `https://github.com/microsoft/vcpkg` — Windows-friendly C/C++ dependency management.
- `https://github.com/cytoscape/cytoscape.js` — candidate graph visualization library.
- `https://github.com/supabase/supabase` — Supabase platform/client ecosystem reference; select the exact supported client/API path before implementation.

Do not clone every upstream repository into the application. Prefer a package manager or pinned upstream release for dependencies. `stb_image` is a single-header project in the `stb` repository, not a separate official repository.