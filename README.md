# AI-Assisted Code Provenance and Attestations

Provenance can answer **who asserted what about which artifact**. It cannot prove that generated code is correct, original, licensed, secure, or even accurately described.

This guide proposes a minimal evidence envelope for AI-assisted changes and shows how it can sit beside established software-supply-chain formats. The AI-specific mapping is an **author design for review**, not a claim that SLSA, SPDX, or in-toto standardizes prompt disclosure.

## Minimal evidence chain

```text
source commit / reviewed patch
        ↓ subject digest
AI-assistance statement (typed predicate)
        ↓ DSSE envelope + signer
attestation stored beside build/release evidence
        ↓ verifier checks identity, digest, policy
build provenance / SBOM remain separate linked evidence
```

Four roles must not be collapsed:

- **Subject:** immutable digest of the patch, source tree, or artifact being described.
- **Predicate:** typed statement about assistance and human review.
- **Signer:** accountable workload/person identity under a declared policy.
- **Verifier:** checks signature, subject binding, predicate type/version, and policy—without assuming content truth.

## A conservative AI-assistance predicate

Use [`assets/provenance-envelope.yaml`](assets/provenance-envelope.yaml) as a local design sketch. Retain only what review requires:

- subject digest and repository/base commit;
- tool/model identifiers at the granularity actually known;
- whether prompts/outputs are retained, redacted, or unavailable;
- changed paths and human reviewer decision;
- tests/security checks referenced by digest;
- explicit unknowns and policy version.

Do **not** embed secrets, private source, full prompts, personal data, or vendor tokens in a public attestation. A hash of sensitive low-entropy text may still leak through guessing; follow the project’s retention/privacy policy.

## Validation sequence

1. Recompute the subject digest from the reviewed artifact.
2. Verify envelope signature and expected signer identity.
3. Reject unknown predicate versions unless policy allows them.
4. Confirm base commit/changed paths match the reviewed patch.
5. Resolve evidence references by digest and ensure tests were run on the same subject.
6. Evaluate policy: required human review, disclosure level, retention, and unresolved unknowns.
7. Keep verification result separate from correctness/license conclusions.

## Standards: core, adjacent, and non-substitutes

| Layer | Standard/tool | Appropriate role | Does not establish |
|---|---|---|---|
| Build provenance | [SLSA Provenance v1.0](https://slsa.dev/spec/v1.0/provenance) | Build process/material predicate bound to an artifact | AI involvement or code correctness |
| Supply-chain statement | [in-toto](https://github.com/in-toto/in-toto) | Signed metadata for steps, materials, and products | Truth of an unverified custom claim |
| Signature/distribution | [Sigstore Cosign](https://github.com/sigstore/cosign) | Sign/verify blobs, containers, and attestations | Predicate sufficiency |
| Envelope | [DSSE](https://github.com/secure-systems-lab/dsse) | Sign arbitrary typed payloads | A standard AI-code predicate |
| Package/license inventory | [SPDX](https://spdx.dev/specifications/) | Software/package/file and license metadata | How a code block was generated |
| Component inventory | [CycloneDX](https://cyclonedx.org/specification/overview/) | Components/services and related BOM data | Prompt history or review quality |

**Adjacent, not core provenance for code:** [C2PA](https://spec.c2pa.org/specifications/specifications/2.2/index.html) is relevant to signed content credentials for media/assets. [OpenSSF Scorecard](https://github.com/ossf/scorecard) reports repository-practice signals. Neither substitutes for a subject-bound AI-assistance statement, so both are excluded from the core chain.

## Worked record (illustrative)

A maintainer reviews a patch at base commit `B`, stores patch digest `P`, runs test report digest `T`, and signs predicate version `example.ai-assistance/v0.1`. The verifier confirms signer, `P`, `B`, and `T`. The valid conclusion is:

> “The expected signer attested that this patch used the declared assistance and received the recorded review/tests under policy X.”

It is **not**: “the code is safe,” “the model owns no copyrighted training data,” or “the license is legally cleared.” Those need separate technical/legal evidence.

## Interoperability questions before adoption

- Where is the attestation stored and discovered?
- Which digest/URI identifies source versus built artifact?
- Which identity root and revocation model does the verifier trust?
- Is the custom predicate type/version stable and documented?
- Can downstream policy engines parse it without exposing private content?
- How are corrected/revoked attestations represented?

Until two independent implementations consume the custom predicate, call it a project schema, not an interoperable standard.

## Scope boundary

This repository addresses evidence format and verification. It does not decide lawful source, similarity/plagiarism, clean-room procedure, code quality, or release safety. Dependency update gates may link SBOM/provenance evidence but remain a separate approval lifecycle.

## Maintenance

Version the predicate on semantic changes. Preserve old schemas and fixtures. Recheck the pinned standard version before changing field mappings; homepage availability alone is not evidence of interoperability.

Sources: [`sources/sources.json`](sources/sources.json).

## License

Original guide and example schema are available under CC BY 4.0. Linked standards retain their terms.
