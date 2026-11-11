# aien-receipts

Published signed verification summaries for the AIEN repositories. When the Spark judge (the
verification pipeline on the project's NVIDIA DGX Spark) checks a commit, it signs a bundle and
publishes that bundle here. Readers can verify its signature, commit binding, declared gate
verdicts and receipt digests. This shelf does **not** contain the underlying per-gate receipt
bodies, commands, stdout, stderr or raw logs; a signature alone cannot re-run or audit the tests.
This repository holds data only: no code, no workflows, no secrets. The single exception is the
org audit under [`audit/`](audit/README.md), which carries its own checker and two workflows.

Supports [Sovereign Intelligence (SI)](https://github.com/aien-dev/sovereign-intelligence).
Every claim the project makes is meant to resolve to a bundle on this shelf.

**Status (2026-10-26): in use.** The first bundles were published on 2026-10-16, and
the Spark publishes signed bundles when it verifies commits. The shelf includes
historical receipts for `aegis-runtime`, `aien-architecture`, `aienos`,
`aien-protocols`, `benchmarks`, `omega`, `physics` and other enrolled repositories;
the presence of a bundle for an older commit says nothing about current
qualification or merge authorization.

Since the 2026-10-24 policy change, the hosted **AIEN Checks** job on PRs and
`main` is advisory and runs portable smoke gates; it does not read this shelf.
Only the **AIEN Alpha** job on an `alpha/*` tag requires a full signed Spark
bundle for the tagged commit on `main`. The earlier `AIEN Verification` hosted
receipt bridge and required-PR-check policy are historical, not current.

## Layout

```
<repo>/<sha>/bundle.json
```

- `<repo>` is the GitHub repository name under `aien-dev`.
- `<sha>` is the full 40-character commit hash of the verified commit in that repository.
- `bundle.json` is the signed summary bundle produced by `aien-test bundle-sign` on the machine
  that ran the gates (the DGX Spark). It carries a receipt **digest** and verdict per gate,
  not the full per-gate receipt or log, plus a signature over the bundle.

Only signed summary bundles are published on this shelf. Underlying gate evidence must be
located separately in a repository's retained `evidence/` paths when published, or requested
from the run operator when it is not public; this shelf supplies no universal artifact URL.
Do not interpret a digest without its corresponding bytes as an independently audited gate run.

## How a bundle is checked

The consuming repository verifies the bundle with `aien-test verify-bundle`, using

- the repository name and commit under test,
- the public key committed in that repository as `.aien/verify.pub`, read from the base
  branch (never from the commit under test), and
- the always-run gate list `.aien/always` from the same base branch, passed as `--require`.

The signing key never leaves the verification machine. This repository only receives the
signed output, so anyone can read and re-verify a bundle with the public key.

## Receipt index

[`index/receipt_index.json`](index/receipt_index.json) lists the receipts that may be cited, from
this shelf and from `aien-dev/omega-evidence`, with pinned commits, live SHA-256 digests, the
correction that applies and the effective verdict. `tools/replay_receipt.py replay --all`
refetches every entry, checks the shelf HEAD for a correction added later, and refuses any
drift or any field it does not check. A correction file outranks the historical status, so the
omega-evidence M8 and M9 receipts are indexed as `REFUSED_UNSUPPORTED_PROVENANCE`. The admission
check, and the few values taken on trust, are described in
[`docs/receipt-index.md`](docs/receipt-index.md).

## How it works / How I built it

The [pipeline design record](https://github.com/aien-dev/aien-architecture/blob/efbba982f5c095e3ae71c057b22faaafc11ed84d/docs/adr/0033-aien-verification-pipeline.md)
identifies the problem: repeated hosted setup and hardware tests that need Spark.
The worker publishes a signed, commit-bound bundle; an independent consumer uses
base-branch trust anchors to check it rather than trusting the candidate's own key
or gate list. The signature authenticates the recorded statement, not the
correctness or sufficiency of the underlying tests.

**[Walk through one immutable real bundle](docs/verify-one-bundle.md)**: exact
source, shelf and verifier pins; base-ref public key and gate list; official
`aien-test verify-bundle` commands; and a disposable tamper/refusal case. It links
the implementation and existing tests. The optional OpenSSL signature-only
exercise was reproduced; the full verifier recipe is explicitly **not yet
reproduced** in the documentation review environment because Cargo is unavailable.
Neither a signature-only check nor a green advisory PR check replaces the full signed Alpha verdict.

## Rules

- Append only (a convention: the branch has no protection rule that enforces it). A bundle for a
  given `<repo>/<sha>` is published once and never rewritten.
  A new verification of the same commit is a new commit here that must carry the same bytes.
- No workflows run on the bundles. The only workflows are the two org-audit jobs scoped to
  `audit/` (approved by the owner on 2026-10-26; ADR 0033 addendum, aien-dev/aien-architecture-mv#337).
- Published by the AIEN verification pipeline on the DGX Spark; see the `aien-architecture`
  repository for the pipeline doctrine (ADR 0033, manifest v2).

## License

Apache-2.0. See `LICENSE`.

## Human-Directed Agentic Engineering

Current repository maintenance follows Human-Directed Agentic Engineering (HDAE):
humans define goals, architecture, constraints, acceptance criteria, and stopping
points; agents implement, execute checks, diagnose failures, and revise within those
boundaries. Human review retains responsibility for claims, acceptance, and release decisions.

For the documented method and a systems-software case study, see Gregory M. Kurtzer,
[*Ginux: Building an Independent Kernel That Runs Linux Userspace through
Human-Directed Agentic Engineering* (2026)](https://doi.org/10.5281/zenodo.23682548).
Historical evidence retains its original wording and scope; this section does not
retroactively characterize the method used to produce earlier records.
