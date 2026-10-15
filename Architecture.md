# AegisForensics — Architecture

**Status:** Draft v0.2
**Target:** Browser frontend + native C++17 API; local-first, optional cloud synchronization
**Decision policy:** Items marked **[OPEN DECISION]** must be resolved before their implementation phase.

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


## 1. Architecture principles

1. The C++17 service owns file ingestion, cryptographic hashing, image decoding/dHash, local persistence, relationship operations, and export generation.
2. The browser is a presentation client. It does not calculate authoritative hashes or decide whether evidence is identical.
3. Local mode must function without Supabase credentials or internet connectivity.
4. Original image bytes are immutable once ingested. Transformations used for preview or dHash are derived data only.
5. Similarity edges are candidates until reviewed.
6. Cloud sync is an adapter, not a hard dependency of the core domain logic.
7. Use CMake for reproducible builds and vcpkg for Windows dependency management unless the toolchain test shows a blocker.

## 2. High-level flow

```text
Browser UI
   │ HTTP/JSON (same-origin in deployment; configured CORS for dev)
   ▼
C++17 API service (cpp-httplib)
   ├── Request validation / upload limits
   ├── Evidence service
   │    ├── SHA-256 / optional MD5 (OpenSSL EVP)
   │    ├── image decode (stb_image)
   │    ├── dHash + Hamming distance
   │    └── duplicate candidate service
   ├── DSA module
   │    ├── separate-chaining hash table (educational implementation)
   │    ├── evidence relationship graph
   │    └── BFS / DFS
   ├── Repository interfaces
   │    ├── SQLite repository
   │    └── optional Supabase sync adapter
   ├── Audit service
   └── CSV / JSON export service
          │
          ├── Local SQLite metadata DB
          ├── Local immutable evidence object directory
          └── Optional Supabase PostgreSQL + Storage
```

## 3. Proposed technology stack

| Layer | Proposed choice | Notes |
|---|---|---|
| Frontend | Browser UI; framework **[OPEN DECISION]** | Discuss framework options with the owner and obtain approval before scaffolding. |
| Styling | CSS variables, responsive CSS, SVG/CSS effects | Avoid relying on heavy animation for core usability. |
| API | C++17 + `cpp-httplib` | API route handlers should remain thin. |
| JSON | `nlohmann/json` | Parse with explicit validation and error handling. |
| Hashing | OpenSSL EVP APIs | SHA-256 primary; MD5 optional and clearly labeled. |
| Image decode | `stb_image` | Confirm supported formats and pixel limits. |
| Database | SQLite | Local source of truth in local mode. |
| Build | CMake | Pin minimum version after local toolchain verification. |
| Dependency management | Microsoft vcpkg where supported | Pin baseline/versions; `stb_image` can be vendored with license notice. |
| Graph UI | Cytoscape.js (selected) | Build a small proof-of-concept to validate the selected Cytoscape.js integration. |
| Cloud | Supabase PostgreSQL + Storage | Disabled unless explicitly configured. |
| Tests | CTest plus selected C++ test framework **[OPEN DECISION]** | Decide test framework before test phase. |
| CI | GitHub Actions | Build/test on the supported toolchain; add Linux CI if portable build is intended. |

### Toolchain decision

Preferred initial Windows path: Visual Studio 2022 or newer with the Desktop development with C++ workload, CMake, Git, and VS Code. Verify the exact compiler and dependency versions before locking them. `cpp-httplib` upstream currently notes support/testing expectations around modern Visual Studio, so do not assume every MinGW/MSYS2 configuration is supported.

## 4. Data model (initial proposal)

### `cases`
- `id` — stable UUID/text ID.
- `name` — display name.
- `created_at` — UTC timestamp.
- `status` — active/archived.

### `evidence`
- `id` — stable UUID/text ID.
- `case_id` — foreign key.
- `original_filename` — untrusted display metadata; never used as a filesystem path.
- `sha256` — indexed primary exact-duplicate lookup value.
- `md5` — optional compatibility value.
- `dhash` — 64-bit value stored as fixed-width hexadecimal or unsigned-compatible representation.
- `byte_size`, `detected_mime`, `width`, `height`.
- `object_key` — generated internal storage key, not the user filename.
- `ingested_at`, `processing_status`, `review_status`.

### `relationships`
- `id`.
- `source_evidence_id`, `target_evidence_id`.
- `relationship_type`: exact duplicate, candidate near-duplicate, reviewer-confirmed relation, or rejected candidate.
- `distance` — optional Hamming distance.
- `decision_source`, `decision_at`, `notes` — constrained and audited.

### `audit_events`
- `id`, `case_id`, `event_type`, `subject_id`, `timestamp_utc`, `details_json`.
- Application-level append-only behavior; do not claim cryptographic tamper-proofing unless a separate verified mechanism is designed.

### `sync_state` (when cloud sync is implemented)
- Local entity ID, remote entity ID, sync status, last attempt, retry count, last error category.
- Do not store service-role keys or credentials in this table.

**[OPEN DECISION]** Exact schema migrations and whether case IDs are UUIDs or another stable format should be finalized before implementation.

## 5. Evidence storage and exact duplicate semantics

- Store original file bytes under a generated, non-user-controlled object key.
- Compute SHA-256 from the received original bytes before any decode/normalization.
- Use SHA-256 as an indexed lookup. For high-assurance exactness, compare byte length and bytes before asserting byte-for-byte identity.
- If a match exists, create an ingestion/audit record that points to the canonical stored object; do not create another managed object copy.
- Do not delete user-provided originals outside the app's managed staging area.
- Never store evidence using a path constructed from an uploaded filename.
- On failure, clean up only app-owned temporary files and mark the ingestion as failed.

## 6. Perceptual hash flow

1. Validate byte size and file signature.
2. Decode to a supported pixel format with explicit dimension/pixel-count limits.
3. Convert to grayscale.
4. Resize to 9×8 using a documented interpolation method.
5. For each row and each of 8 horizontal neighbor pairs, set a bit based on `P[x,y] > P[x+1,y]`.
6. Compare dHashes using popcount of XOR.
7. Return candidate matches within the configured Hamming threshold (initial target <= 6).
8. Persist candidate edges separately from reviewer-confirmed relationships.

**Known limitation:** simple dHash is sensitive to some crops, rotations, overlays, and major visual changes. Threshold-to-similarity percentages are approximations, not calibrated probabilities.

## 7. API outline

API prefix: `/api/v1`.

- `GET /health` — process health only; do not leak configuration or secrets.
- `GET /cases` / `POST /cases` — list/create cases.
- `GET /cases/{id}/evidence` — list/search evidence.
- `POST /cases/{id}/evidence` — ingest image; enforce size and format limits.
- `GET /evidence/{id}` — metadata and processing state.
- `GET /evidence/{id}/matches` — exact and perceptual candidate matches.
- `GET /cases/{id}/graph` — nodes and typed edges.
- `POST /relationships/{id}/decision` — confirm/reject candidate with audit event.
- `GET /cases/{id}/traversal?algorithm=bfs|dfs&start={id}` — deterministic traversal result.
- `GET /cases/{id}/export?format=csv|json` — export metadata.
- `POST /sync/run` — optional authenticated sync trigger; absent/disabled in local-only mode.

**[OPEN DECISION]** Final route contracts, pagination format, authentication, CORS policy, and upload transport are to be defined before frontend integration. Never expose raw local filesystem paths through the API.

## 8. Proposed repository folder structure

```text
aegis-forensics/
├── README.md
├── PRD.md
├── Architecture.md
├── Rules.md
├── Phases.md
├── Design.md
├── LICENSE
├── THIRD_PARTY_NOTICES.md
├── .gitignore
├── .gitattributes
├── .editorconfig
├── CMakeLists.txt
├── CMakePresets.json
├── vcpkg.json
├── docs/
│   ├── decisions/
│   ├── api/
│   └── testing/
├── frontend/
│   ├── package.json              # if framework/toolchain requires it
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── styles/
│   │   └── main.*
│   └── public/
├── backend/
│   ├── include/aegis/
│   │   ├── api/
│   │   ├── domain/
│   │   ├── hashing/
│   │   ├── image/
│   │   ├── graph/
│   │   ├── storage/
│   │   ├── sync/
│   │   └── export/
│   ├── src/
│   │   ├── main.cpp
│   │   ├── api/
│   │   ├── domain/
│   │   ├── hashing/
│   │   ├── image/
│   │   ├── graph/
│   │   ├── storage/
│   │   ├── sync/
│   │   └── export/
│   └── resources/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── fixtures/
│   └── README.md
├── scripts/
├── supabase/
│   ├── migrations/
│   └── config.toml
└── .github/
    └── workflows/
```

This structure is a proposal, not a requirement to create every folder immediately. The frontend file extensions and build files depend on the framework decision.

## 9. Git repositories to use

### The one application repository (new)
Use the owner-supplied target `https://github.com/rutika-6177/aegis-forensics` (intended public). The repository must be checked for existence and current contents before any push. It will contain application code, documentation, tests, schema migrations, and configuration templates.

### Upstream dependency repositories
- `https://github.com/yhirose/cpp-httplib` — C++ HTTP API.
- `https://github.com/nothings/stb` — image decoder header (`stb_image.h`).
- `https://github.com/nlohmann/json` — JSON.
- `https://github.com/openssl/openssl` — hashing and TLS library.
- `https://github.com/sqlite/sqlite` — SQLite.
- `https://github.com/microsoft/vcpkg` — dependency manager.
- `https://github.com/cytoscape/cytoscape.js` — graph visualization candidate.
- `https://github.com/supabase/supabase` — Supabase platform repository/reference.

Do not make a separate Git repository for every dependency, and do not add dependencies as Git submodules by default. Use pinned versions/package-manager manifests; record vendored code and licenses in `THIRD_PARTY_NOTICES.md`.

## 10. Deployment model

### Local development
- Browser UI runs on a development server.
- C++ API listens on loopback by default.
- SQLite and managed evidence files live in a configurable local data directory outside source control.
- Secrets are supplied through environment variables or a local ignored config file.

### Hosted demo
- Serve frontend over HTTPS and run API behind a reverse proxy or equivalent managed routing.
- Do not expose a developer machine or local evidence folder directly to the internet.
- Hosted demo should use synthetic/public-domain test images unless a reviewed privacy/security plan explicitly permits other data.
- Use TLS, authentication if needed, upload limits, rate limiting, and restricted cloud storage policies.
- **[OPEN DECISION]** Hosting provider and whether C++ service is deployed as a container or native process.

## 11. Resilience and security boundaries

- All API inputs are untrusted.
- Limit request bodies, file size, decoded dimensions, concurrency, and processing time.
- Use prepared statements for SQLite.
- Bind local development server to loopback unless deliberate network exposure is configured.
- Keep service-role credentials server-side only; never place them in frontend bundles.
- Make cloud sync idempotent and retryable; record sync status.
- Do not silently overwrite remote objects or treat a failed sync as success.
- Add a documented backup/export strategy before calling the application production-ready.
