# Chronicles — Testimony Submission MVP Proposal

**Issue:** [#1 — Plan and validate the testimony submission MVP](https://github.com/DAN19-bit/Chronicles/issues/1)
**Status:** Draft for review. Planning document only — no application code, database schema, or production configuration has been changed.
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
   - "I understand I can request editing or removal at any time."
6. Visibility options shown before submit:
   - **Public** — listed in browse/search.
   - **Unlisted** — readable only by direct link (useful for sharing with a church group without public listing).
7. Display-name choice: real first name, pseudonym, or "Anonymous contributor" (display only; the account still exists — see open decision D2).

### Stage C — Review
8. Submission enters the moderation queue with status `pending`.
9. A moderator approves, requests changes, or removes with a reason. The contributor is notified of any action that affects their testimony.

### Stage D — Publish and aftercare
10. Approved testimonies with `public` visibility appear in browse/search; `unlisted` ones are link-only.
11. The author keeps edit and removal ("unpublish") rights at any time, without needing to justify.
12. A "Report this testimony" link on every published testimony leads to a short report form.

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

No fields for real full name, location, contact details, or health identifiers are collected for display. Email is collected only for the account itself.

---

## 5. Candidate data model (stack-neutral, for Architect review)

This is a proposal for review, **not** a schema to implement. No database has been touched.

- **profiles** — one per account: internal account reference, chosen display name, role (`contributor`, `moderator`), timestamps.
- **testimonies** — the core record: author reference, title, body, category reference, display name choice (denormalized snapshot at submission time), visibility (`public`, `unlisted`), status (`draft`, `pending`, `published`, `removed`), moderation fields (reviewer reference, decision, reason, timestamps), timestamps.
- **categories** — founder-curated topic list: name, description, display order.
- **reports** — reporter reference, testimony reference, reason, free-text detail, status, timestamps.

Access-control principles (aligning with docs/ARCHITECTURE.md principles, without implementing them):
- Authorization enforced at the data layer, not only in the interface.
- Authors see and edit only their own testimonies in all statuses; everyone can read only `published` + `public` rows (plus unlisted rows by link, which needs a design decision — see D4).
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

---

## 7. Acceptance criteria and test plan (draft for QA review)

### Acceptance criteria
1. A signed-in contributor can save a draft, submit it, and see its status.
2. A submission cannot complete without both consent checkboxes.
3. A non-author cannot read a testimony that is draft, pending, or removed.
4. A visitor can browse and search published public testimonies by category and date.
5. A moderator can approve, request changes, or remove a pending/published testimony, with the action recorded.
6. An author can unpublish their testimony at any time; it disappears from public views immediately.
7. A visitor can report a published testimony; the report appears in the moderation queue.
8. Every published testimony shows its display name, category, and the personal-account framing notice.

### Test plan
- **Journey tests (manual):** walk Stages A–D end to end with a test account; verify emails/notifications and status transitions.
- **Access-control tests (priority):** for each pair of roles (visitor, contributor, author, moderator), attempt every read/write combination against testimonies in every status. Any unauthorized success is a release blocker.
- **Consent tests:** submitting without either checkbox fails with a clear message; consent state is recorded with the submission.
- **Validation tests:** over-length fields, empty fields, unsupported content; verify friendly errors, no data loss.
- **Privacy review:** confirm no real name, email, or account identifier is exposed on public testimony views.
- **Removal tests:** author unpublish and moderator removal both leave the testimony invisible publicly and record who/when/why.
- QA/Security should run these independently and record reproducible evidence per docs/AGENT_ROLES.md.

---

## 8. Risks and privacy considerations

| Risk | Mitigation proposed |
|---|---|
| Testimonies include third parties' personal details | Guidance text in the form; moderation checks for it; removal path for complaints. |
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
| D6 | Do edits to published testimonies require re-review? | Re-review is safer; immediate is friendlier. |
| D7 | Data retention and account-deletion policy (what happens to testimonies when an account is deleted). | Options: keep published testimonies (with display choice honored), unpublish them, or delete them. |
| D8 | Initial category list. | Founder-curated; the implementation agent can propose a starter set. |
| D9 | Approval of the candidate data model in Section 5 for Architect review. | Required before any schema work. |

---

## 10. Proposed next steps

1. Founder reviews this proposal and answers (or defers) the decisions in Section 9.
2. Architect agent reviews the data model and access-control principles.
3. QA/Security reviews the test plan for coverage.
4. Only after data-model approval: implementation issue is created, schema is designed via migration on a task branch, and the submission/build/browse flows are implemented.
5. Founder approves any merge and every production release.

---

## Handoff (per docs/AGENT_PROTOCOL.md)

- **Task and status:** Issue #1 planning; draft proposal complete, awaiting review.
- **Work completed:** research summary, submission journey, fields, candidate data model, moderation scope, acceptance criteria, test plan, risks, founder decision list.
- **Files changed:** this document only (`docs/PROPOSAL_TESTIMONY_MVP.md`), on branch `task/issue-1-mvp-proposal`.
- **Decisions and rationale:** recommendations only; all major decisions deferred to the founder (Section 9).
- **Tests run and results:** none — planning task; the test plan itself is the deliverable (Section 7).
- **Unresolved questions:** Section 9 (D1–D9).
- **Risks and privacy/security implications:** Section 8.
- **Proposed next owner/action:** Founder → Architect → QA, per Section 10.
- **Founder approval required:** Yes — product decisions D1–D8 and data-model direction D9.
