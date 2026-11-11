# aien-receipts (append-only shelf of signed AIEN verification bundles)

> Data-only repository. When the Spark judge (the verification pipeline on the project's NVIDIA DGX Spark) checks a commit of an `aien-dev` repository, it signs the result and publishes it here as `<repo>/<sha>/bundle.json`. Consuming repositories verify a bundle with `aien-test verify-bundle` against the public key and gate list on their own base branch. Bundles have no code, no build and no workflow. The one exception is the org audit under `audit/` (see below).

Shared rules: read https://www.aienos.com/agents/RULES.md first. This file adds what is specific to this repository; where the two differ, this file wins here.

## Scope

- Applies to the whole repository.
- `<repo>/<sha>/bundle.json` directories are published data. `<repo>` is the repository name under `aien-dev`; `<sha>` is the full 40-character commit hash that was verified. A commit that has not been verified has no directory.
- Hand-written files are only `README.md`, `docs/verify-one-bundle.md`, `docs/receipt-index.md`, `tools/replay_receipt.py`, `tests/test_replay.py`, `LICENSE`, `NOTICE` and this file. Documentation changes to those are the only changes an agent may propose.
- `index/receipt_index.json` is written only by `tools/replay_receipt.py admit` (see `docs/receipt-index.md`). Never type a digest, commit or verdict into it by hand.
- Bundles are written only by the AIEN verification pipeline on the DGX Spark (commits titled `publish N bundle(s)`). An agent never adds, edits, moves, reformats or deletes a bundle, not even to fix a typo or pretty-print JSON.

## Installation

```bash
git clone https://github.com/aien-dev/aien-receipts.git
cd aien-receipts
```

Nothing to build. Reading bundles needs only Git and a JSON reader. Full signature verification needs the `aien-test` tool from `aien-sovereign-core`, built with Cargo; `docs/verify-one-bundle.md` gives the pinned commits and toolchain.

## Usage

```bash
# Every bundle parses as JSON (633 at the time of writing)
python3 -c "import json,glob; fs=glob.glob('*/*/bundle.json'); [json.load(open(f)) for f in fs]; print(len(fs), 'ok')"

# Fingerprint of the bundle used in the walkthrough
sha256sum aien-architecture/1a3594340c70969c245554b07062efd8ead8a026/bundle.json
# expected: cac84aa17376d4349e13d16ba935e349a24db421eaf44cb2bb6be2be5263612a

# Replay the receipt index (needs GitHub access, Git and OpenSSL 3); last line is the result
python3 tools/replay_receipt.py replay --all
# expected last line: INDEX receipt_index.json v2: 16/16 entries reproduce as indexed rc=0
python3 -m unittest discover -s tests -v   # offline
```

A bundle's top-level fields are `schema`, `repository`, `commit`, `executor`, `plan`, `checks`, `verdict`, `started_utc`, `finished_utc`, `tree_digest`, `graph_digest`, `receipt_id_scheme` and `signature`. Each entry in `checks` carries `check_id`, `gate`, `backend`, `exactness`, `required`, `status`, `verdict` and `receipt`.

To verify a bundle, follow `docs/verify-one-bundle.md`: it clones the shelf, the source repository and the verifier at fixed commits, reads `.aien/verify.pub` and `.aien/always` from the base commit, and runs `aien-test verify-bundle ... --pubkey verify.pub --require <gates>`. Exit 0 is PASS, 1 a verified FAIL, 2 NOT_RUN or a usage error, 3 REJECTED. Never treat a nonzero exit as a pass. That recipe needs network access and Cargo; it has not been reproduced in the documentation review environment, as the document itself says.

## Conventions

- Append only. A bundle for a given `<repo>/<sha>` is published once and never rewritten. This is a convention: the `main` ruleset blocks force pushes and branch deletion, not edits.
- A new verification of the same commit must carry the same bytes.
- Workflows are allowed only for the org audit under `audit/`: `.github/workflows/audit-attest.yml` (signs stamps) and `.github/workflows/audit-refresh.yml` (marks stamps STALE, recounts lines of code). The owner approved this on 2026-10-26; the scope is recorded in the ADR 0033 addendum (aien-dev/aien-architecture-mv#337). No other workflow or CI may be added, and no workflow may touch a bundle. The refresh job writes only `audit/status.json`, `audit/loc/latest.json` and `audit/loc/history.jsonl`. Read `audit/README.md` before changing anything under `audit/`.
- When reasoning about a bundle, use the trust anchors from the consuming repository's base branch. Never use a key that appears only inside a bundle.
- The verdict of record is the Spark's commit status `AIEN Verification (Spark)`. A hosted check named `AIEN Verification` only bridges to it; green there means the bridge ran, not that the commit passed. A valid signature authenticates the recorded statement, not the correctness or sufficiency of the tests behind it.
- When citing a bundle in a claim, quote its path and the `backend` of its checks. A `host` backend means the gate ran on the host, not on the GPU.

## Safety and approvals

- Never edit or delete a published bundle. If a bundle looks wrong, open an issue that names its path; do not change it.
- Never commit signing keys or anything private. The signing key never leaves the verification machine; this repository only receives signed output.
- Changes to the layout, the rules above or the trust model described in `README.md` need explicit approval from the maintainer, and the pipeline doctrine in `aien-architecture` (ADR 0033) changes first.
- Do not run tampering experiments inside this checkout. The walkthrough's tamper test uses a disposable copy outside the shelf; keep it that way.

## Further reading

- `README.md`: layout, how a bundle is checked, rules.
- `docs/verify-one-bundle.md`: step-by-step verification of one real bundle, with a tamper refusal case.
- `docs/receipt-index.md`: the receipt index, its admission check (correction precedence, signature, source provenance) and how to replay or admit an entry.
- ADR 0033, the verification pipeline design: https://github.com/aien-dev/aien-architecture/blob/efbba982f5c095e3ae71c057b22faaafc11ed84d/docs/adr/0033-aien-verification-pipeline.md
- Verifier source (`bundle.rs`): https://github.com/aien-dev/aien-sovereign-core/blob/c26f4d7ac3156088469b27462390f5b08ef4f436/tools/aien-test/src/bundle.rs
