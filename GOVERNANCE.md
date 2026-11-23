# Governance

**Onestar** — native Android build experiment without Gradle or AGP.

Document version: `1.0.0` · Date: 2026-11-23 · Status: **in effect**

---

## 0. The short version

This is a single-author project, published as open source. It is not a community-owned
project and it is not governed by vote.

One person holds final authority over the work. That authority is written down here, in
a file, on purpose: **the file outranks whatever buttons a hosting platform happens to
give someone.** Roles on GitHub are revocable; a document that states who the founder
is, and what cannot be taken from them, is not.

This repository lives in the maintainer's personal account. Section 1 states the
conditions under which it may move, and Section 1.5 records — without flattery — how far
it is from meeting them.

---

## 1. Where this project lives, and when that may change

**Current home: `wang-sunyang/onestar` — the maintainer's personal account.**

This is deliberate, and it is the rule for every project in this series:

> A project stays in the maintainer's personal account until it is **published** and its
> **collaboration loop is mature**. Only then may it move into the organisation
> `nightglean`.

The organisation exists for finished, publicly shipped work. It is not a staging area,
and premature repositories are not opened there.

### 1.1 Conditions for moving into `nightglean`

All four must hold. Meeting three of them is not meeting them.

| # | Condition | Test |
|---|---|---|
| A | **Published.** | A tagged, downloadable release exists and has been publicly obtainable for at least one release cycle. |
| B | **Collaboration loop is mature.** | At least three external contributions have completed the full loop — issue → PR → review → merge — with review recorded in the PR, under the contribution rules in force (`CONTRIBUTING.md`, and §2–§3 of this document where no such file exists). |
| C | **Governance is on disk.** | `GOVERNANCE.md` (this file), `TRADEMARK.md`, `CODEOWNERS`, and `AUTHORS.md` exist in the repository and are current. |
| D | **Ownership is stated.** | Copyright and trademark are declared as held by the individual author, and the organisation's use of both is described as licensed, not owned. |

### 1.2 What a move does not change

A move transfers **custody of the repository**, not ownership of the work.

- Copyright in the source, the build scripts, and the documentation remains with the
  individual author.
- The project name and marks remain the individual author's property. `nightglean`
  receives a licence to display them in connection with this project, and nothing more.
- This document travels with the repository and continues to apply in full.

### 1.3 Organisation ownership

When this repository moves, the organisation's Owner list contains **the founder only**,
until §1.1 is satisfied. Additional owners are added after governance is ratified, not
before — an Owner seat granted before the rules exist is a seat that can be used to
remove the founder from their own project.

### 1.4 This file is not a migration announcement

The existence of this file satisfies condition C only in part, and nothing else. It
declares no intent to move, and it grants no one any new rights.

### 1.5 Status of the conditions, as of 2026-11-23

A dated observation, not a standing claim. Re-read the test, not this table, before
treating any row as settled.

| # | Status | Basis |
|---|---|---|
| A | **Partly met** | Tagged releases exist: `v6.3`–`v6.7` published 2026-11-07 → 2026-11-08, plus `v0.12.2-rc1` on 2026-11-09. The v6 line has been public for roughly two weeks — shorter than one release cycle as this document reads it. |
| B | **Not met** | One issue and one PR in the repository's history, both opened by the maintainer. Zero external contributions have completed the loop. |
| C | **Met** | `GOVERNANCE.md`, `TRADEMARK.md`, `AUTHORS.md` and `.github/CODEOWNERS` all exist and are current; `main` rejects force-pushes and deletion, admins included. |
| D | **Met** | `LICENSE` (MIT) and `AUTHORS.md` name Wang Sunyang personally; `TRADEMARK.md` states the marks belong to the individual and are used by the organization under permission only. |

---

## 2. Ownership and licensing

- **Copyright:** 王孙阳 (Wang Sunyang), 2026. Stated in `LICENSE`.
- **Code licence:** MIT. See `LICENSE`.
- **Attribution in artefacts:** the version string in `assets/COPYRIGHT.txt` must match
  `AndroidManifest.xml`; that file is the in-APK attribution surface and is not optional.
- **Third-party components:** anything vendored or linked keeps its own licence. The
  repository layout in `README.md` names each directory, so provenance stays traceable.

The licence is not a governance document. It grants rights in the code; it does not
grant a say in the project. Section 3 covers who decides.

---

## 3. Decisions

| Role | Who | Powers | Granted by | Revoked by |
|---|---|---|---|---|
| **Founder / BDFL** | 王孙阳 (`wang-sunyang`) | Final decision on every matter in this section. Permanent seat. | Authorship | Nothing short of §4 |
| **Maintainer** | None at present | Would merge non-reserved changes, triage issues, cut releases. Scope written down when granted. | BDFL, in writing | BDFL, in writing |
| **Contributor** | Anyone with a merged PR | None beyond normal GitHub permissions | — | — |

**Reserved to the BDFL.** No maintainer, contributor, or organisation role may decide:

1. the licence, or any change to it;
2. the project name, marks, or the attribution rules in §2;
3. this document, `TRADEMARK.md`, `CODEOWNERS`, or `AUTHORS.md`;
4. moving the repository, or adding an organisation Owner;
5. rewriting published history or deleting a published release.

Technical judgement is delegated freely. These five are not technical questions.

---

## 4. Changing this document

Changes arrive by pull request and require the BDFL's approval. A change that weakens
§1.1 — lowering a condition, redefining "release cycle" downwards, or counting the
maintainer's own contributions as external — is not a normal change and will be refused.

If §1.1 is ever edited, the edit is recorded in the PR description together with the
reason. Silence in the history is not consent.
