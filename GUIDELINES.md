# Community Guidelines

## Purpose of This Repository

This repository is a **community of programmable examples** for the Nokia Network Services Platform (NSP). It is intended for customers, engineers, partners, and anyone who wants to learn how to build NSP artifacts such as:

- **Workflows** (Mistral-based automation)
- **Intents** (intent types, design, and lifecycle)
- **Adaptors** and **Mappers**
- Other programmable artifacts supported by NSP

The goal is to provide clear, reusable, and well-documented examples that follow NSP best practices and align with the guidance published on the [Nokia Network Developer Portal](https://network.developer.nokia.com/). By contributing, you help others accelerate automation, improve developer experience, and adopt programmability on NSP.

---

## Maintenance and Ownership

- **Maintainers:** Research & Development (R&D) owns the maintenance of this repository.
- **Scope:** R&D is responsible for reviewing contributions, merging pull requests, releasing updates, and ensuring alignment with NSP releases and documentation.
- **Releases:** Content may be tagged or organized by NSP release where relevant; maintainers will keep examples compatible with supported NSP versions where possible.

---

## How to Contribute

We welcome contributions from the community. All changes should be submitted via **pull requests** (PRs).

### Contribution Process

1. **Fork** the repository and create a branch from the default branch (e.g. `main`).
2. **Create or update** content (new exercises, fixes, or improvements to existing READMEs/code).
3. **Follow** the [README template](Activity_template.md) for every new or substantially updated activity.
4. **Submit** a pull request with a clear title and description of what you changed and why.
5. **Respond** to review feedback from maintainers in a timely manner.
6. Once approved, a maintainer will merge your PR.

### What You Can Contribute

- New exercises or examples (workflows, intents, adaptors, mappers, etc.)
- Fixes to existing examples (bugs, clarity, compatibility)
- Improvements to documentation (READMEs, comments, diagrams)
- Corrections to guidelines or templates

---

## Best Practices and Rules for Contributing

### General Open Source Practices

- **Be respectful and constructive** in issues, pull requests, and discussions.
- **Write clear commit messages** and PR descriptions so reviewers can understand intent and scope.
- **Keep changes focused:** one logical change per PR when possible (e.g. one new exercise or one fix).
- **Do not submit** confidential, proprietary, or licensed material that you are not authorized to share.
- **Respect licensing:** ensure your contributions are compatible with the repository license (see [License](#license) below).

### Repository and Documentation Standards

- **One activity per folder:** each exercise or example should live in its own directory with a README and any required artifact files.
- **Use the README template:** every activity must include the sections defined in [Activity_template.md](Activity_template.md), including the summary table, introduction, pre-requisites, solution overview, and references.
- **Specify NSP version:** in the summary table, indicate the NSP release the example was created or validated for (e.g. NSP 24.8, 25.8).
- **Use inclusive language** and avoid jargon where a short explanation helps newcomers.

### NSP Artifact and Coding Best Practices

When implementing or documenting NSP artifacts, follow the practices and patterns described on the [Nokia Network Developer Portal](https://network.developer.nokia.com/). Examples:
  - Document the role of each artifact (e.g. northbound/southbound, data transformation).
  - Follow naming and structure conventions used in NSP documentation and existing examples in this repo.
  - Do not hardcode credentials or sensitive data; use placeholders, environment variables, or configuration references and document them in the README.
  - Validate examples against the NSP release stated in the README where possible.
  - Reference official NSP and Network Developer Portal documentation for APIs, schemas, and behaviour rather than duplicating or contradicting them.

---

## Pull Request Guidelines

- **Title:** Use a short, descriptive title (e.g. “Add workflow example: wait for Kafka event with nsp.sensor”).
- **Description:** Explain what changed, why, and how to verify (e.g. steps to run the example).
- **Checklist:** Confirm that the activity README follows [Activity_template.md](Activity_template.md) and that no sensitive data is included.
- **Size:** Prefer smaller, reviewable PRs; split large contributions into multiple PRs if it helps.

Maintainers may request changes or close PRs that do not meet these guidelines or that conflict with repository goals.

---

## Code of Conduct

- Treat everyone with respect and professionalism.
- Focus on constructive feedback and technical discussion.
- Do not harass, discriminate, or engage in behaviour that makes others feel unwelcome.

Violations may result in content removal or restrictions on participation, at the maintainers’ discretion.

---

## Reporting Issues

- Use **Issues** for bugs, unclear documentation, or suggestions (e.g. new example ideas).
- Provide context: NSP version, steps to reproduce, and expected vs. actual behaviour where relevant.
- For security-sensitive findings, follow the project’s or Nokia’s responsible disclosure process rather than posting publicly.

---

## License

Contributions are accepted under the license specified in the repository (see [LICENSE](LICENSE) or repository root). By submitting a PR, you agree that your contribution may be used under that license.

---

## References

- [Nokia Network Developer Portal](https://network.developer.nokia.com/) — NSP resources, APIs, and developer guidance
- [README / Activity Template](Activity_template.md) — Required structure for every activity in this repository
- NSP product and API documentation (Nokia Documentation Portal) — for release-specific details and intent/workflow reference
