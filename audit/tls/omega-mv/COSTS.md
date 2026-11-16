# Steps that need my money or signature

Only published figures are quoted, each with its source. Nothing here has been started, signed or paid.
Background and more sources: INDUSTRY.md.

| step | what it buys | who must act | published cost | source |
|---|---|---|---|---|
| Independent code audit, full report published | the strongest single trust step; OpenSSL, Go and rustls all have one | owner signs the engagement, or applies for funding | OSTIF: "initial audits could range anywhere from $30k to $200k"; engineer-week rates are not published | https://ostif.org/get-an-audit/ |
| Funding for that audit | someone else pays | owner applies | OSTIF finds sponsors; the Sovereign Tech Agency and Alpha-Omega have paid for audits before | https://ostif.org/wp-content/uploads/2026/02/OSTIF-STA-Report.pdf |
| CAVP algorithm certificates (ACVP production server) | official A-numbers for each algorithm | an accredited lab submits on behalf of a legal vendor | the lab fee is not published | https://csrc.nist.gov/Projects/cryptographic-module-validation-program/cst-lab-accreditation-and-fees |
| FIPS 140-3 certificate (CMVP), Level 1 | the government certificate that OpenSSL, BoringSSL, AWS-LC, wolfSSL and Go hold | owner (legal vendor) contracts an NVLAP-accredited lab and signs the submission | NIST cost recovery: $16,000 plus a $3,000 extended fee. The lab fee is not published. The NIST queue averaged 348 days (January 2026) | https://csrc.nist.gov/projects/cryptographic-module-validation-program/nist-cost-recovery-fees |
| Entropy certificate (ESV) | certifies the random source | the kernel or platform vendor, or a lab | not published | https://csrc.nist.gov/projects/cryptographic-module-validation-program/entropy-validations |
| OSS-Fuzz listing | Google runs my fuzzers for free | owner's consent to submit; a maintainer email must receive the bug reports | no fee | https://google.github.io/oss-fuzz/getting-started/accepting-new-projects/ |
| Security timelines in SECURITY.md (3 / 10 working days, 90-day disclosure) | public promise to reporters | owner approves the wording | none | omega-mv#844 |

Things that need no money and are done or in progress: the public test suites, the ACVP sample-vector run,
the benchmark, the reproducible build, fuzzing on my own machine, and the security policy draft
(see README.md).
