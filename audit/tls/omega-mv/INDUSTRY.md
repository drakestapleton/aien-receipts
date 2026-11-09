# How established TLS libraries prove they were tested

Research slice for the Omega TLS 1.3 client proof pack (aien-dev/omega-mv draft PR #833, `om/tls/`).
Question from Drake: "What certificates do they post to prove it's been tested? What's the industry standard? People are not just going to trust me, bro."

All pages were fetched on 2026-10-26 unless a different date is given. "date not stated" means the page shows no date. [UNVERIFIED] marks anything I could not confirm from a source. This file does no arithmetic. Every number is quoted from its source.

## Short answer

- Only one kind of formal, government-issued certificate is common for TLS/crypto libraries: a **FIPS 140-3 certificate** from the US/Canadian Cryptographic Module Validation Program (CMVP). Each one comes with **CAVP algorithm certificates** (A-numbers). OpenSSL, BoringSSL, AWS-LC, wolfSSL and Go all hold one. rustls and s2n-tls do not hold their own; they rely on the AWS-LC certificate.
- None of the six libraries holds its own **Common Criteria** certificate. Common Criteria certifies whole products (for example an operating system), not a TLS library on its own.
- Everything else is **community evidence**, and it is not a certificate: published third-party audit PDFs, continuous fuzzing (OSS-Fuzz), shared conformance suites (BoGo, tlsfuzzer, Wycheproof, x509-limbo, NIST PKITS), formal proofs, a public security policy, CVE handling, and OpenSSF badges and scores.
- A FIPS certificate requires an accredited lab, NIST fees and a long queue. Lab prices are not published. The NIST fee alone for a new Level 1 submission is $16,000 plus a $3,000 extended fee. The NIST queue averaged 348 days over the last 30 validations (NIST, January 2026).
- A new library earns trust in the order the established libraries did: run the public test suites in CI and publish the results, get continuous fuzzing, write a security policy, and then pay for an independent audit and publish the full report. FIPS comes last, if at all.

---

## 1. Per-library table

### 1a. Formal certificates (accredited bodies)

All CMVP certificates below are FIPS 140-3, overall Level 1, software modules. Certificate pages follow the pattern `https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/<N>`. "Sunset" is the date the certificate moves to the Historical list.

| Library | CMVP certificate(s) (module, validation date, lab, sunset) | CAVP (algorithm) certs | Common Criteria | Notes |
|---|---|---|---|---|
| **OpenSSL** | [#4985](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4985) "OpenSSL FIPS Provider" (3.1.2). Validated 3/11/2025 (Acumen Security); updated 11/21/2025 (Lightship Security). Active, sunset 3/10/2030. Older cert [#4282](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4282) is Historical. | A3548 (listed on #4985; includes the TLS 1.3 KDF) | None found as a standalone library (CC portal product list, see 1c). | Announcement: [OpenSSL Corp, 2025-03-11](https://openssl-corporation.org/post/2025-03-11-fips-140-3/index.html). Security Policy: [140sp4985.pdf](https://csrc.nist.gov/CSRC/media/projects/cryptographic-module-validation-program/documents/security-policies/140sp4985.pdf). A newer OpenSSL FIPS Provider is in "Comment Resolution – CMVP" (10/21/2026) on the [MIP list](https://csrc.nist.gov/Projects/Cryptographic-Module-Validation-Program/Modules-In-Process/Modules-In-Process-List). |
| **BoringSSL** ("BoringCrypto", Google LLC) | [#4735](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4735): 7/23/2024 (Acumen), sunset 7/22/2029. [#5104](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5104): 12/9/2025 (DEKRA), sunset 12/8/2030. [#5244](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5244): 4/18/2026 (DEKRA), sunset 4/17/2031. [#5296](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5296): 6/2/2026 (DEKRA), sunset 1/26/2030; it replaced interim cert #4953 (1/27/2025, now Historical). #4407 is Historical. | A2811 (#4735), A5370 (#5104), A6134 (#5244), A4687 (#5296) | None found | Three more BoringCrypto entries are on the MIP list: CR-CMVP 10/10/2026, CR-Lab 7/6/2026, CR-Lab 6/5/2026. Other vendors certify BoringCrypto under their own names: #5274 and #5358 (Zebra Technologies, "BoringCrypto"), #4973 ("VMware's BoringCrypto Module", vendor Broadcom Inc.), #5521 ("Samsung BoringCrypto Android 14", Samsung Electronics). |
| **AWS-LC** (Amazon) | [#4631](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4631) "AWS-LC Cryptographic Module" (AWS-LC-FIPS 1.0): 10/6/2023, updated 6/27/2025, sunset 10/5/2028. [#4816](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4816) "(static)" v2.0: 10/1/2024, sunset 9/30/2029. [#5298](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5298) "AWS-LC 3 (dynamic)": 6/3/2026, sunset 6/2/2031. [#5314](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5314) "AWS-LC 3 (static)": 6/5/2026, sunset 6/4/2031. All tested by atsec. | A4509 and others (#4816); A6160 and others (#5298); A6283 and others (#5314) | None found | Index of certs: [FIPS.md](https://github.com/aws/aws-lc/blob/main/crypto/fipsmodule/FIPS.md) (date not stated). [AWS blog on AWS-LC FIPS 3.0 with ML-KEM](https://aws.amazon.com/blogs/security/aws-lc-fips-3-0-first-cryptographic-library-to-include-ml-kem-in-fips-140-3-validation) (date not stated). On the MIP list: AWS-LC 4 dyn/static, CR-Lab 10/22/2026; AWS-LC 5 dyn/static, Pending Review 10/5/2026. |
| **s2n-tls** (Amazon) | None of its own. s2n-tls uses AWS-LC as its libcrypto, so its FIPS story is the AWS-LC certificate. | via AWS-LC | None found | [s2n-tls README](https://github.com/aws/s2n-tls/blob/main/README.md) (date not stated). |
| **rustls** | None of its own. FIPS mode uses aws-lc-rs (`fips` feature, `default_fips_provider()`, `ClientConfig::fips()`); the rustls manual cites AWS-LC cert #4816. | via AWS-LC | None found | [rustls FIPS manual page](https://docs.rs/rustls/latest/rustls/manual/_06_fips/index.html) (docs 0.23.46, date not stated). Caveat: [rustls issue #3256](https://github.com/rustls/rustls/issues/3256) (opened about 2026-09) reports that `fips()` returned false in 0.23.44 once ML-DSA was enabled. A "FIPS mode" flag is only as good as the code that checks it. |
| **wolfSSL** ("wolfCrypt", wolfSSL Inc.) | [#4718](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4718): 7/11/2024 (AEGISOLVE), updated 5/16/2025 and 1/29/2026, sunset 7/10/2029. [#5041](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5041): 7/18/2025 (Acumen), sunset 7/17/2030. [#5548](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5548): 10/9/2026 (AEGISOLVE), sunset 10/8/2031. Older FIPS 140-2 cert #3389. | A4308 (#4718), A2461 (#5041), A7470 (#5548) | None found as a standalone library. wolfSSL writes about NIAP/CNSA 2.0 ([blog](https://www.wolfssl.com/cnsa-2-0-update-part-2-niap/), about 2024-05) but no wolfSSL CC certificate appears on the CC portal list. | [wolfSSL FIPS page](https://www.wolfssl.com/license/fips) (date not stated). A wolfCrypt entry is "Pending Resubmission" (4/16/2026) on the MIP list. |
| **Go crypto/tls** ("Go Cryptographic Module", vendor Geomys LLC) | [#5247](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5247): first validated 4/27/2026, updated 7/7/2026, both by Lightship Security. Sunset 4/26/2031. | A6650 and A8668 (on #5247). The [Go docs](https://go.dev/doc/security/fips140) say module v1.26.0 is covered by CAVP A8028 and was "Pending Review" in the MIP list as of 2026-04-28. | None found | [Go blog "The FIPS 140-3 Go Cryptographic Module", 2025-07-15](https://go.dev/blog/fips140): v1.0.0 ships in Go 1.24 (February 2025), reached the MIP list in May 2025, and covers 23 tested Operating Environments; Google and others pay Geomys to add more. Users turn it on with `GOFIPS140=v1.0.0` and the `fips140=on` GODEBUG. [Geomys FIPS page](https://geomys.org/fips140) (date not stated). |

### 1b. Community and self-published evidence

| Library | Third-party audit (public report) | Formal verification | Continuous fuzzing | Conformance suites | Releases / supply chain | Security policy, CVE issuer (CNA), bounty | OpenSSF badge / Scorecard |
|---|---|---|---|---|---|---|---|
| **OpenSSL** | Trail of Bits for OSTIF, autumn 2023, 9 engineer-weeks, 23 findings with security impact (4 medium, 6 low, 13 informational), plus 4 new fuzzers ([OSTIF post, 2024-06-03](https://ostif.org/openssl-audit-complete/); [OpenSSL post, 2024-05-02](https://openssl-library.org/post/2024-05-02-ostif); [full PDF](https://ostif.org/wp-content/uploads/2024/05/OSTIF-OpenSSL-Comprehensive-Report-1.pdf); [ToB PDF](https://github.com/trailofbits/publications/blob/master/reviews/2023-09-openssl-securityreview.pdf)). Earlier: Quarkslab for OSTIF on 1.1.1 ([2019-01-21](https://ostif.org/the-ostif-and-quarkslab-audit-of-openssl-is-complete); [PDF](https://ostif.org/wp-content/uploads/2019/01/18-04-720-REP_v1.2.pdf)). Alpha-Omega paid for OpenSSL audits ([2024 annual report](https://alpha-omega.dev/wp-content/uploads/sites/22/2025/01/Alpha-Omega-Annual-Report-2024_012925.pdf)). | None found in the main tree. | Yes: OSS-Fuzz project [`openssl`](https://github.com/google/oss-fuzz/tree/master/projects/openssl); in-tree [`fuzz/`](https://github.com/openssl/openssl/blob/master/fuzz/README.md) (libFuzzer/AFL); also covered by OSS-Fuzz `cryptofuzz`. | tlsfuzzer run from the test suite (`test/recipes/95-test_external_tlsfuzzer.t`); some Wycheproof vectors (`test/mldsa_wycheproof_parse.py`); listed in [x509-limbo results](https://x509-limbo.com/testcases/rfc5280/). | Tarballs signed with OpenPGP ([downloads page](https://openssl-library.org/source); [Release Artifacts Signing Policy](https://openssl-library.org/policies/general/artifact-signing-policy), date not stated). SLSA provenance or Sigstore: not found. | CNA: OpenSSL is its own CNA (e.g. [CVE-2025-9232](https://www.cve.org/CVERecord?id=CVE-2025-9232), published 2025-09-30, assigner "openssl"). OpenSSL runs no bounty of its own and points to HackerOne's Internet Bug Bounty ([community page](https://mirror.openssl-library.org/community), date not stated). | Best Practices "passing", [project 54](https://www.bestpractices.dev/projects/54). Scorecard 6.3/10 (scored 2026-10-19, [API](https://api.securityscorecards.dev/projects/github.com/openssl/openssl)). |
| **BoringSSL** | No public third-party audit report found. | fiat-crypto (machine-checked) field arithmetic in [`third_party/fiat`](https://boringssl.googlesource.com/boringssl/+/refs/heads/master/third_party/fiat) ([fiat-crypto](https://github.com/mit-plv/fiat-crypto)). | OSS-Fuzz [`boringssl`](https://github.com/google/oss-fuzz/tree/master/projects/boringssl); cryptofuzz. | Owns **BoGo**, the TLS test runner other libraries reuse ([`ssl/test`](https://github.com/google/boringssl/tree/master/ssl/test)). Wycheproof converter `util/convert_wycheproof`. NIST PKITS data in [`pki/testdata/nist-pkits`](https://boringssl.googlesource.com/boringssl.git/+/71bcfad967d575343145f95158b6e9926ce94337/pki/testdata/nist-pkits/README.md). In x509-limbo results ("boringssl-legacy"). | Not a released product: no tagged releases or provenance. | Google LLC is a CNA [UNVERIFIED for BoringSSL CVEs specifically]. BoringSSL is **not** on Google's OSS VRP tier list ([tier list](https://github.com/google/bughunters/blob/main/oss-repository-tier/README.md)). Bounty coverage through Chrome VRP: [UNVERIFIED]. | No Best Practices badge found (searched 2026-10-26). Scorecard 6.4/10 for the GitHub mirror `google/boringssl` (2026-10-26). |
| **AWS-LC** | No public third-party audit report found. | SAW, NSym and s2n-bignum (HOL Light) proofs run in CI on every change ([aws-lc README](https://github.com/aws/aws-lc); [aws-lc-verification](https://github.com/awslabs/aws-lc-verification); [s2n-bignum](https://github.com/awslabs/s2n-bignum)); script `tests/ci/run_formal_verification.sh`. AWS talk: [NIST, "Formal verification of cryptographic software at AWS"](https://www.nist.gov/document/formal-verification-cryptographic-software-aws-current-practices-and-future-trends) (date not stated). | In-house CI fuzzing (`tests/ci/run_fuzz_tests.sh`, `run_cryptofuzz.sh`, `FUZZING.md`). No OSS-Fuzz project directory for aws-lc. | Inherits BoGo from BoringSSL; Wycheproof vectors; in x509-limbo results (aws-lc-5.12.0). | Not checked in detail. | CNA: Amazon ("AMZN"). Reports go to the AWS Vulnerability Disclosure Program on HackerOne ([AWS VDP](https://hackerone.com/aws_vdp)), which is a disclosure program, not a bounty. | No badge found. Scorecard: no public score (API returned 404 on 2026-10-26). |
| **s2n-tls** | The 2015 launch post says s2n had three external security evaluations, but no report is public ([AWS blog, 2015-06-30](https://aws.amazon.com/blogs/security/introducing-s2n-a-new-open-source-tls-implementation); see also [Albrecht & Paterson, ePrint 2015/1129](https://eprint.iacr.org/2015/1129)). | Strongest public example: SAW proofs of HMAC, DRBG and the TLS handshake state machine, run continuously ([CAV 2018 paper](https://link.springer.com/chapter/10.1007/978-3-319-96142-2_26)); CBMC memory-safety proofs checked in CI ([tests/cbmc](https://github.com/aws/s2n-tls/blob/main/tests/cbmc/README.md)); ctverif and sidetrail (timing) in `tests/`. | In-repo fuzzers (`tests/fuzz`); no OSS-Fuzz project directory. | Not found (tlsfuzzer not found by code search). | Not checked in detail. | [SECURITY.md](https://github.com/aws/s2n-tls/blob/main/SECURITY.md): AWS VDP on HackerOne. CNA Amazon ([CVE-2026-16317](https://www.cve.org/CVERecord?id=CVE-2026-16317), published 2026-07-21; [AWS bulletin 2026-062](https://aws.amazon.com/security/security-bulletins/2026-062-aws/)). | No badge found; Scorecard 404 (2026-10-26). |
| **rustls** | Cure53, May–June 2020, covering rustls, ring and webpki, paid for by CNCF ([report in repo](https://github.com/rustls/rustls/blob/main/audit/TLS-01-report.pdf); [Cure53 copy](https://cure53.de/pentest-report_rustls.pdf); [author's blog, 2020-06-14](https://jbp.io/2020/06/14/rustls-audit.html)). No later audit found (the repo's `audit/` folder holds only that report). | None in rustls itself. | OSS-Fuzz [`rustls`](https://github.com/google/oss-fuzz/tree/master/projects/rustls); [SECURITY.md](https://github.com/rustls/rustls/blob/main/SECURITY.md) says it fuzzes with a mock crypto provider. | Runs **BoGo** ([`bogo/README.md`](https://github.com/rustls/rustls/blob/main/bogo/README.md)); rustls-webpki in x509-limbo results. | crates.io releases; provenance not checked. | [SECURITY.md](https://github.com/rustls/rustls/blob/main/SECURITY.md): private GitHub advisories, [RustSec](https://rustsec.org/packages/rustls.html), fixes backported to releases under 2 years old. CVEs come from GitHub's CNA (e.g. [CVE-2024-32650](https://www.cve.org/CVERecord?id=CVE-2024-32650), assigner GitHub_M, 2024-04-19). No bounty found. | Best Practices "passing", [project 9034](https://www.bestpractices.dev/projects/9034). Scorecard 404 (2026-10-26). |
| **wolfSSL** | No published third-party code audit found. | None found. | OSS-Fuzz [`wolfssl`](https://github.com/google/oss-fuzz/tree/master/projects/wolfssl) and cryptofuzz; vendor overview of its own testing: [wolfSSL testing overview](https://www.wolfssl.com/overview-of-testing-in-wolfssl-2) (about 2024-01). | Wycheproof mentioned in the README; tlsfuzzer not found by code search. | Not checked in detail. | Own CNA since at least 2026 ("wolfSSL Inc.", e.g. [CVE-2026-5194](https://www.cve.org/CVERecord?id=CVE-2026-5194), 2026-04-09); older CVEs were issued by MITRE (e.g. [CVE-2019-19962](https://www.cve.org/CVERecord?id=CVE-2019-19962)). [Vulnerability page](https://www.wolfssl.com/docs/security-vulnerabilities) (date not stated); [why wolfSSL reports many CVEs](https://www.wolfssl.com/why-is-wolfssl-reporting-so-many-cves) (2026-06). No bounty found. | No badge found. Scorecard 5.9/10 (2026-10-19). |
| **Go crypto/tls** | Trail of Bits for Google, March 2025, 12 weeks, 6 findings (1 low in the legacy Go+BoringCrypto path, 5 informational). The whole FIPS module was in scope ([Go blog, 2025-05-19](https://go.dev/blog/tob-crypto-audit); [PDF](https://github.com/trailofbits/publications/blob/master/reviews/2025-03-google-gocryptographiclibraries-securityreview.pdf); [ToB summary](https://trailofbits.com/library/go-crypto-libraries)). | P-256/P-384/P-521 field arithmetic generated by fiat-crypto "from a formally verified model" ([nistec/fiat README](https://github.com/golang/go/blob/master/src/crypto/internal/fips140/nistec/fiat/README)). | OSS-Fuzz [`golang`](https://github.com/google/oss-fuzz/tree/master/projects/golang); cryptofuzz. | Runs **BoGo** (`src/crypto/tls/bogo_shim_test.go`, `bogo_config.json`); its ACVP test wrapper follows BoringSSL's ACVP protocol (`src/crypto/internal/fips140test/acvp_test.go`); x509-limbo results ("gocryptox509"). Wycheproof: [UNVERIFIED]. | Reproducible toolchain since Go 1.21: anyone can rebuild and verify the binaries ([Go blog, 2023-08-28](https://go.dev/blog/rebuild); [report](https://go.dev/rebuild)). | CNA: "Go" (e.g. [CVE-2025-22871](https://www.cve.org/CVERecord?id=CVE-2025-22871), 2025-04-08). golang/go is a "flagship" (OT0) repo in Google's OSS VRP ([tier list](https://github.com/google/bughunters/blob/main/oss-repository-tier/README.md)), but Google **stopped paying for product bugs in OSS VRP from 2026-10-01** ([The Hacker News, 2026-10-06](https://thehackernews.com/2026/10/google-pauses-oss-product-bug-bounty.html); [rules page](https://bughunters.google.com/about/rules/open-source/google-open-source-software-vulnerability-reward-program-rules)). | No badge found. Scorecard 6.2/10 (2026-10-26). |

### 1c. How the "none found" entries were checked

- **Common Criteria.** I downloaded the CC portal product list ([certified_products.csv](https://www.commoncriteriaportal.org/products/certified_products.csv), 2,102 rows, fetched 2026-10-26) and searched product names and manufacturers for wolf, openssl, boring, aws-lc, s2n and rustls. There were no matches. The only "cryptographic library" rows are smart-card chip libraries (STMicroelectronics NesLib, NXP, Infineon, Google H1D3). CC certifies a product such as an operating system ([example: Red Hat Enterprise Linux 8.2 certificate](https://www.commoncriteriaportal.org/nfs/ccpfiles/files/epfiles/st_vid11202-ci.pdf)), and the TLS code inside it is evaluated through the [TLS Functional Package](https://github.com/commoncriteria/tls) attached to a protection profile such as the [NIAP Application Software PP](https://www.niap-ccevs.org/static_html/protection-profile/462/462/index.html). For a library vendor, CC is not a normal goal.
- **OpenSSF Best Practices badge.** Searched [bestpractices.dev](https://www.bestpractices.dev/en/projects.json) by name and repository URL on 2026-10-26. Only OpenSSL (54, passing) and rustls (9034, passing) appear.
- **OpenSSF Scorecard.** Queried [api.securityscorecards.dev](https://api.securityscorecards.dev) on 2026-10-26. aws-lc, s2n-tls and rustls returned 404, meaning they are not in the public weekly scan. Any repo can run Scorecard itself.

---

## 2. FORMAL CERTIFICATES (issued by accredited bodies)

These are the only items a buyer can look up in a government registry.

| Certificate | Who issues it | What it says | What it does **not** say | Registry |
|---|---|---|---|---|
| **CAVP algorithm certificate** (A-number) | NIST Cryptographic Algorithm Validation Program, after a run on NIST's ACVP server that an accredited lab submits | A specific implementation of specific algorithms (e.g. AES-GCM, ECDSA P-256, the TLS 1.3 KDF) gave correct answers to NIST's test vectors | Nothing about memory safety, protocol logic, side channels, or certificate path validation | [CAVP validation search](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program) |
| **CMVP FIPS 140-3 certificate** | NIST and the Canadian Centre for Cyber Security, jointly, after testing by an NVLAP-accredited lab ([CMVP overview](https://csrc.nist.gov/Projects/Cryptographic-Module-Validation-Program)) | The named module, at the named version and on the listed operating environments, meets FIPS 140-3 (ISO/IEC 19790 plus 24759 test requirements) at the stated level, and its Security Policy describes the approved mode | That the TLS protocol code is correct (CMVP tests the crypto module boundary; only the TLS KDF is algorithm-tested), that there are no bugs, or that any other version is covered | [Validated Modules search](https://csrc.nist.gov/Projects/Cryptographic-Module-Validation-Program/Validated-Modules) |
| **Common Criteria certificate** | A national scheme under the CCRA (in the US, NIAP) after evaluation by an approved lab against a protection profile | A whole product meets a protection profile | Not normally issued to a TLS library on its own (see 1c) | [CC portal](https://www.commoncriteriaportal.org/products/), [NIAP PCL](https://www.niap-ccevs.org/products) |

"Validated" versus "compliant":
- **FIPS 140-3 validated** means a current certificate number on the CMVP Validated Modules list for that exact module version and operating environment ([Validated Modules page](https://csrc.nist.gov/Projects/Cryptographic-Module-Validation-Program/Validated-Modules): modules "tested and validated under the CMVP").
- **On the Modules In Process (MIP) list** means only that the CMVP is "actively working on" it ([MIP page](https://csrc.nist.gov/Projects/Cryptographic-Module-Validation-Program/Modules-In-Process/Modules-In-Process-List)). It is not validated. Go's own docs keep the two apart, with separate `inprocess` and `certified` settings ([Go FIPS docs](https://go.dev/doc/security/fips140)).
- **"FIPS compliant", "FIPS-ready" or "uses FIPS-approved algorithms"** are marketing phrases. CMVP defines no status with those names (my reading of the CMVP pages above; I found no CMVP definition of "compliant"). The Go docs warn that using "a FIPS 140-3 compliant and validated cryptographic module may not—on its own—satisfy all relevant regulatory requirements".
- **Historical** certificates (past sunset, or moved for cause) are no longer valid for new US federal procurement. Older certs such as OpenSSL #4282 and BoringSSL #4407 are in this state.

---

## 3. The FIPS 140-3 process in detail

### Who is involved
- **Validation authority:** CMVP, run jointly by NIST and the Canadian Centre for Cyber Security (CCCS). The January 2026 NIST status talk says every report gets two reviews, one from NIST and one from CCCS ([Hawes, "CMVP Status and Future Plans", 2026-01-21](https://csrc.nist.gov/csrc/media/presentations/2026/cmvp-status-and-future-plans/images-media/1.4-cmvp_status-hawes.pdf)).
- **Testing lab (CSTL):** must be accredited by NVLAP under the Cryptographic and Security Testing program ([NVLAP CST LAP](https://www.nist.gov/nvlap/cryptographic-and-security-testing-lap); [CMVP lab and fee page](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/cst-lab-accreditation-and-fees)). Labs seen on the certificates above: Acumen Security, Lightship Security, DEKRA, AEGISOLVE and atsec ([atsec testing services](https://atsec.com/services/cryptographic-testing), NVLAP lab code 200658-0). NVLAP Lab Bulletin [LB-163-2026](https://www.nist.gov/nvlap/nvlap-lab-bulletin-lb-163-2026) (2026-03-09) announces a pause on new CST lab applications (I only read the summary; check the full text before quoting).
- **Vendor:** the legal entity named on the certificate. For Go this is a company, Geomys LLC, not "the Go project". For OpenSSL it is OpenSSL Corporation. An open-source project needs some legal entity to sign the lab contract and own the certificate.
- **Consultants (optional):** firms such as [SafeLogic](https://www.safelogic.com/fips-140-3) and [Corsec](https://www.corsec.com/fips-140-3) write the documents and run the project for the vendor.

### Steps
The order follows the [Intel FIPS 140-3 primer (January 2026)](https://cdrdv2-public.intel.com/850484/fips-140-3-intel-s-approach-to-cryptographic-module-validation-jan26.pdf) ("contracting an approved CST lab, providing design evidence, Cryptographic Algorithm Validation testing, Entropy Source Validation, source code review…") and the [FIPS 140-3 CMVP Management Manual](https://csrc.nist.gov/CSRC/media/Projects/cryptographic-module-validation-program/documents/fips%20140-3/FIPS-140-3-CMVP%20Management%20Manual.pdf) (version dated 04-09-2026).
1. **Define the module boundary**: the exact code and binary, the version, and the operating environments to be tested.
2. **Contract an accredited lab.**
3. **Algorithm testing (CAVP)** through NIST's ACVP server. The lab submits; the result is an A-number per algorithm set.
4. **Entropy source validation** (SP 800-90B, the ESV program) for the randomness source, or use of an already validated one.
5. **Write the documents**: the Security Policy (published with the certificate) and the design and test evidence for each ISO/IEC 24759 requirement. The NIST talk counts about 650 test/evidence pairs.
6. **Lab testing**: the lab checks every requirement, including self-tests, approved-mode indicator, zeroisation and the integrity check.
7. **Lab submits the report.** The module appears on the MIP list as "Pending Review".
8. **Cost-recovery fee**: "For a module to transition from Pending Review to Review, the lab must first pay the NIST Cost Recovery fee" ([MIP page](https://csrc.nist.gov/Projects/Cryptographic-Module-Validation-Program/Modules-In-Process/Modules-In-Process-List)).
9. **Review**, then rounds of **Comment Resolution** (Lab, then CMVP) until both NIST and CCCS sign off. The Management Manual says a dropped module must restart and pay a new fee.
10. **Certificate issued** and the module listed as Active with a 5-year sunset date. Later changes need update submissions (UPDT, OEUP and others; fees below).

Queue snapshot from the MIP list ("Last Updated 10/26/2026"): 196 entries (187 displayed). Among the parsed rows: Review 58, Comment Resolution-Lab 50, Comment Resolution-CMVP 36, Pending Review 31, Pending Resubmission 12.

### Time (quoted, not computed)
- NIST, 2026-01-21 ([Hawes slides](https://csrc.nist.gov/csrc/media/presentations/2026/cmvp-status-and-future-plans/images-media/1.4-cmvp_status-hawes.pdf)): the queue had been "greater than 2 years, after lab testing". As of 2026-01-01 the queue backlog was 180 days, and the average of the last 30 validations was 348 days. NIST's goal was a near-zero backlog by 2026-07-01.
- KeyPair, 2024-02 ([article](https://keypair.us/2024/02/fips-140-3-validation-times)): an average of 579 days from report receipt to certificate (secondary source).
- Real examples:
  - Go module v1.0.0 shipped in February 2025, was on the MIP list in May 2025 ([Go blog](https://go.dev/blog/fips140)), and was certified 4/27/2026 (#5247).
  - OpenSSL announced its 3.1.2 provider certificate on 2025-03-11 ([OpenSSL Corp](https://openssl-corporation.org/post/2025-03-11-fips-140-3/index.html)).
- Lab time comes on top and is not published by NIST. [UNVERIFIED] for any lab-phase duration.

### Money
NIST cost-recovery fees, charged to the lab and passed on to the vendor, effective 2026-01-01 ([NIST cost recovery fees](https://csrc.nist.gov/projects/cryptographic-module-validation-program/nist-cost-recovery-fees), updated 2026-01-05):

| Submission type | Cost Recovery (CR) | Extended CR (ECR) |
|---|---|---|
| FIPS 140-3 full submission, Level 1 | $16,000 | $3,000 |
| Full submission, Level 2 | $17,000 | $4,000 |
| Full submission, Level 3 | $17,500 | $4,000 |
| Full submission, Level 4 | $19,000 | $4,000 |
| UPDT (update) | $5,500 | $1,500 |
| ALG, OEUP, PTSC, CVE, TRNS, PHYS, NSRL, RBND | $2,500 | $1,000 |
| VUP, VAOE | $0 | $1,000 |
| Entropy source (ESV), full | $5,500 | $1,500 |
| ESV update (add OE) | $2,500 | $1,000 |

- **Lab and consultant fees are not published.** NIST says only that "CST labs and NIST each charge fees for their respective parts" ([lab and fee page](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/cst-lab-accreditation-and-fees)). SafeLogic says cost depends on module type and other factors ([SafeLogic](https://www.safelogic.com/fips-140-3)). Any total-cost figure is [UNVERIFIED]. I did not quote aggregator sites or AI-generated estimates.
- Ongoing costs: Go's blog says Google and others pay Geomys to add operating environments to the certificate ([Go blog](https://go.dev/blog/fips140)). Each new OE or version is a paid update.

### ACVP access (algorithm testing)
- **Demo server:** a free sandbox that anyone can request access to, for practice. Results there are not certificates.
- **Production server:** "only for accredited CST labs", and a lab must first show proficiency on Demo ([How to access ACVTS](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/how-to-access-acvts)). Access needs a TLS client certificate plus a TOTP one-time password ([ACVP protocol docs](https://pages.nist.gov/ACVP)).
- **Tools:** [NIST ACVP-Server](https://github.com/usnistgov/ACVP-Server), Cisco [libacvp](https://github.com/cisco/libacvp), and BoringSSL's [acvptool / ACVP.md](https://boringssl.googlesource.com/boringssl/+show/main/util/fipstools/acvp/ACVP.md). Go reuses BoringSSL's module-wrapper protocol for its own tests (`src/crypto/internal/fips140test/acvp_test.go`).
- What this means for Omega: I can run NIST's published ACVP test vectors through Omega's AES-GCM, ChaCha20-Poly1305 (not a FIPS algorithm), X25519 (not FIPS-approved for key agreement; [UNVERIFIED] whether SP 800-186 changes this for TLS use), ECDSA and RSA code, and publish the results. I cannot call that a CAVP certificate. Only a lab can submit to production.

---

## 4. The independent audit path

### What established libraries actually published

| Library | Firm | When | Effort | Funded by | Public report |
|---|---|---|---|---|---|
| OpenSSL 3 | Trail of Bits | autumn 2023 (published 2024-05/06) | 9 engineer-weeks | OSTIF, with Alpha-Omega funding | [PDF](https://ostif.org/wp-content/uploads/2024/05/OSTIF-OpenSSL-Comprehensive-Report-1.pdf) |
| OpenSSL 1.1.1 | Quarkslab | published 2019-01-21 | not stated | OSTIF | [PDF](https://ostif.org/wp-content/uploads/2019/01/18-04-720-REP_v1.2.pdf) |
| Go crypto (FIPS module) | Trail of Bits | 2025-03 (blog 2025-05-19) | 12 weeks | Google | [PDF](https://github.com/trailofbits/publications/blob/master/reviews/2025-03-google-gocryptographiclibraries-securityreview.pdf) |
| rustls + ring + webpki | Cure53 | 2020-05/06 | not stated here | CNCF ([author's blog, 2020-06-14](https://jbp.io/2020/06/14/rustls-audit.html)) | [PDF](https://github.com/rustls/rustls/blob/main/audit/TLS-01-report.pdf) |
| s2n | three external evaluations (names not public) | 2015 | not stated | Amazon | not public |

### Firms that publish crypto/TLS reviews
- Trail of Bits: [publications repo](https://github.com/trailofbits/publications), including OpenSSL and Go.
- Cure53: [publications](https://github.com/cure53/Publications), including rustls.
- Quarkslab: OpenSSL 1.1.1.
- NCC Group, X41 D-Sec and Include Security also do OSTIF work ([OSTIF engagements list](https://github.com/ostif-org/OSTIF/blob/main/Completed-Engagements.md)). I did not find TLS-library reports from them among the six libraries here.

### Who pays for audits of open-source projects
- **OSTIF** (Open Source Technology Improvement Fund) scopes and manages audits ([get an audit](https://ostif.org/get-an-audit/)). Its stated range: "initial audits could range anywhere from $30k to $200k", and it helps projects find sponsors. 2025 summary: about $2,800,000 in audit funding, of which 87% went to auditing and 13% to OSTIF overhead ([OSTIF/Sovereign Tech Agency 2024-2025 report](https://ostif.org/wp-content/uploads/2026/02/OSTIF-STA-Report.pdf)). The STA-funded part alone was $642,000 ([same report](https://ostif.org/wp-content/uploads/2026/02/OSTIF-STA-Report.pdf); [2025 annual report](https://ostif.org/wp-content/uploads/2026/01/OSTIF-2025-Annual-Report.pdf)).
- **Alpha-Omega** (OpenSSF): paid for OpenSSL audits ([2024 report](https://alpha-omega.dev/wp-content/uploads/sites/22/2025/01/Alpha-Omega-Annual-Report-2024_012925.pdf)), and gave $530,000 to ISRG's Prossimo for rustls and Rust-for-Linux work ([2023-09-18](https://alpha-omega.dev/blog/advancing-rustls-and-rust-for-linux-with-openssf-support/)). Its 2024 total is stated as "nearly $4.5 million" on [alpha-omega.dev](https://alpha-omega.dev/blog/alpha-omega-2024-annual-report) and "nearly $6 million" on [openssf.org](https://openssf.org/blog/2025/01/29/alpha-omega-2024-annual-report/); the two pages disagree.
- **Sovereign Tech Agency** (Germany, formerly Sovereign Tech Fund): €1,436,729 to Prossimo for 2023–2024, which includes rustls work ([STA Prossimo page](https://www.sovereigntechfund.de/tech/prossimo)). STA also funds audits through OSTIF (report above).
- Price per engineer-week: neither Trail of Bits nor Cure53 publishes rates. Third-party rate-card sites exist but are not primary sources. [UNVERIFIED].

### OpenSSF Best Practices badge and Scorecard (free, self-run)
- **Best Practices badge** ([bestpractices.dev](https://www.bestpractices.dev)): a self-attested questionnaire with levels passing, silver and gold. It covers a security policy, tests, static analysis, signed releases and so on. OpenSSL and rustls hold "passing".
- **Scorecard** ([scorecard.dev](https://scorecard.dev); [checks list](https://github.com/ossf/scorecard/blob/main/docs/checks.md)): an automated 0–10 score from repo settings, covering Fuzzing, SAST, Signed-Releases, Branch-Protection, Code-Review, Security-Policy, Token-Permissions and more. Scores on 2026-10-26: OpenSSL 6.3, BoringSSL 6.4, Go 6.2, wolfSSL 5.9. Even mature crypto libraries score in the 6s. This is a hygiene signal, not a security verdict.

---

## 5. Conformance and test suites a TLS 1.3 client can run (all free)

| Suite | What it tests | Who uses it | Link |
|---|---|---|---|
| **BoGo** | Thousands of TLS protocol tests driven through a small "shim" program; the de facto TLS conformance runner | BoringSSL (origin), AWS-LC, rustls, Go | [boringssl/ssl/test](https://github.com/google/boringssl/tree/master/ssl/test) |
| **tlsfuzzer** | TLS protocol behaviour, malformed messages, and timing side channels (Bleichenbacher, Lucky13 scripts) | OpenSSL (in its test recipes), GnuTLS | [tlsfuzzer](https://github.com/tlsfuzzer/tlsfuzzer); [docs](https://tlsfuzzer.readthedocs.io/en/latest) |
| **TLS-Anvil** | About 408 RFC-compliance tests for TLS 1.2/1.3 clients and servers, using combinatorial testing | Research and evaluation (USENIX Security 2022) | [TLS-Anvil](https://github.com/tls-attacker/TLS-Anvil); [paper](https://www.usenix.org/system/files/usenixsecurity22-maehren.pdf) |
| **Wycheproof** | Known-attack test vectors for AES-GCM, ChaCha20-Poly1305, ECDSA, ECDH/X25519, RSA and more; now maintained by C2SP | BoringSSL, AWS-LC, OpenSSL (partial), pyca/cryptography | [C2SP/wycheproof](https://github.com/C2SP/wycheproof) |
| **x509-limbo** | Certificate path-validation test cases with a public results table across libraries | Results published for OpenSSL, BoringSSL, AWS-LC, rustls-webpki, Go, GnuTLS, pyca | [x509-limbo.com](https://x509-limbo.com/) (date not stated) |
| **NIST PKITS** | The classic X.509 path-validation suite (2004, v1.0.0) | BoringSSL vendors its data | [PKITS.pdf](https://csrc.nist.gov/CSRC/media/Projects/PKI-Testing/documents/PKITS.pdf); [PKI testing](https://csrc.nist.gov/Projects/pki-testing) |
| **Frankencerts** | Differential testing with randomly spliced certificates; found 208 discrepancies across OpenSSL, NSS, CyaSSL (now wolfSSL), GnuTLS, PolarSSL and others | A 2014 research method, not a maintained CI suite | [IEEE S&P 2014 paper](https://www.cs.utexas.edu/~shmat/shmat_oak14.pdf) |
| **ACVP Demo vectors** | NIST algorithm known-answer tests | Every FIPS vendor | [ACVTS access](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/how-to-access-acvts) |
| **OSS-Fuzz** | Free continuous fuzzing for accepted open-source projects | OpenSSL, BoringSSL, rustls, wolfSSL, Go | [oss-fuzz](https://github.com/google/oss-fuzz) |

---

## 6. Minimum credible package for a new TLS library

Ranked by how much trust each step buys per unit of cost. "Did the same" lists established libraries that publish the same kind of evidence.

### Free (needs work, no money)
1. **Run the shared conformance suites in CI and publish pass/fail tables with the exact commit.**
   - BoGo for the protocol (rustls, Go and AWS-LC did the same).
   - Wycheproof for the AEAD, ECDSA, X25519 and RSA primitives (BoringSSL, AWS-LC).
   - x509-limbo plus NIST PKITS for chain validation (rustls-webpki, Go, OpenSSL and BoringSSL appear in limbo results; BoringSSL ships PKITS).
   - tlsfuzzer or TLS-Anvil for malformed-input and RFC 8446 behaviour (OpenSSL runs tlsfuzzer).
   - List every expected failure and skip by name. A test that did not run is NOT_RUN, never PASS.
2. **Run NIST ACVP Demo vectors** for every FIPS algorithm the library implements, and publish the results with the words "not a CAVP certificate" (BoringSSL and Go built ACVP wrappers before certifying).
3. **Continuous fuzzing.** Apply to OSS-Fuzz, or run cryptofuzz-style differential fuzzing against BoringSSL/OpenSSL in CI (OpenSSL, BoringSSL, rustls, wolfSSL and Go are all in OSS-Fuzz; AWS-LC fuzzes in its own CI).
4. **Interop matrix**: handshake against real servers and libraries (OpenSSL `s_server`, BoringSSL, Go), and record which cipher suites and groups were negotiated. This is what the BoGo shim approach automates.
5. **SECURITY.md** with a private reporting channel, a supported-versions window and a threat model (rustls does all three). Publish advisories through GitHub Security Advisories, which can issue CVEs through GitHub's CNA, as rustls's CVE-2024-32650 shows.
6. **OpenSSF Best Practices badge** ("passing", self-attested) and **Scorecard** in CI (OpenSSL and rustls hold the badge; OpenSSL, BoringSSL, Go and wolfSSL have public scores).
7. **Signed, reproducible releases**: signed tags or tarballs (OpenSSL OpenPGP policy) and a reproducible build with a published rebuild check (Go since 1.21).
8. **Constant-time evidence**: a timing test such as tlsfuzzer's timing scripts or a dudect-style harness, published with its method (s2n's ctverif and sidetrail are the strong form).

### Paid (needs money or a legal entity)
9. **One independent code audit with the full report published**, findings and fixes included. This is the biggest single trust step and every comparable library did it: OpenSSL (Trail of Bits, Quarkslab), Go (Trail of Bits), rustls (Cure53). OSTIF's stated range for initial audits is $30k–$200k ([OSTIF](https://ostif.org/get-an-audit/)). Funding routes: OSTIF, Alpha-Omega, Sovereign Tech Agency, or a corporate sponsor (Google paid for Go's; CNCF paid for rustls's).
10. **Machine-checked proofs for the riskiest parts** (field arithmetic, AEAD, the state machine). The cheap route is generated code from fiat-crypto, as BoringSSL and Go did. The expensive route is SAW/CBMC/HOL Light proofs run in CI, as AWS-LC and s2n do.
11. **FIPS 140-3 validation**, only if US-federal or regulated customers are a real target.
    - Requirements: a legal vendor, an accredited lab, NIST fees ($16,000 + $3,000 at Level 1, January 2026), unpublished lab fees, and a NIST queue that averaged 348 days in January 2026.
    - Done by OpenSSL, BoringSSL, AWS-LC, wolfSSL and Go. rustls and s2n chose to reuse AWS-LC's certificate instead, which is the realistic path for a new library: put a validated module underneath for FIPS customers.
    - Note for Omega's current suite set: ChaCha20-Poly1305 is not a FIPS-approved cipher. X25519's FIPS status for TLS key agreement is [UNVERIFIED] here and needs checking against SP 800-56A/SP 800-186 before any FIPS claim.
12. **Bug bounty**: optional and late. OpenSSL relies on HackerOne's Internet Bug Bounty, Amazon runs a disclosure program (not a bounty), and Google paused OSS VRP product-bug payouts on 2026-10-01 after a flood of invalid automated reports. A bounty is not a standard expectation for a TLS library.
13. **Common Criteria**: not a library-level norm (none of the six holds one). Skip unless a product built on Omega TLS targets a NIAP protection profile.

### What not to claim
- Do not say "FIPS compliant", "FIPS ready" or "certified" without a CMVP certificate number. Only "validated, cert #N" carries meaning.
- Do not call ACVP Demo results, fuzzing hours or a Scorecard number a certification.
- Do not count a skipped or NOT_RUN suite in a pass rate. The established libraries publish the suite and the exact commit so anyone can rerun them.
