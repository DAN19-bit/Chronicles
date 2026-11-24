# Chronicles — Testimony Submission MVP Proposal

**Issue:** [#1 — Plan and validate the testimony submission MVP](https://github.com/DAN19-bit/Chronicles/issues/1)
**Status:** Revision 2 — updated to address the Project Director review on PR #3 (reporter identity, unlisted access security, consent evidence/withdrawal, editing rules; plus age eligibility and third-party escalation). Planning document only — no application code, database schema, or production configuration has been changed.
**Owner:** Lead Developer agent (Implementation/Planning)
**Founder approval required:** Yes — see the open decisions list at the end.

---

## 1. Verified facts, assumptions, and recommendations

Per the protocol, this proposal separates the three categories explicitly.

### Verified facts (from repository documents)
- Chronicles is a hybrid Christian testimony archive and community (docs/PRODUCT.md, confirmed in docs/DECISIONS.md).
- The MVP principle is a safe, usable submission and discovery experience before social features.
- Testimonies are personal accounts, not independently verified facts; publication must not imply verification.
- The stack (Next.js + Supabase) is **proposed, not approved**. This proposal therefore describes requirements and a candidate data model in stack-neutral terms, with the proposed stack noted only where it maps cleanly.

### Assumptions (require founder confirmation)
- Contributors will register accounts before submitting (no fully anonymous submissions at launch).
- Text-first testimonies; media is deferred or optional behind an explicit founder decision.
- Moderation is review-before-publish at MVP scale (volume is expected to be low).
- English-only at launch.

### Recommendations (this agent's judgment, not decisions)
- A two-stage consent flow and a plain-language submission form (Section 3).
- Private-by-default visibility: a testimony is visible only to its author until approved and published.
- A minimal moderation model: one moderator role, an approval queue, a report button, and removal/correction paths.

---

## 2. Goals and non-goals

### Goals
1. A contributor can create an account, submit a testimony, and understand exactly how it will be used and shown.
2. A visitor can browse and read published testimonies by topic and date.
3. A moderator can review, approve, request changes, or remove testimonies.
4. Every published testimony carries clear consent and "personal account" framing.

### Non-goals for MVP
- Comments, reactions, witness accounts, and community profiles (the issue defines this as planning scope only; social features come after the archive MVP).
- Anonymous publishing (pending founder decision).
- Photo/video uploads (pending founder decision on media types and limits).
- Multi-language support, public APIs, or integrations.

---

## 3. Submission journey (proposed)

### Stage A — Write
1. Contributor signs in and opens the submission form.
2. Plain-language fields (see Section 4), autosaving drafts so nothing is lost.
3. Guidance text beside sensitive fields: "Do not include other people's full names or private details without their permission."

### Stage B — Consent and visibility
4. Before submitting, the contributor sees a short, plain-language summary of: how the testimony will be displayed, that it is presented as a personal account (not verified fact), and how to request edits or removal later.
5. Two explicit consent checkboxes (required):
   - "I consent to this testimony being published on Chronicles."
   - "I understand I can request editing or removal at any time, and that Chronicles will remove it from public display promptly while copies in backups or logs may persist for a limited period under the retention policy."
   The exact consent wording is recorded with the submission, together with a version label for the consent text and a timestamp, so we can later evidence what the contributor agreed to and when (see Section 6a).
6. Visibility options shown before submit:
   - **Public** — listed in browse/search.
   - **Unlisted** — readable only by direct link (useful for sharing with a church group without public listing). *Subject to founder decision D3/D4; recommended out of MVP scope until a security-reviewed access design exists (see Section 5, access-control principles).*
7. Display-name choice: real first name, pseudonym, or "Anonymous contributor" (display only; the account still exists — see open decision D2).
8. Eligibility check: the form states that contributors must be at least 13 years old (or the age of digital consent in their country, if higher), and requires an affirmative confirmation of this. Accounts believed to belong to children below that threshold are suspended and their testimonies removed from display pending founder review (see Section 6a, Age/eligibility rule).

### Stage C — Review
9. Submission enters the moderation queue with status `pending`.
10. A moderator approves, requests changes, or removes with a reason. The contributor is notified of any action that affects their testimony. Content meeting the escalation criteria is referred to the founder instead (Section 6a).

### Stage D — Publish and aftercare
11. Approved testimonies with `public` visibility appear in browse/search; if `unlisted` is approved at all, those are accessible only through a secure link mechanism (Section 5).
12. The author keeps edit and removal ("unpublish") rights at any time, without needing to justify. Removal from public display is prompt; deletion of copies held in backups or logs follows the retention policy once approved (D7) — the interface must not promise absolute erasure before that policy exists.
13. A "Report this testimony" link on every published testimony leads to a short report form (see Section 6a for reporter identity and abuse protections).

---

## 4. Required fields (proposed)

| Field | Required | Notes |
|---|---|---|
| Title | Yes | Max ~120 characters. |
| Story body | Yes | Min length to discourage spam; max to be decided (open decision D5). |
| Category/topic | Yes | Chosen from a small founder-curated list (e.g. healing, provision, faith, answered prayer). |
| Date of event | No | Free-text or approximate date; many testimonies span years. |
| Display name preference | Yes | Real first name / pseudonym / anonymous contributor. |
| Visibility | Yes | Public / unlisted. |
| Consent checkboxes | Yes | Two, as described in Stage B. |
| Age eligibility confirmation | Yes | At least 13 (or local digital-consent age, if higher); recorded in the consent record (Section 6a). |

No fields for real full name, location, contact details, or health identifiers are collected for display. Email is collected only for the account itself.

---

## 5. Candidate data model (stack-neutral, for Architect review)

This is a proposal for review, **not** a schema to implement. No database has been touched.

- **profiles** — one per account: internal account reference, chosen display name, role (`contributor`, `moderator`), timestamps.
- **testimonies** — the core record: author reference, title, body, category reference, display name choice (denormalized snapshot at submission time), visibility (`public`, `unlisted`), status (`draft`, `pending`, `published`, `removed`), moderation fields (reviewer reference, decision, reason, timestamps), timestamps.
- **categories** — founder-curated topic list: name, description, display order.
- **reports** — testimony reference, reason, free-text detail, status, timestamps, and a reporter reference that is nullable (see Section 6a: signed-in reporters are recorded; signed-out reports are anonymous).
- **consents** — one per submission (and per accepted re-consent): testimony reference, consent text version label, the exact choices made, and timestamps, providing evidence of what the contributor agreed to and when (Section 6a).

Access-control principles (aligning with docs/ARCHITECTURE.md principles, without implementing them):
- Authorization enforced at the data layer, not only in the interface.
- Authors see and edit only their own testimonies, with actions limited by status per the rules in Section 6a; everyone can read only `published` + `public` rows through ordinary public queries.
- Unlisted access is a security-sensitive capability: unlisted testimonies must never be retrievable through predictable identifiers or ordinary public listing/search queries. If unlisted is kept in scope (D3/D4), access requires a separate, high-entropy, non-enumerable secret link token reviewed by the Architect — otherwise the MVP is public-only. This is recommended out of MVP scope until that design is reviewed.
- Moderators need a defined, auditable role — not blanket access to all data.
- Personal data minimization: the display name snapshot is what is shown publicly; the account identity is never published.

---

## 6. Moderation, reporting, and removal (proposed MVP scope)

- **Review-before-publish** for all new submissions and re-submissions after a requested change.
- **Removal:** by the author (self-serve, immediate) or by a moderator (with reason recorded and author notified).
- **Correction:** authors edit published testimonies; edits to published testimonies go through a lightweight re-review or are applied immediately — an open decision (D6).
- **Reports:** every published testimony has a report action; reports land in the same moderation queue.
- **Appeals:** at MVP scale, a simple "contact the moderators" path (email or form) is sufficient; a formal appeal workflow is deferred.
- Editorial guardrail from docs/PRODUCT.md carried into the interface: framing text on every testimony that it is a personal account, and no promises of outcomes.

### 6a. Clarifications from the Project Director review (PR #3)

**Reporter identity and abuse protection.** The report form is open to signed-in contributors and to signed-out visitors. Signed-in reports record the reporter's account reference (visible to moderators only); signed-out reports are stored with no reporter reference and no collected personal data beyond the report fields themselves. Abuse protections, without collecting unnecessary personal data: rate limiting per network/client on signed-out reports, a limit on open reports per account, duplicate-report collapsing for the same testimony, and free-text length caps. Reports by visitors cannot read moderation outcomes; reporters who want follow-up are encouraged to sign in.

**Consent evidence, withdrawal, and retention.** Each submission stores a consent record: the version label of the consent text shown, the contributor's affirmative choices, and a timestamp; re-consent after material wording changes creates a new record rather than overwriting. Withdrawal of consent is the removal path: the testimony leaves public display promptly and unconditionally. The interface and policy must not promise absolute erasure "at any time" — copies in backups and operational logs persist for a limited period under a documented retention schedule, which is founder decision D7. Until D7 is approved, the interface says only: "removed from public display promptly; residual copies may persist for a limited time."

**Editing rules by status.** Allowed author actions are defined per status to remove the ambiguity between "edit in all statuses" and re-review:

| Status | Author may read | Author may edit | Notes |
|---|---|---|---|
| draft | yes | yes | freely editable |
| pending | yes | yes (withdraw and resubmit) | editing resets it to `pending` with the new version queued |
| published | yes | yes | the edited version returns to `pending` and the previously approved version stays publicly visible until the edit is approved — no temporary unpublication, so there is no window where an approved testimony vanishes |
| removed | yes (read-only) | no | author may appeal via the contact path |

If the founder prefers simpler behavior, D6 offers the alternative that published testimonies are temporarily unpublished while an edit is under review; the recommended default is the stay-visible approach above. Edits never bypass review-before-publish.

**Age/eligibility rule.** Contributors confirm they are at least 13 years old, or the minimum age of digital consent where they live if higher. The confirmation is captured as part of the consent record. The MVP does not verify ages; if a report or moderation review indicates a contributor is below the threshold, the account is suspended and testimonies are removed from display pending founder review. Whether a stricter minimum age (e.g. 16) applies is a founder decision (D10).

**Escalation for sensitive third-party information.** Some testimony content requires more than routine moderation — for example, credible risk to a person's safety, content about minors by non-guardians, or alleged crimes. For such content, the moderator does not approve or edit it themselves: it is marked escalated, removed from the queue, and referred to the founder with the reason recorded. The founder decides the outcome. The escalation criteria and contact route are documented in the moderation guide before launch, and the form's guidance text steers contributors away from including third-party identifying details in the first place (Stage A, step 3).

---

## 7. Acceptance criteria and test plan (draft for QA review)

### Acceptance criteria
1. A signed-in contributor can save a draft, submit it, and see its status.
2. A submission cannot complete without both consent checkboxes.
3. A non-author cannot read a testimony that is draft, pending, or removed; unlisted testimonies, if in scope, cannot be reached through public queries or predictable identifiers (Section 5).
4. A visitor can browse and search published public testimonies by category and date.
5. A moderator can approve, request changes, or remove a pending/published testimony, with the action recorded.
6. An author can unpublish their testimony at any time; it disappears from public views immediately.
7. A visitor can report a published testimony; the report appears in the moderation queue. Signed-out reports carry no reporter reference; rate limiting prevents report flooding.
8. Every published testimony shows its display name, category, and the personal-account framing notice.

### Test plan
- **Journey tests (manual):** walk Stages A–D end to end with a test account; verify emails/notifications and status transitions.
- **Access-control tests (priority):** for each pair of roles (visitor, contributor, author, moderator), attempt every read/write combination against testimonies in every status. Any unauthorized success is a release blocker.
- **Consent tests:** submitting without either checkbox fails with a clear message; the consent record captures the text version, choices, and timestamp; withdrawing consent removes the testimony from public display promptly.
- **Validation tests:** over-length fields, empty fields, unsupported content; verify friendly errors, no data loss.
- **Privacy review:** confirm no real name, email, or account identifier is exposed on public testimony views.
- **Removal tests:** author unpublish and moderator removal both leave the testimony invisible publicly and record who/when/why; editing a published testimony keeps the approved version visible until the edit passes review (or, under the D6 alternative, temporarily unpublishes it).
- QA/Security should run these independently and record reproducible evidence per docs/AGENT_ROLES.md.

---

## 8. Risks and privacy considerations

| Risk | Mitigation proposed |
|---|---|
| Testimonies include third parties' personal details | Guidance text in the form; moderation checks for it; removal path for complaints; escalation to founder for high-sensitivity cases (Section 6a). |
| Vulnerable contributors share under emotional pressure | Plain-language consent; self-serve removal at any time; no dark patterns on the removal path. |
| Moderator overload or absence | Founder acts as moderator at MVP scale; queue is intentionally tiny. |
| Content discouraging medical care | Editorial guardrail text; moderation rejects content promising outcomes. |
| Re-identification via story details | Display-name control; a note encouraging contributors to consider what details identify others or themselves. |
| Data retention beyond need | Retention/deletion policy is an open founder decision (D7). |

---

## 9. Unresolved founder decisions

| # | Decision | Options |
|---|---|---|
| D1 | Anonymous submissions (no account) allowed at launch? | Yes / no / deferred. |
| D2 | Fully anonymous display names — accepted, or pseudonym/first-name only? | Affects trust and abuse profile. |
| D3 | Is `unlisted` visibility in MVP scope, or public-only? | Simpler if public-only. |
| D4 | How unlisted sharing works (secret link token vs. signed-in link viewers). | Token is simpler; token leaks are untraceable. |
| D5 | Story length limits. | Suggest generous max (e.g. ~10,000 characters) at MVP. |
| D6 | Editing behavior for published testimonies. | Recommended: edited version re-enters review while the approved version stays visible (Section 6a); alternative: temporarily unpublish during review. |
| D7 | Data retention and account-deletion policy (what happens to testimonies when an account is deleted; how long backups/logs retain removed content). | Options: keep published testimonies (with display choice honored), unpublish them, or delete them; backup retention window to be documented. |
| D8 | Initial category list. | Founder-curated; the implementation agent can propose a starter set. |
| D9 | Approval of the candidate data model in Section 5 for Architect review. | Required before any schema work. |
| D10 | Minimum age / eligibility rule. | 13 (or local digital-consent age) proposed in Section 6a; founder may set a stricter minimum. |
| D11 | Escalation policy for sensitive third-party content (criteria and contact route). | Proposed in Section 6a; founder confirms criteria and acts as escalation contact at MVP scale. |

---

## 10. Proposed next steps

1. Founder reviews this proposal and answers (or defers) the decisions in Section 9.
2. Architect agent reviews the data model and access-control principles.
3. QA/Security reviews the test plan for coverage.
4. Only after data-model approval: implementation issue is created, schema is designed via migration on a task branch, and the submission/build/browse flows are implemented.
5. Founder approves any merge and every production release.

---

## Handoff (per docs/AGENT_PROTOCOL.md)

- **Task and status:** Issue #1 planning; revision 2 complete, addressing the Project Director review on PR #3.
- **Work completed:** research summary, submission journey, fields, candidate data model, moderation scope, acceptance criteria, test plan, risks, founder decision list; plus (revision 2) reporter identity and abuse protections, secure unlisted-access principles, consent evidence/withdrawal/retention wording, per-status editing rules, age/eligibility rule, and third-party escalation path.
- **Files changed:** this document only (`docs/PROPOSAL_TESTIMONY_MVP.md`), on branch `task/issue-1-mvp-proposal`.
- **Decisions and rationale:** recommendations only; all major decisions deferred to the founder (Section 9).
- **Tests run and results:** none — planning task; the test plan itself is the deliverable (Section 7).
- **Unresolved questions:** Section 9 (D1–D11).
- **Risks and privacy/security implications:** Section 8; unlisted access treated as security-sensitive and recommended out of MVP until Architect review.
- **Proposed next owner/action:** Founder → Architect → QA, per Section 10.
- **Founder approval required:** Yes — product decisions D1–D8, D10–D11, and data-model direction D9.
