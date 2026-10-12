# hve-center-kit

A reusable HVE Center starter kit for Enterprise Systems Integrators (ESIs) and
Software Development Companies (SDCs), intended to support evidence-backed
discovery, product decisions, and engineering handoffs.

## Current status

This repository is a planning baseline, not a working demo. The implementation
roadmap is in [the HVE Center implementation plan](plan.md). Agents, artifact
templates, approval gates, and the static HTML workspace described there have
not been installed or implemented in this repository yet.

## Direction

- Use GitHub as the system of record for context, evidence, decisions, and handoffs.
- Reuse HVE Core agents and skills rather than rebuilding their capabilities.
- Add a read-mostly HTML workspace for workshop visibility.
- Require human review and explicit confirmation before publishing backlog items.
- Extend the kit over time with reusable ESI/SDC practices and workshop guidance.

Start with the demo scope in `plan.md`, validate it with partners, and choose a
scaling approach only after that validation. Keep the existing Decision Room
workshop separate and unchanged.

## Reuse as a template

Use GitHub's **Use this template** action to create an independent repository,
then adapt the plan to your organization, roles, and sample scenario.

This repository is public. Publish only synthetic examples or material approved
for public sharing. Keep customer data, internal documents, and credentials in
appropriately restricted repositories; do not copy them into this template.

## Contributions

Propose scoped changes through pull requests. Distinguish implemented capabilities
from proposed practices and keep the roadmap aligned with the actual repository.

Initial repository documentation was prepared with GitHub Copilot assistance.
