- Feature Name: `diagnostics_bundle_evidence_artifact`

# Summary
[summary]: #summary

Define a governed diagnostics-bundle operating model for SourceOS/SociOS: local-first capture, explicit tiering, raw-versus-share artifact separation, deterministic redaction, and an evidence-contract handoff into the canonical typed-contract lane.

# Motivation
[motivation]: #motivation

We need a repeatable host-diagnostics primitive that is operator-usable, privacy-conscious, and machine-checkable.

The gap today is not just "how do we run a few commands"; it is how we standardize the resulting artifact so that it can be:

- captured locally on immutable or semi-immutable hosts
- reviewed and diffed later
- exported safely when support, incident, or forensics workflows require it
- signed, catalogued, or policy-checked by downstream commons tooling

This enhancement supports the expected SourceOS/SociOS outcomes:

- no background collection by default
- no silent surveillance posture
- explicit capture tiers for increasingly invasive probes
- a stable contract shape that runtime and packaging repos can consume

# Explanation and Examples
[explanation-and-examples]: #explanation-and-examples

The proposal introduces a diagnostics bundle as a first-class evidence artifact with the following operating rules.

## 1. Capture is local-first and explicit

Diagnostics capture is initiated by a local operator, user, or explicitly authorized automation path. It is not enabled by default as a scheduled or ambient collection surface.

## 2. Capture is tiered

The bundle supports capture tiers.

- Tier 0: safe baseline inventory and logs (uptime, journal slices, process snapshot, network summary, mounts)
- Tier 1: deeper but still generally safe inventory (firewall rules, SELinux state, podman summaries, firmware inventory)
- Tier 2: invasive or high-sensitivity probes (for example perf sampling or packet capture)

Higher tiers must be explicit. The bundle records which tiers were enabled.

## 3. Raw and share-safe forms are distinct

A diagnostics capture may produce:

- a raw local artifact tree
- a redacted share-safe artifact tree
- an optional packaged tarball

The redaction policy is recorded as part of the evidence object so that reviewers can tell what was masked and what was intentionally preserved.

## 4. Integrity is first-class

The bundle records per-probe stdout/stderr hashes and may also record bundle-level digests and signatures. This makes later transport, comparison, and attestation more defensible.

## 5. Repo ownership stays split

- `SourceOS-Linux/sourceos-spec` owns the machine-readable contract for the bundle
- `SociOS-Linux/enhancements` owns the design and governance rationale
- a runtime repo under `SociOS-Linux` or a revived `socios` lane should own the actual collector implementation
- `SociOS-Linux/socios-coreos-config` should own immutable-image inclusion once the runtime capability is stable

## Example operator flow

1. Operator runs a local diagnostics capture at tier 0/1.
2. The runtime emits a `DiagnosticsBundle` object and a raw artifact tree.
3. A share-safe artifact tree is generated with deterministic masking.
4. The operator or automation layer exports only the share-safe form unless stronger authorization exists.

# Drawbacks
[drawbacks]: #drawbacks

- This adds another evidence object family to the ecosystem, which means more schema and validation work.
- Redaction policies can create false confidence if operators mistake a share-safe artifact for a complete forensic record.
- Tiering creates operational policy decisions that must be documented and enforced consistently.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
  - It separates contract, governance rationale, runtime implementation, and image packaging into the repos that already own those responsibilities.
  - It keeps capture explicit and local-first, which matches the SourceOS/SociOS trust posture.
  - It preserves a clean raw/share distinction instead of mixing privacy decisions into ad hoc scripts.

- What other designs have been considered and what is the rationale for not choosing them?
  - Keeping diagnostics as a loose script format was rejected because it creates no canonical evidence shape.
  - Reusing `RunRecord` directly was rejected because diagnostics custody and redaction posture are not the same thing as workload execution provenance.
  - Landing first in stale runtime repos was rejected because repo vitality matters for initial alignment.

- What is the impact of not doing this?
  - Every runtime or support path will invent its own host-diagnostics artifact format.
  - Privacy and masking behavior will drift.
  - Incident handoff and downstream validation will remain inconsistent.
