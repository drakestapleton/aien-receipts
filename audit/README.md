# Org audit of aien-dev

> "We believe that an open source community is only as open as we are, as transparent as we are.
> Transparency will set us free." (my words, 2026-10-26)

This directory publishes what has been checked in every `aien-dev` repository, what has not, and
how anyone can re-check it. It is separate from the signed Spark bundles in the rest of this
repository: nothing here edits or replaces a bundle.

Scope and approval: I approved workflows for this subtree on 2026-10-26. The scope is recorded in
the ADR 0033 addendum, aien-dev/aien-architecture-mv#337. Until that addendum merges, this subtree
is a proposal.

## What a stamp says, and what it does not

A stamp (`stamps/<repo>/<sha>/stamp.json`) describes one commit. It keeps two kinds of evidence apart:

- **DETERMINISTIC-CHECKED**: tools in `tools/tools.lock.json`, pinned by version and SHA-256, run by
  `tools/checks.py`. Each check records tool, version, command and the SHA-256 of its output, so
  anyone can re-run it (see [VERIFY.md](VERIFY.md)).
- **MODEL-REVIEWED**: a named model at a named effort read a stated number of files. Each review
  records the files covered out of the total and the SHA-256 of its report. A model review is an
  opinion with evidence, not a proof; partial coverage is shown as partial.

Verdicts are `PASS`, `PASS-WITH-NOTES` or `FAIL`, plus a list of NOT_RUN items. `tools/stamp.py`
refuses a stamp that claims a better verdict than its own contents allow: an open high or critical
finding forces `FAIL`; any NOT_RUN item, incomplete check, untriaged hit, open finding, incomplete
review or missing review caps it at `PASS-WITH-NOTES`.

Upstream forks are `UPSTREAM-FORK`, pinned to the upstream merge-base; only my diff is checked and
reviewed. The 11 private repositories appear only as `private-01` .. `private-11` with a salted
SHA-256 commitment; their stamps are self-reported, because nobody outside can re-run them.

## Checks

| Id | Check | Tool |
|---|---|---|
| C1 | secrets in the full history | gitleaks (`--redact`) |
| C2 | root LICENSE, SPDX headers and package-manifest licence fields agree (by licence family) | checks.py |
| C3 | home paths, private and CGNAT IPv4 addresses, non-noreply e-mail addresses (e-mail and IP values hashed in the output) | checks.py |
| C4 | relative Markdown links resolve at the commit | checks.py |
| C5 | static analysis | semgrep with rules pinned by commit (config rebuilt by `tools/semgrep_config.py`, SHA-256 in the lock) |
| C6 | GitHub workflow hygiene | actionlint, zizmor |
| C8 | status words (PASS, QUALIFIED, DEPLOYED ...) with nothing cited on the same line | checks.py |
| C9 | committed crumb data equals what `crumb compile` writes | crumb, built from pinned source |
| C11 | known advisories in lock files (live database, dated, not byte-reproducible) | osv-scanner |

`FLAGGED` means the tool reported something that a person still has to triage; it is not a finding
until triaged. `INCOMPLETE` means the tool did not cover every file (for example a semgrep parse
error); it never counts as a pass.

## Coverage on 2026-10-26

No stamp is published yet. The table is the state of the working run behind the first stamps,
each repository at its default-branch HEAD on 2026-10-26: C2 and C4 from checker
`aien-audit-checks/3` (nested licences, SPDX header grammar, full-tree link resolution), every
other column from `aien-audit-checks/2`. P = PASS, F = FLAGGED, I = INCOMPLETE, N = NOT_RUN.
A FLAGGED cell is raw tool output; per-item triage (false positive, accepted, defect, patched)
happens before a stamp and is recorded in the stamp, not in this table.

| Repository | Kind | Commit | C1 | C2 | C3 | C4 | C5 | C6 | C8 | C9 | C11 | Model review |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| .github | FIRST-PARTY | `2a9e2282ef9a` | P | P | F | F | P | P | F | P | P | NONE |
| aegis-runtime-mv | FIRST-PARTY | `a39788b0eaaa` | P | P | F | F | I | F | F | P | F | NONE |
| agent-archive | FIRST-PARTY | `e102eb6bf343` | P | P | P | P | P | P | P | F | P | NONE |
| aien-architecture-mv | FIRST-PARTY | `fabf7b765a29` | F | F | F | F | I | F | F | P | P | NONE |
| aien-edge | FIRST-PARTY | `fce984350062` | P | P | F | P | P | P | P | P | F | NONE |
| aien-harness | ARCHIVED | `2da4b5a3a010` | P | P | F | P | F | F | F | P | P | NONE |
| aien-local-stack | ARCHIVED | `2ef0b913657d` | P | P | P | P | F | F | P | P | F | NONE |
| aien-protocols-mv | FIRST-PARTY | `f3785177fcaa` | P | P | F | P | I | F | P | P | F | NONE |
| aien-receipts | FIRST-PARTY | `2998d9888292` | F | P | P | P | P | P | F | P | P | NONE |
| aien-rig | FIRST-PARTY | `2a9bef612b59` | P | P | P | P | P | P | P | P | P | NONE |
| aien-sovereign-core-mv | FIRST-PARTY | `f30eb963918d` | F | P | F | F | I | F | F | P | F | NONE |
| aien-train | FIRST-PARTY | `b6c33e3a8261` | P | P | P | P | P | P | P | P | P | NONE |
| aien-translate | FIRST-PARTY | `fc09dfe90e01` | P | P | P | P | F | P | P | P | P | NONE |
| aien-yardsticks | ARCHIVED | `6c66814788b5` | P | F | F | P | F | P | P | F | P | NONE |
| aienix | FIRST-PARTY | `292a38cd0357` | P | P | P | P | P | P | P | F | P | NONE |
| AienOS-nv | FIRST-PARTY | `240ee9affd10` | P | P | F | P | F | P | P | P | P | NONE |
| aienos.com | FIRST-PARTY | `8139afd7e231` | F | P | F | P | I | F | P | F | P | PARTIAL: live-site security survey (no shell; several items NOT_RUN); source tree not model-reviewed |
| aienux | FIRST-PARTY | `659ddd2d7e3e` | F | P | F | P | I | F | F | F | P | PARTIAL: 8 slices planned; 1F, 2F, 2O failed on HTTP 429; 3F 37/127, 3O 15/127, 4F 15/117, 4O 41/117 files; others not started |
| allen | FIRST-PARTY | `a36dcfc6b69c` | P | P | F | P | P | P | P | P | P | NONE |
| arm-trusted-firmware | UPSTREAM-FORK | `92591fe59f5b` | P | F | P | P | P | P | P | P | F | NONE |
| atlas | FIRST-PARTY | `c624eb489275` | P | P | P | P | I | P | F | P | P | NONE |
| benchmarks-mv | FIRST-PARTY | `0f369dbc852b` | F | P | F | F | I | F | F | P | F | NONE |
| cortex | FIRST-PARTY | `e9f87bc8b61f` | P | P | P | P | P | P | P | P | P | NONE |
| cortex-rs | ARCHIVED | `3f9365fb37f1` | P | P | F | P | I | F | F | P | F | NONE |
| crumb-spec-mv | ARCHIVED | `e2dc22ea78df` | P | P | F | P | F | F | F | F | F | NONE |
| CUDApp | FIRST-PARTY | `780a0af1f08c` | P | F | F | P | I | P | F | P | P | PARTIAL: one reviewer pass (CudaAuditFable, report outside this repo); paired security-reviewer pass has no report |
| edk2 | UPSTREAM-FORK | `d7d4f09ff815` | P | P | P | P | P | P | P | P | P | NONE |
| edk2-non-osi | UPSTREAM-FORK | `2f029c7fdd3e` | P | F | P | P | P | P | P | P | P | NONE |
| edk2-platforms | UPSTREAM-FORK | `8e1779b538bc` | P | P | P | P | P | P | P | P | P | NONE |
| encounter | ARCHIVED | `4937786098a2` | P | F | P | P | P | P | P | P | P | NONE |
| harvester | ARCHIVED | `0fa381acd0cf` | P | P | F | P | F | F | F | P | F | NONE |
| interplane-mv | FIRST-PARTY | `1de1b0065b2e` | F | P | F | P | I | F | F | P | F | NONE |
| interplane.io | FIRST-PARTY | `62d5b27beea1` | P | P | P | P | P | P | F | F | F | NONE |
| mobhub | FIRST-PARTY | `5e1130654715` | F | P | F | P | P | P | F | F | F | NONE |
| MojoLlama | ARCHIVED | `249949d1c3fe` | P | P | F | P | I | F | P | P | F | NONE |
| odysseus | UPSTREAM-FORK | `27764b0c8b26` | P | P | F | P | I | P | P | P | P | NONE |
| omega-bridge | FIRST-PARTY | `358c50510579` | P | P | F | P | I | P | F | P | P | NONE |
| omega-harness | FIRST-PARTY | `eb07daf4ae9a` | P | P | P | P | P | P | P | P | P | NONE |
| omega-mv | FIRST-PARTY | `77ebd3fd5910` | F | F | F | P | I | F | F | F | P | NONE |
| omega-projects | FIRST-PARTY | `4f405e1093b9` | P | P | P | P | P | P | P | P | P | NONE |
| omnet | FIRST-PARTY | `83f605b14720` | F | P | F | F | I | P | P | P | N | NONE |
| omtex | FIRST-PARTY | `9071a0685922` | P | P | P | P | I | P | F | P | P | NONE |
| open-humanity | FIRST-PARTY | `38c0c1c48de5` | F | P | F | F | F | P | F | P | F | NONE |
| physics-mv | FIRST-PARTY | `8e92aa629c94` | P | P | F | P | I | F | F | P | P | NONE |
| physics0 | ARCHIVED | `fe587d2c7433` | P | P | P | P | P | P | P | P | P | NONE |
| pi-register | FIRST-PARTY | `4935b8a03ca9` | P | P | F | P | P | P | P | F | P | NONE |
| rad-id-sync | ARCHIVED | `235666135afd` | F | P | F | P | F | F | F | F | P | NONE |
| readings-window | FIRST-PARTY | `c6e2ab345983` | P | P | P | P | P | P | P | F | P | NONE |
| rpi5-uefi | UPSTREAM-FORK | `da0cceee29ed` | P | F | P | P | P | P | P | P | P | NONE |
| sovereign-intelligence | FIRST-PARTY | `75662d15ff53` | F | P | F | P | I | F | F | P | P | NONE |
| spark-adapters | ARCHIVED | `edd9aa10c0ed` | P | P | F | P | F | F | F | P | P | NONE |
| spark-connector | FIRST-PARTY | `c25797453076` | P | P | P | P | P | P | P | P | P | NONE |
| spark-crumbs-mv | ARCHIVED | `156633e1af13` | P | P | F | P | F | F | F | P | P | NONE |
| spark-debugger | ARCHIVED | `dc907a4ebc0d` | P | P | F | P | F | F | F | P | P | NONE |
| spark-dream | ARCHIVED | `fc8726847e9f` | P | P | F | P | F | F | F | P | P | NONE |
| spark-hive | ARCHIVED | `52f6b9cc1a0c` | P | P | F | P | F | F | F | P | P | NONE |
| spark-inquisitor | ARCHIVED | `0efa3e4dcd26` | F | P | F | P | I | F | F | P | P | NONE |
| spark-rsi-mv | FIRST-PARTY | `fc13d3f77e6c` | F | P | F | F | I | F | F | P | F | NONE |
| spark-supervisor | FIRST-PARTY | `371ffbebfbd8` | P | F | F | P | F | F | F | P | P | NONE |
| synapse | FIRST-PARTY | `099a980fedab` | F | P | F | P | I | F | F | P | P | NONE |
| t3code | UPSTREAM-FORK | `ecad5ad8fe45` | P | P | F | P | P | P | P | P | F | NONE |
| translate | UPSTREAM-FORK | `7e7c982ebc03` | P | P | P | P | P | F | P | P | P | NONE |
| private-01 | FIRST-PARTY | — | P | F | F | P | F | P | P | P | P | NONE |
| private-02 | FIRST-PARTY | — | P | P | F | P | I | P | P | P | P | NONE |
| private-03 | FIRST-PARTY | — | F | F | F | P | P | P | P | P | P | NONE |
| private-04 | FIRST-PARTY | — | P | F | F | P | I | P | P | P | P | NONE |
| private-05 | FIRST-PARTY | — | P | F | F | P | I | P | F | F | P | NONE |
| private-06 | FIRST-PARTY | — | P | P | F | P | F | P | F | P | P | NONE |
| private-07 | FIRST-PARTY | — | P | P | P | P | I | P | F | P | P | PARTIAL: one reviewer pass, report private; no security-reviewer report |
| private-08 | FIRST-PARTY | — | P | P | F | P | I | P | P | P | P | NONE |
| private-09 | FIRST-PARTY | — | F | P | F | P | I | P | F | P | P | NONE |
| private-10 | FIRST-PARTY | — | F | P | F | P | I | P | P | P | P | NONE |
| private-11 | FIRST-PARTY | — | P | F | F | P | I | P | F | P | P | NONE |

The public repository `aien` was created after this pin set; it is counted in `loc/latest.json`
(flagged `unlisted`) but has not been checked yet.

Model reviews still to run are listed in the pull request that added this directory. Triage of the
FLAGGED items is part of each stamp, not of this table.

## Disclosures

Problems the audit found in already-published claims. Private repositories appear by their
`private-NN` id; historical bytes are never edited, corrections sit beside them.

- 2026-10-26, private-09 (Omega evidence shelf): the historical receipts M8
  (`omega_program_core_qualification_receipt.json`) and M9
  (`omega_synthesis_v0_qualification_receipt.json`) say `QUALIFIED / PASS`, but their provenance
  does not support it. Both were regenerated in omega-mv commit `469f53c1` by
  `tests/run_m8_gates.sh` and `tests/run_m9_gates.sh`, which write a hardcoded earlier commit
  identity; the commit M9 names does not exist in omega-mv. Disposition:
  `REFUSED_UNSUPPORTED_PROVENANCE` for admission, recorded in adjacent `.correction.json` files
  (M8 merged, M9 pending review). The original bytes are unchanged. Proposed upstream fix: the gate
  scripts record `git rev-parse HEAD` of a clean tree instead of a default (NOT_RUN: no omega-mv
  change yet).

## Live line count

`loc/latest.json` is the count of first-party code in every public repository at its current HEAD,
made with scc at the version in `loc/repos.json`. Forks count only the lines I added on top of the
upstream merge-base. `loc/history.jsonl` gains a line only when a count changes. Private
repositories are not counted here.

## Jobs

- `.github/workflows/audit-attest.yml` runs when a stamp is pushed: it validates the stamp, re-runs
  its checks at the stamped commit on a GitHub-hosted runner, and signs it with a GitHub artifact
  attestation.
- `.github/workflows/audit-refresh.yml` runs daily: it marks stamps `STALE` when a repository's
  default branch has moved past the stamped commit (`status.json`) and recounts lines of code. It may
  change only `status.json`, `loc/latest.json` and `loc/history.jsonl`, and fails without pushing if
  anything else changed.

## Host dependencies

The checker and its helpers are Python, and the scanners are third-party binaries. They are pinned in
`tools/tools.lock.json`. The project rule is first-party code in Omega; these Python tools are a recorded
exception until an Omega port exists (proposal: port `stamp.py`, `status.py` and `loc.py`'s
bookkeeping to `.om` first, since they are pure data checks; the scanners stay as verification oracles).
