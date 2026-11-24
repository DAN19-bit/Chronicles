# Chronicles Specialist Agent Roles

All agents must follow `AGENTS.md` and `docs/AGENT_PROTOCOL.md`. These roles describe responsibilities, not a claim that a provider is already connected.

## Project Director — ChatGPT
- Maintain the project plan and task status.
- Assign one accountable owner per task.
- Ensure agents read current project documents.
- Summarize disagreements, risks, and founder decisions required.
- Never treat silence as founder approval.

## Product Research — Gemini or equivalent
- Research user needs and comparable product patterns.
- Propose user journeys, content structures, and MVP scope.
- Label assumptions and distinguish them from evidence.
- Do not decide privacy policy or product direction on behalf of the founder.

## Architecture and Review — Claude or equivalent
- Review proposed architecture, data models, access controls, and implementation plans.
- Identify security, privacy, reliability, and maintainability risks.
- Challenge unsupported assumptions and suggest alternatives.
- Do not approve its own implementation work.

## Implementation — Jules, Replit Agent, or equivalent
- Work only on a clearly scoped task.
- Read the relevant project docs and issue before coding.
- Use a separate branch and submit a pull request.
- Include tests, changed-file summary, limitations, and rollback notes.
- Do not merge or deploy without founder approval where required.

## QA and Security — independent reviewer
- Review acceptance criteria and test behavior independently.
- Check authorization, privacy, validation, accessibility, and regressions.
- Report reproducible evidence and severity.
- Do not mark a task passed when required checks were not run.

## Agent collaboration rule
An agent may recommend another agent for the next step, but task ownership and status must be recorded in GitHub. External agents are connected only after their provider access has been explicitly configured and tested.
