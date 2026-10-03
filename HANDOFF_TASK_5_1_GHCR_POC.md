# Task 5.1 — GHCR PoC: Continuity Handoff

**Status:** PARTIAL — local OCI model tested; no GHCR push/pull performed. This is a continuation checkpoint, not the final Task 5.1 report. A full context update and verbatim task appendix will follow.
**Updated:** 2026-10-03 (Asia/Aden)
**Language:** Arabic preferred.

## Goal
Test GitHub Container Registry (GHCR) only as a possible storage layer for Titan Community Registry—not as the registry/query/privacy/moderation system. Official repository intended by the task: WaheedFox/Titan; WaheedFox/Titan-3 is transitional.

## Boundaries
- Synthetic data only; no real Telegram or personal data.
- No edits to Titan source, ADRs, CONTRACT, or ROADMAP; no commits/pushes to Titan/Titan-3; no production infrastructure or package on the official repo without explicit approval.
- The user explicitly authorized a narrow exception for this private continuity repository and asked for an early checkpoint followed by a fuller update. This does not authorize a GHCR package push to Titan/Titan-3.
- User previously approved one separate, uniquely named public GHCR test package with synthetic data, anonymous-pull check, then deletion—but only if safe credentials can be supplied.

## What was actually tested locally
A temporary Node script built a hand-authored OCI Image Layout 1.0.0 in /tmp and removed it after the test:
- root artifact + 2 shard artifacts; all 3 manifests parsed.
- 16 descriptor digest/size checks passed.
- A lookup loaded one selected shard out of two; a synthetic tombstone resolved as tombstone.
- A synthetic 100 KiB blob was hashed/verified.
- Retry/dedup was simulated in an in-memory publisher model (one simulated outage, then retry/replay); this is NOT GHCR behavior.
- No project/repository source files were changed by that local test.

## Blockers / not proven
- Docker CLI exists but daemon is unavailable; ORAS is absent.
- GitHub account is WaheedFox; GET /repos/WaheedFox/Titan returned 404 (not proof it does not exist or is public). Titan-3 is accessible with push/admin permission.
- Package listing returned 403: read:packages missing; reauthorization did not add it.
- requestSecrets is undefined in both ordinary and impure CodeExecution; replit secrets list returned PermissionDenied. No GHCR PAT/token was supplied, stored, or used.
- Therefore no empirical GHCR acceptance of OCI media types, push/pull, anonymous read, package visibility, duplicate tags, concurrent root updates, actual upload/download sizes, or deletion/retention behavior.

## Next steps
1. Continue in this conversation/workspace; do not offer a new project again unless the user raises it.
2. Do not ask for a token in chat. Only proceed with GHCR writes if a secure Secrets path becomes available for a short-lived classic PAT with package-only scopes (read:packages, write:packages, delete:packages as required). Otherwise finish as local-only and clearly mark GHCR points unproven.
3. If credentials become available, use a unique isolated package under ghcr.io/waheedfox/titan-ghcr-poc-<unique>; never publish to Titan/Titan-3. Test real OCI push, digest/pull, anonymous read, index-to-one-shard lookup, retry/idempotency distinction, concurrency/root update, tombstone projection, and feasible sizes. Delete only the test package afterward; downloaded copies/caches may remain.
4. Final task classification should be evidence-based. On current evidence, provisional conclusion is D — experiment insufficient; another targeted experiment required.

## Related context
Existing architecture investigation in the Titan workspace: docs/internal/investigations/shared-community-message-links-registry.md. Original task attachment: attached_assets/Pasted-Task-5-1-GHCR-Registry-Proof-of-Concept--1790964963253_1790964963260.txt (its full text will be embedded in the next update).