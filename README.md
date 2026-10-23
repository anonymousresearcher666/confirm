# ConFIRM
**Independently Verifiable Trust Boundaries for Web Knowledge-Graph Reasoning**

*Anonymous research artifact for The Web Conference (WWW) 2027 — Semantics and Knowledge*

ConFIRM is a research prototype for **safe consumption of results produced by untrusted Web knowledge-graph (KG) reasoning services**. A remote reasoning provider proposes an answer and supplies a proof-sensitive search transcript; an independently operated checker reconstructs the bounded search from authorized source data and rule policies. A downstream application acts only on an independently verified receipt.

> **Research contribution:** ConFIRM establishes an independently verifiable trust boundary between untrusted Web knowledge-graph reasoning services and downstream semantic applications.

The contribution concerns **application integrity and trust**, not a new general-purpose Datalog engine, general OWL entailment, or biological discovery.

## 1. Motivation

A third-party KG reasoner might label a claim as `PROVEN` and provide a plausible proof. This does not establish that the reasoner used the approved source snapshot, obeyed the distinction between trusted and candidate rules, explored every required search branch, or preserved the search record between verification and consumption. Checking *one proof* is different from independently checking **bounded full-search completeness and the resulting authority of a claim**.

For example, a semantic application may receive a proposed GO/GO-CAM annotation. If a provider silently changes an original `UNKNOWN` result to `PROVEN`, removes a transition from the search history, or replaces a source digest, a naive consumer may authorize an unsupported action. ConFIRM treats those artifacts as untrusted and independently reconstructs the permitted search before allowing any authorization.

## 2. Architecture and trust model

```text
       TRUSTED OPERATOR
 source + schema + rule policy + query manifest
        /                          \
       v                            v
UNTRUSTED PRODUCER         INDEPENDENT CHECKER
proposed verdict           loads pinned trusted sources
full search transcript ---> replays bounded search frontier
pruning certificates       checks certificates and verdicts
source/status claims       binds source + query + exact trace
                                 |
                                 v
                         VERIFIED RECEIPT
                                 |
                                 v
                         DOWNSTREAM CONSUMER
                  AUTHORIZE / DENY / ABSTAIN / QUARANTINE
```

**Trusted:** operator-controlled source files, query manifest, schema, trusted/candidate/rejected rule designations, checker, and verification receipt custody.

**Untrusted:** producer-provided verdicts, source/status claims, search traces, certificate assertions, and any unverified final answer.

The checker reloads the authorized finite Horn source and independently reconstructs the ordered bounded search (including generated children, terminals, certificates and verdicts). Accepted receipts are bound to the exact transcript bytes, source digest, and original query. W1 uses separate OS processes and local JSON IPC; W2 uses separate Docker service roles and HMAC-bound receipts under controlled network faults **on one trusted host**.

**Boundaries:** This does not authenticate original Web publishers, protect a compromised operator or privileged host, certify factual biological truth, prove how the remote producer historically executed, or implement unrestricted OWL reasoning.

## 3. Outcomes and consumer behavior

| Verified result | Meaning under pinned source and bounded rules | Application policy |
| --- | --- | --- |
| **PROVEN** | Independently reconstructed trusted-rule support | Authorize an otherwise permitted action |
| **HYPOTHESIZED** | Support relies on a candidate (not trusted) rule | Abstain; do not promote candidate evidence |
| **REFUTED** | An applicable source-backed qualified negative witness was checked | Deny that particular claim |
| **UNKNOWN** | No qualifying conclusion within the supported bounded search | Abstain; not biological falsity |
| **Invalid / forged / stale** | Source, rule authority, frontier, witness, query, or receipt failed verification | Quarantine |

GAF `NOT` annotations are **relation- and extension-qualified**, not unrestricted negations of any GO association.

## 4. Implementation

| Component | Files |
| --- | --- |
| Producer and certificates | `src/trustkg/trust.py`, `src/trustkg/certificate_evidence.py`, `scripts/capture_www2027_go_transition_audit.py` |
| Independent checker | `src/trustkg/independent_checker.py`, `src/trustkg/independent_transition_audit.py`, `src/trustkg/independent_verdict_audit.py`, `scripts/verify_www2027_go_transition_audit.py` |
| Consumer action policy | `scripts/www2027_web_consumer_process.py`, `src/trustkg/downstream_boundary.py` |
| Genuine GO/GAF source-backed cohort | `scripts/evaluate_www2027_p24_full_go_real_not_mixed.py`, `scripts/evaluate_www2027_p28_real_kg_three_statuses.py` |
| Distributed services | `experiments/www27/w2/`, `scripts/run_www2027_w2_distributed.py` |
| Regression tests | `tests/test_www2027_*.py` |
| Reference evidence | `results/www2027/` |

The installable Python module is named `trustkg`. This archive retains experiment-relevant components; it is not a snapshot of every ancillary development file.

## 5. Experiments and measured results

The JSON reports supplied with this archive are **previously recorded controlled measurements**, not a fresh execution when you download the ZIP.

| Study | Research question | Reference observation |
| --- | --- | --- |
| **W1 — Action integrity** | Can an untrusted producer alter decisions, searches, witnesses, or trace binding? Compare naive producer-only, strongly anchored final-only, and full independent replay | Full checker rejects **9/9** forged producer traces; exact post-verification trace binding rejects **1/1** substitution (**10/10** controlled unsafe requests blocked). Naive consumer authorizes three forged `PROVEN` cases; anchored final-only misses four internal manipulations |
| **W2 — Distributed faults** | Can verified action decisions fail closed with network faults, stale sources, and forged receipts? | **31/31** controlled scenarios complete, **zero unauthorized actions** among tested cases on a single trusted Docker host |
| **W3 — Request isolation** | Can a bad response be isolated without unnecessarily quarantining all legitimate requests? | Per-query fresh independent capture and replay preserves **4** or **3** legitimate authorizations in two paired eight-query attacks, versus whole-batch quarantine |
| **W4 — Warm checker cost** | Can repeated full verification amortize source/index loading without weakening safety? | 11 matched traces (8 genuine, 3 altered): **16.021414 s** summed cold processes vs **1.601870 s** one warm process; **10.00169x** *conditional verifier amortization*, identical authorization outcomes and three attack rejections |

W3 uses one reused eight-query cohort, not seven independent applications. W4 does not establish 10x distributed end-to-end throughput. W2 has independently separated service roles, but **not** an Internet-scale public deployment or Byzantine-host guarantee.

Reference reports:

```text
results/www2027/www27_w1_web_consumer_v1.json
results/www2027/www27_w2_distributed_faults_v1.json
results/www2027/www27_w3_web_application_safe_batch_isolation_v1.json
results/www2027/www27_w4_warm_checker_web_application_boundary_v2.json
results/www2027/www27_nemo_p11_gocam_t3_t4_balanced_v1.json
```

The included official Nemo report provides a **separate ground-membership validation used to construct the GO-CAM cohort**. It is not a matched independent full-frontier proof verification benchmark.

## 6. Installation and smoke tests

Recommended environment: Python 3.12; Docker Engine/Desktop or Colima with Docker Compose v2 is needed **only for W2**.

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e . pytest
export PYTHONPATH="$PWD/src:$PWD/scripts:$PWD/experiments/www27/w2"
```

Run the included experiment-focused tests:

```bash
python -m pytest -q \
  tests/test_www2027_w1_web_consumer.py \
  tests/test_www2027_w2_services.py \
  tests/test_www2027_w3_application_trust_boundary.py \
  tests/test_www2027_w4_persistent_safe_web_consumer.py \
  tests/test_www2027_independent_transition_audit.py \
  tests/test_www2027_independent_verdict_audit.py
```

Inspect the archived reports immediately, **without claiming a new run**:

```bash
python - <<'PY'
import json
from pathlib import Path
for p in sorted(Path('results/www2027').glob('www27_w*.json')):
    print(p.name, json.loads(p.read_text()).get('passed'))
PY
```

These are source-level smoke checks and existing result inspection. **They do not establish successful clean-room reproduction of all W1–W4 cases.**

## 7. External datasets: required for full replay

To rerun the published experiments, supply the **exact frozen original data bytes** in these locations:

```text
data/go_case_study/
  manifest.json                    # INCLUDED
  processed/
    facts.jsonl                    # external frozen input
    queries.jsonl                  # external frozen input
    schema.json                    # external frozen input
    trusted_rules.jsonl            # external frozen input
  raw/
    goa_human.gaf.gz               # external frozen input
    mgi.gaf.gz                     # external frozen input
```

The full-GO source builder checks the following SHA-256 values:

| Processed input | Exact SHA-256 |
| --- | --- |
| `facts.jsonl` | `4b7d88e5b8644d72852ec66b0214cf8a22efce244fc4ea94aa97d4f111406a3b` |
| `queries.jsonl` | `582cf475116ec44caa39008363d85387e7aea6055f514bbae902d59bb78db7cb` |
| `schema.json` | `7054e2d96778b9818cc9dfa2f93f2439a1c9b91562c2b56ee3f28e597b1e9251` |
| `trusted_rules.jsonl` | `d8d9288326b8bff16af5e168719ea4895738629e612ced4ccb5f6701f1022260` |

The historical human and mouse raw GAF hashes are recorded in `data/go_case_study/manifest.json` and validated by the evaluation code. **Do not silently substitute current Web downloads for original pinned archives.** Upstream GO/GAF redistribution rights are distinct from this code's MIT license.

## 8. Run W1–W4 after restoring the frozen inputs

Use the environment variables from section 6 and run from the extracted artifact root. Results are written to **new** files under `reproduced/`, never over the archived reference JSON:

```bash
mkdir -p reproduced

# W1: independent replay versus unsafe/final-only consumer
python scripts/evaluate_www2027_w1_web_consumer.py \
  --timeout 180 --output reproduced/w1.json

# W2: real Docker services and Toxiproxy faults (start Docker first)
python scripts/run_www2027_w2_distributed.py \
  --repetitions 3 --output reproduced/w2.json

# W3: fresh per-query capture, attack injection, action isolation
python scripts/evaluate_www2027_w3_application_trust_boundary.py \
  --timeout 180 --output reproduced/w3.json

# W4: cold versus warm independent full checker
python scripts/evaluate_www2027_w4_persistent_safe_web_consumer.py \
  --timeout 180 --output reproduced/w4.json
```

Check both the process exit status and each new JSON's top-level `"passed": true`. Missing datasets, missing Docker, timeouts, integrity mismatches and incomplete test cells are **BLOCKED/FAILED**, not successful reproduction. Runtime measurements vary with hardware and warm/cold process lifecycle.

## 9. Current reproducibility status and limitations

**Available now:** code for the specified experiments, regression tests, Docker definitions, pinned source manifests, and original machine-readable result files.

**Not yet independently validated:** complete clean-environment W1–W4 execution using only this ZIP. The exact frozen GO/GAF files are external, and full end-to-end reruns require them. This README intentionally distinguishes *inspection of original evidence* from *new independent experimental reproduction*.

ConFIRM does not guarantee that its operator-authorized source was never semantically poisoned, that an original Web publisher's identity is cryptographically attested, that a privileged host is trustworthy when compromised, or that its local bounded Horn semantics represent every OWL or biological conclusion.

## 10. Double-blind review and license

The source snapshot omits Git history, paper manuscript, personal contact metadata, host-specific logs and local workspace details. **Do not cite or publicly share an identified repository-owner URL during double-blind review**. Submit the extracted ZIP through the venue's approved anonymous artifact channel.

License: **MIT**, as provided in `LICENSE`; third-party datasets retain their upstream licensing terms.

