# External Verification Evidence Contract

## Context

Runtime-heavy projects can prove completion outside ordinary source files and
unit test output. Unity, browser E2E, mobile simulators, hardware rigs, games,
and staging smoke tests may produce the relevant logs, screenshots, traces, or
device output. The original decision predated the protocol-2
`AcceptanceReceipt`, exact normalized review subject, Change Assessment packet,
and canonical verification-evidence fingerprint now present in 0.15.2.

## Decision

Document a v1 external verification evidence convention as provider-generated
evidence ingestion, not provider invocation. External tools keep their own
runtime, trust boundary, command execution, and cleanup. They publish a small
manifest plus relative artifact references that review and handoff flows can
cite. The manifest is supporting provider evidence, not an acceptance authority.

The public reference should say explicitly that this is a convention only today:
`repo-harness` does not yet discover, summarize, or gate on these manifests
automatically. That wording avoids implying an implemented check gate before
manifest ingestion exists in code.

## 0.15.2 Integration Boundary

The current workflow has three separate authorities:

1. A provider manifest describes runtime observations and artifact digests. Its
   pass outcome is only a provider claim.
2. `verify-sprint --prepare-acceptance` freezes canonical verification evidence
   for the normalized final subject and Change Assessment selection packet. It
   currently has no automatic external-manifest ingestion field.
3. The host records the protocol-2 `AcceptanceReceipt` after the reviewer frozen
   in the contract returns a semantic disposition. The closed `source` enum
   allows only `claude-review`, `codex-review`, or `user-waiver`; a provider
   manifest cannot issue `external_pass` or authorize merge.

Therefore a runtime-heavy task must put its dependency on provider evidence in
the active contract. A project-owned verification command should compare the
manifest's subject hash and target revision with the current selection packet,
verify artifact digests, reject validation gaps that violate the contract, and
fail closed on missing, stale, partial, or unredacted evidence. An exact manual
check with concrete review evidence is acceptable when the observation cannot
be automated. Only after that contract evidence passes should
`verify-sprint --prepare-acceptance` run and the semantic reviewer inspect the
provider artifacts.

`RuntimeEvidenceReceipt` is a separate release-side exception for npm tarball,
clean-install, and installed-hook readback. It deliberately has no task contract,
review subject, or merge disposition and must not be presented as an
`AcceptanceReceipt`.

The recommended path uses the existing ignored runtime evidence surface:

```text
.ai/harness/runs/external/<task-id>/<run-id>/manifest.json
.ai/harness/runs/external/<task-id>/<run-id>/artifacts/...
```

This fits the current information lifecycle better than adding a new committed
evidence directory. Durable conclusions still belong in `tasks/reviews/`,
`tasks/contracts/`, `tasks/notes/`, or project documentation after redaction.
The manifest is cited by repo-relative path from durable review files, while
artifact paths inside the manifest stay relative to the manifest directory.
Provider examples should not call typical Unity/browser/mobile validation
`read_only` when those tools write ignored runtime state, caches, logs, or build
outputs.

## Non-goals

- Do not add Unity, Playwright, mobile, hardware, or staging-specific execution
  logic to repo-harness.
- Do not define a full plugin permission system in this slice.
- Do not make missing external evidence count as a pass. Missing, skipped, or
  partial external manifests remain validation gaps.

## Follow-up

Future implementation work could teach `repo-harness check` or review rendering
to discover `repo-harness.external-evidence.v1` manifests. Any such ingestion
must normalize and validate the manifest, bind it to the current exact subject
and target revision, and include it in canonical prepared verification before it
can influence acceptance. Discovery or a provider `pass` field alone must never
become an acceptance gate.
