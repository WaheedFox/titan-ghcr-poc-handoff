# Task 5.1 — GHCR Registry PoC Handoff

**Updated:** 2026-10-03  
**Status:** Partial; local OCI model tested, GHCR not tested directly.  
**Authorization:** Standalone GHCR PoC is authorized; isolate it from Titan/Titan-3 and use synthetic data only.

## Current state
The local OCI-layout/model checks passed, but no GHCR package has been created, pushed, or pulled. Current evidence is insufficient for a production suitability claim; provisional final classification is D — another targeted experiment is required.

## Tested locally
A temporary Node.js script hand-built an OCI Image Layout v1.0.0 in /tmp and removed it after the test. No Titan/project source files were changed.
- One root artifact and two shard artifacts; 3 OCI manifest envelopes parsed.
- Manifest media type: application/vnd.oci.image.manifest.v1+json. Custom artifact types: application/vnd.titan.registry.root.v1+json and application/vnd.titan.registry.shard.v1+json. Config: application/vnd.oci.empty.v1+json. Root/shard ref-name annotations were included.
- Root payload referenced shard manifests by digest. 16 descriptor digest/size checks passed locally.
- Synthetic indexed lookup read one selected shard out of two; an active record was found and a tombstone projected as tombstone.
- A synthetic 100 KiB blob was hashed and verified.
- Retry/dedup was simulated in memory: one failure before commit, then retry and replay; stable digest and one receipt. This is publisher-model behavior, not GHCR evidence.
- No ORAS/registry client, compressed layer, actual pull, or registry API was used.

## Results and evidence limits
| Topic | Current evidence | Status |
|---|---|---|
| OCI structure, descriptors, custom media types | Hand-authored local layout only | CONFIRMED locally; GHCR acceptance NEEDS EXPERIMENT |
| Root → shard digests and selective lookup | One of two local shards read | CONFIRMED in the model; remote pull path untested |
| SHA-256 stability | Local content hashing | CONFIRMED locally; GHCR dedup/tag behavior untested |
| Retry/idempotency and receipts | In-memory simulation only | NEEDS EXPERIMENT on GHCR; do not attribute publisher semantics to GHCR |
| Concurrent root updates/generations | Not run | UNRESOLVED |
| Anonymous public pull/package visibility | No POC package | FEASIBLE per GitHub docs; not empirically confirmed here |
| Size/batching | 100 KiB local blob only | Actual 100 KiB–1 GiB transfers/timings UNRESOLVED |
| Tombstone/deletion/retention | Local projection only | Storage erasure and persistence UNRESOLVED |
| Pages Explorer, CORS/proxy/cache | Not run | UNRESOLVED |
| GHCR as complete Registry | Not the tested role; query, policy, identity, privacy, and moderation are separate | REJECTED for this architecture |
| External publisher/relay | Architecture only | FEASIBLE, not implemented or tested |

## Technical constraints and credentials
- Environment observed: Node 24.13.0, Python 3.13.11, Nix; Docker CLI 27.5.1 but Docker daemon unavailable; ORAS absent.
- GitHub App connection has repo access but no packages scope. Listing user container packages returned 403 requiring read:packages, including after one reauthorization/retry.
- requestSecrets was undefined in both ordinary and impure CodeExecution; replit secrets list returned PermissionDenied. No PAT/token was supplied, stored, or used. Never request or accept credentials in chat.
- GHCR operations require a secure registry credential path; GitHub docs describe classic PAT package scopes: read:packages for metadata/read, write:packages for push, delete:packages for cleanup. Use only required package scopes; avoid broad repo scope if possible.
- GET /repos/WaheedFox/Titan returned 404 for this connection; this does not prove the repository is absent. WaheedFox/Titan-3 was accessible with push/admin permission but remains unchanged.
- Official documentation previously reviewed: GHCR supports public anonymous access and documents a 10 GB per-layer maximum and 10-minute upload deadline; these are documented limits, not measured results. GitHub Pages documentation notes a 1 GB site size and 100 GB/month soft bandwidth limit; no GHCR-to-Pages integration was tested.

## Allowed scope and prohibitions
- Use only synthetic data and one uniquely named standalone package such as ghcr.io/waheedfox/titan-ghcr-poc-<unique> if secure credentials become available.
- Never publish to or modify WaheedFox/Titan or Titan-3. No changes to Titan source, ADRs, CONTRACT, ROADMAP, or the existing investigation; no production infrastructure.
- User approved public anonymous-read testing and cleanup of only the created test package. Make it public only for that test; downloaded copies and caches may persist. No package has yet been created.

## Architecture context
Existing investigation: docs/internal/investigations/shared-community-message-links-registry.md (source snapshot 7c43f11627a06c4341bc8aab45ba0fc3ce9b8369). It describes local SQLite per runtime, no remote publication or /link fallback, and titan_id as local rather than globally unique. ctx.send()/ctx.reply() identity registration is post-send and best-effort. GitHub/GHCR cannot prove Telegram delivery or enforce a compliant runtime.
Recommended shape remains:
```text
Titan Runtime → local durable identity/outbox → trusted provider-agnostic publisher/relay → GHCR storage (shards/index/root) → separate query/Explorer/privacy/moderation
```
Keep registration, publisher acceptance, registry publication, Telegram verification, and public visibility distinct. PUBLIC / RESTRICTED / PRIVATE projections must remain separate. A tombstone is not proof of erasure from immutable blobs, downloads, or caches.

## Next step
If a secure Secrets path becomes available in this environment, use a short-lived classic PAT with only required package scopes, then test a unique isolated package: real OCI push/pull and media-type acceptance; anonymous pull; digest and same/different-content retry behavior; two publishers racing to update root; shard-only lookup; tombstone versus actual package deletion; and synthetic sizes 100 KiB, 1 MiB, 10 MiB, 100 MiB, and 1 GiB only if safe and practical. Record methods, timings, and failures. Delete only the package created for this test. If secure credential intake remains unavailable, stop before registry writes and deliver a local-only report.

## Final judgment criteria
- A — GHCR suitable as primary public storage.
- B — GHCR usable only as limited/secondary storage.
- C — GHCR should be rejected.
- D — experiment insufficient; another targeted experiment required.
Current evidence supports **D** only. Final report must distinguish CONFIRMED, FEASIBLE, NEEDS EXPERIMENT, UNRESOLVED, and REJECTED, and assess GHCR storage, complete Registry, Pages Explorer, publisher, visibility projections, and registry identity model.

## References
- GHCR: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry
- GitHub Packages REST API/scopes: https://docs.github.com/en/rest/packages/packages?apiVersion=2022-11-28
- GitHub Pages limits: https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits
- Original Task 5.1 attachment in prior workspace: attached_assets/Pasted-Task-5-1-GHCR-Registry-Proof-of-Concept--1790964963253_1790964963260.txt.