# AegisForensics — AI and Engineering Rules

**Status:** Draft v0.3  
**Applies to:** Every AI coding assistant and human contributor  
**Decision policy:** Resolve open choices with the project owner before implementing affected components.

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


## 1. General operating rules

1. Read `PRD.md`, `Architecture.md`, `Rules.md`, `Phases.md`, and `Design.md` before changing project structure or implementing a feature.
2. Work only on the current approved phase and its acceptance criteria.
3. Ask before changing a locked decision, adding a major dependency, changing the stack, deleting data, or introducing cloud behavior.
4. Do not invent completed features, test results, benchmarks, security properties, or deployment success.
5. Explain unfamiliar setup and code in beginner-friendly steps, including commands, expected output, and common failure recovery.
6. If a requirement is ambiguous, ask a concise question or mark the decision **OPEN**. Do not bury guesses in code.
7. Prefer small, reviewable changes. Do not rewrite unrelated files while fixing a narrow bug.
8. Keep documentation synchronized with actual implementation; distinguish planned behavior from implemented behavior.

## 2. Stack and dependency rules

### Required direction
- C++17 backend.
- `cpp-httplib` for HTTP routing/server unless a documented blocker is approved.
- `stb_image` for initial image decoding, with supported formats and limits tested.
- `nlohmann/json` for JSON.
- OpenSSL EVP APIs for SHA-256 and optional MD5.
- SQLite for local metadata.
- CMake build system.
- vcpkg or another explicitly approved reproducible dependency method.
- React + TypeScript browser frontend; confirm the build tool before scaffolding. Keep forensic authority in the C++ backend.
- Supabase is optional and must not be required for local mode.

### Dependency policy
- Do not add a library just because an AI suggests it.
- Before adding a dependency, state the reason, license, maintenance/compatibility considerations, and whether the standard library or an existing dependency suffices.
- Pin versions or a package-manager baseline. Avoid unbounded “latest” dependencies in reproducible builds.
- Review licenses and record third-party notices.
- Do not copy random code from blogs or repositories without verifying provenance and license.
- Do not add a dependency through an unreviewed install script.
- Do not create Git submodules unless the owner explicitly approves them.
- Use `stb_image` as a vendored header only with its license notice and version recorded.

## 3. Security and evidence integrity

1. Treat all uploaded bytes, filenames, MIME headers, JSON, query parameters, and image metadata as untrusted.
2. Never concatenate user input into filesystem paths or SQL queries.
3. Use generated object IDs/keys for managed evidence storage.
4. Never alter original evidence bytes. Thumbnails or normalized pixels are derived outputs and must be kept separate.
5. Compute cryptographic hashes over the original received bytes.
6. SHA-256 is the primary integrity hash. MD5 is optional compatibility/demo metadata and must not be described as collision-resistant.
7. Perceptual matches are candidate relationships, never proof of identity, provenance, or wrongdoing.
8. Do not automatically delete originals or existing records to “deduplicate” them.
9. Do not upload evidence to Supabase unless the cloud feature is explicitly configured and the user has been informed.
10. Do not put real case evidence, credentials, API keys, private keys, tokens, or `.env` files in Git.
11. Never disable TLS certificate validation to make a connection work.
12. Use parameterized SQL statements.
13. Apply request body, upload size, pixel count, and processing-time limits.
14. Do not expose stack traces, secrets, absolute filesystem paths, or sensitive configuration in API responses.
15. Treat audit logs as application-level records, not cryptographically tamper-proof logs unless a separately reviewed mechanism is implemented.

## 4. Hashing and dHash rules

- SHA-256/MD5 inputs are raw file bytes, not decoded pixels.
- Document whether hashing is streamed or buffered; prefer streaming for large files.
- dHash algorithm must be deterministic: decode, grayscale, resize to 9×8, compare adjacent horizontal pixel values, emit exactly 64 bits.
- Document the resize interpolation and bit ordering.
- Hamming distance is popcount of XOR between two 64-bit fingerprints.
- Initial candidate threshold is `distance <= 6`, configurable and tested.
- Never present `1 - distance/64` or a threshold-derived percentage as a calibrated probability of visual identity.
- Test empty, malformed, oversized, unsupported, truncated, and valid images.
- Be explicit that simple dHash may miss crops, rotations, overlays, and substantially altered variants.

## 5. Error-handling rules

- Use structured internal errors with stable error codes and safe user-facing messages.
- Return suitable HTTP status codes; do not return HTTP 200 for failed processing.
- Validate all external input at system boundaries.
- Do not swallow exceptions or ignore return values from file/database/crypto operations.
- If file writing succeeds but metadata persistence fails (or vice versa), perform safe rollback/cleanup of app-owned temporary artifacts and record the failure where possible.
- Processing states should distinguish `queued`, `processing`, `complete`, and `failed` (final names can be decided in schema design).
- Log enough context to debug, but never log file contents, secrets, or unnecessary sensitive metadata.
- Surface actionable errors in the UI and preserve a correlation/request ID where practical.

## 6. C++ rules

- Compile as C++17; use modern RAII and avoid manual resource ownership where possible.
- Use `std::filesystem` carefully with generated paths and canonical-root checks.
- Avoid global mutable state for application services.
- Keep API handlers thin; move business logic into testable services.
- Keep hash, image, graph, storage, export, and sync concerns in separate modules.
- Avoid `using namespace std;` in headers.
- Prefer `const` references for large read-only objects where appropriate.
- Validate integer conversions and guard against overflow when handling dimensions and byte sizes.
- Do not return references or pointers to expired local objects.
- Use explicit types for hash byte arrays and dHash bit representations.
- Keep compiler warnings enabled; do not silence warnings globally to obtain a successful build.

## 7. Frontend rules

- Do not invent backend API fields; follow the approved API contract.
- Keep secrets out of browser code.
- Show loading, empty, success, and error states.
- Make keyboard focus visible and controls operable without a mouse.
- Respect `prefers-reduced-motion`.
- Use text labels and accessible names; do not rely on neon color alone to convey status.
- Do not render user-supplied filenames as trusted HTML.
- Keep animation decorative and separate from evidence-processing correctness.
- Do not add sound effects that autoplay; any optional sound must be user-controllable and off by default.

### React + TypeScript rules
- Keep components focused and reusable; avoid putting the entire application in one component.
- Type API payloads and responses explicitly; do not use `any` as a shortcut for unknown API contracts.
- Keep API communication in a small service layer rather than duplicating `fetch` calls across components.
- Keep cryptographic hashing, authoritative duplicate classification, evidence storage, and BFS/DFS implementation in the C++ backend.
- Do not add frontend state-management or UI libraries without a concrete need and owner approval.

## 8. Database and cloud rules

- Use migrations for schema changes; do not silently recreate a database to fix a migration bug.
- Use transactions for related database writes.
- Enable and test SQLite foreign-key constraints.
- Keep local mode fully usable without cloud credentials.
- Supabase credentials belong in server-side environment configuration.
- Never use a Supabase service-role key in frontend code.
- Define row-level security and storage policies before enabling remote case data.
- Cloud sync must be opt-in, idempotent, observable, and retryable.
- Define conflict behavior before implementing bidirectional sync.
- Never claim cloud sync is a backup until restore procedures have been tested.

## 9. Testing rules

- Add unit tests for new algorithmic behavior and regression tests for bugs.
- Tests must not require private keys, real evidence, or live cloud services by default.
- Use synthetic or public-domain fixture images with documented provenance.
- Include tests for hash-table collisions, BFS/DFS ordering, dHash bit order, Hamming distance, duplicate pointers, database constraints, export formats, and invalid uploads.
- Integration tests should cover failed writes and interrupted ingestion where feasible.
- Do not claim tests pass unless they were actually run; include the command and result.
- Benchmarks must state dataset size, machine/toolchain, and method.

## 10. Git and collaboration rules

- The application target is `https://github.com/rutika-6177/aegis-forensics`, intended public. Existence and current contents remain **UNVERIFIED**; inspect the repository before pushing and never overwrite existing work without approval.
- Keep `main` in a buildable state; use short feature branches when practical.
- Commit in small logical units with messages such as `feat: add sha256 hashing` or `test: cover dhash bit ordering`.
- Never force-push or rewrite shared history without explicit approval.
- Never commit generated build folders, local databases, evidence images, `.env`, tokens, or private keys.
- Use `.gitignore`, `.editorconfig`, and a README from the start.
- Review `git status` and staged changes before each commit.
- Push only after the user confirms the target repository and GitHub authentication is configured.
- The AI must not claim it created a GitHub repository, authenticated, committed, or pushed unless the relevant action actually succeeded.

## 11. Definition of done

A change is done only when:
- It satisfies the relevant acceptance criteria.
- Build and applicable tests have actually been run, or unrun checks are clearly disclosed.
- Errors and boundary cases are handled.
- Documentation reflects the actual behavior.
- No secrets or user evidence are staged.
- The diff is reviewed for unrelated changes.
- Any changed/open architectural decision is recorded for owner approval.