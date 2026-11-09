# om/tls: FIPS 140-3 readiness checklist

Wording: om/tls is **tested against the FIPS 140-3 / CAVP test methods, not NIST-validated**. READY
means my own evidence covers the item and a lab would have something to test; it does not mean a lab
agreed. GAP is work I can do myself. BLOCKER needs money, a signature or an outside party.

| # | item (FIPS 140-3 area / NIST document) | state | evidence or reason |
|---|---|---|---|
| 1 | Approved algorithms implemented: AES-GCM, SHA-2, HMAC, HKDF/TLS 1.3 KDF, ECDSA/RSA verify | READY | acvp/logs/run.log `ACVP PASS algos=21 tests=7623 not_applicable=7299 not_run=NONE` (NIST ACVP-Server sample vectors 975de31e; self-run, MEASURED) |
| 2 | Approved key agreement (SP 800-56A ECC CDH P-256/P-384) | READY (draft) | ECDH suite `vectors=2458 (cavp=100 wycheproof=2284 edge=74) mutants=9/9` (draft omega-mv#847, MEASURED). ACVP KAS-ECC: NOT_APPLICABLE, because NIST's sample file has no P-256/P-384 group |
| 3 | Approved DRBG (SP 800-90A HMAC_DRBG) | READY (draft) | `DRBG PASS vectors=810 (acvp=90, cavp=720) api=23 mutants=13/13` (draft omega-mv#846, MEASURED) |
| 4 | Approved-only mode that refuses ChaCha20 and X25519 | READY (draft) | omega-mv#847 interop rows and mutants (`TLS PASS … interop=57 … mutants=34/34`, MEASURED) |
| 5 | Pre-operational / conditional self-tests and latched error state | READY (draft) | `FIPS PASS checks=45 drbgerr=46 kat-mutants=14/14 code-mutants=16/16` (omega-mv#846, MEASURED) |
| 6 | DRBG and approved mode integrated (key generation from the DRBG, PCT failure enters error state) | READY (draft) | omega-mv#855 at 461c607f: `FIPS PASS checks=49 drbgerr=50 ecgenerr=52 kat-mutants=14/14 code-mutants=18/18`, TLS_NET net=PASS, HTTP-CHECK PASS (MEASURED, Integrated/ logs); dudect on the new path NOT_RUN |
| 7 | Software integrity test at start-up (FIPS 140-3 area 5) | GAP | not implemented. Reproducible build MEASURED (repro/run.log) |
| 8 | Show-version service (area 4) | GAP | not implemented |
| 9 | SSP zeroisation | READY, with a GAP | residue scan `RESIDUE PASS secrets=18 hs_only=12` (MEASURED); compiler does not scrub registers or spilled locals (THREAT_MODEL limit 1, GAP) |
| 10 | Entropy source assessed (SP 800-90B) | READY as an engineering result; BLOCKER for an ESV | NIST EA tool on getrandom: non-IID 7.18, restart 6.64 bits/byte (MEASURED, FipsModule entropy/run2). This grades the Linux kernel's output. An ESV certificate needs the kernel/platform vendor or a lab |
| 11 | Health tests (RCT, APT, start-up) on every entropy byte | READY (draft) | omega-mv#846 checks 10-24 plus mutants (MEASURED); cut-offs MODELED in Wolfram |
| 12 | Security policy (SP 800-140B) | READY as a draft | SECURITY-POLICY.md (self-written, no lab review) |
| 13 | Non-invasive / timing | READY, with a GAP | Welch t < 4.5 on secret code (MEASURED, one loaded host); PSTATE.DIT detected, not set (needs compiler word, GAP) |
| 14 | Vulnerability handling (life-cycle) | READY (draft) | SECURITY.md in omega-mv#844; GitHub private vulnerability reporting enabled (MEASURED via gh api) |
| 15 | Fuzzing | see fuzz/STATUS.md | — |
| 16 | Operational environment tested beyond one machine | GAP | only the DGX Spark (Ubuntu 24.04.5, arm64) was used |
| 17 | ACVP production-server validation (CAVP certificate) | BLOCKER | needs a NIST ACVP account through an accredited lab; costs in COSTS.md |
| 18 | CMVP module validation (FIPS 140-3 certificate) | BLOCKER | needs an accredited lab contract and NIST fees; costs in COSTS.md |
| 19 | Independent third-party code audit | BLOCKER | needs a paid auditor; costs in COSTS.md |
