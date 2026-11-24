# Chronicles Agent Communication Protocol

## Purpose
This document defines how specialist agents collaborate through the Chronicles repository. GitHub is the shared source of truth; separate provider chat histories are not authoritative project records.

## Roles
- Project Director: coordinates tasks, maintains status, and reports to the founder.
- Product Research: researches users, requirements, and product options.
- Architect: proposes system design and reviews technical decisions.
- Implementation: builds only the assigned scope and submits a pull request.
- QA/Security: independently checks acceptance criteria, privacy, security, and regressions.
- Founder: approves major product/architecture decisions and all production releases.

## Task lifecycle
1. Create a GitHub Issue with objective, context, scope, exclusions, acceptance criteria, risks, owner, and approval needs.
2. Assigned agent acknowledges the task and reads relevant project documents.
3. Research/design findings are recorded in the issue or a linked document.
4. Implementation happens on a task branch; do not commit directly to the default branch unless the founder explicitly authorizes it.
5. Implementer opens a pull request with summary, changed files, test evidence, known limitations, and rollback considerations.
6. QA/Security reviews independently and records findings.
7. Project Director summarizes the outcome and identifies any founder decision required.
8. Founder approves major changes and production releases. No agent may treat silence as approval.

## Required handoff format
- Task and current status
- Work completed
- Files/links changed
- Decisions and rationale
- Tests run and results
- Unresolved questions
- Risks and privacy/security implications
- Proposed next owner/action
- Founder approval required: yes/no, with reason

## Discussion rules
- Challenge ideas respectfully and provide evidence or a reproducible reason.
- Separate verified facts, assumptions, and recommendations.
- Record unresolved disagreements; escalate material conflicts to the Project Director and founder.
- Do not invent test results, user research, or external verification.
- Do not place secrets, credentials, or private testimony content in issues or public repository files.

## Permission boundaries
Agents may draft, research, code on assigned branches, run checks, and submit pull requests. They may not independently merge major changes, deploy production, modify production data, rotate credentials, or change privacy/security policy.
