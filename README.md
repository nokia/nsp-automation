# NSP Automation Examples

This repository is a **curated collection of programmable examples** for the Nokia Network Services Platform (NSP). It is intended for engineers, customers, partners, and anyone who wants to learn how to build NSP artifacts—such as workflows, network intents, adaptors, mappers, and other supported programmability features—using clear, reusable patterns aligned with Nokia’s published guidance.

The goal is to accelerate adoption of NSP automation and developer experience by sharing examples that are documented consistently and kept compatible with supported NSP releases where possible.

---

## What You Will Find Here

- **Examples and exercises** organized by area (for example: workflows, network intents, adaptors, and related topics).
- **Documentation-first activities:** each example lives in its own folder with a README that explains purpose, prerequisites, and how to reproduce the solution.
- **Pointers to official resources:** the [Nokia Network Developer Portal](https://network.developer.nokia.com/) remains the primary source for APIs, product documentation, and platform behaviour; examples here illustrate and complement that content.

---

## Repository Layout

High-level structure (folders may evolve as content grows):


| Area                                 | Purpose                                                                                                                 |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| **Workflows**                        | Mistral-based automation and related examples                                                                           |
| **Network Intents** (or **Intents**) | Intent types, design, lifecycle, and related examples                                                                   |
| **Adaptors / Mappers / Others**      | Integration and mapping examples where applicable                                                                       |
| **Community**                        | Externally contributed examples that follow the same documentation and quality bar (see [GUIDELINES.md](GUIDELINES.md)) |


Exact folder names and nesting may vary; each area should remain easy to browse and should not duplicate the official product documentation—only illustrate it.

---

## Contributing and Community Guidelines

We welcome participation. **How to fork, open pull requests, and what we expect from contributions**—including scope, documentation standards, licensing, and conduct—is described in:

**[GUIDELINES.md](GUIDELINES.md)**

Please read that document before opening an issue or pull request.

---

## Activity README Template

Every exercise or example must include an activity `README.md` that follows a common structure (summary table, introduction, prerequisites, solution overview, references, and optional sections as needed). The canonical structure is defined in:

**[Activity_template.md](Activity_template.md)**

> **Note:** [GUIDELINES.md](GUIDELINES.md) refers to this template as `Activity_template.md`. Make sure to rename `Activity_template.md` to `README.md` at the root of your example / exercise.

---

## Where to Learn More


| Resource                                                               | Description                                                                  |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| [GUIDELINES.md](GUIDELINES.md)                                         | Maintenance model, contribution process, PR expectations, and best practices |
| [template_github.md](template_github.md)                               | Required sections for each activity README                                   |
| [Nokia Network Developer Portal](https://network.developer.nokia.com/) | Official NSP developer documentation and APIs                                |
| [LICENSE](LICENSE)                                                     | Terms under which content is made available (when present in the repository) |


---

## Issues and Feedback

Use **Issues** for bugs in examples, unclear documentation, or suggestions for new topics. For security-sensitive findings, follow responsible disclosure through the appropriate Nokia channel rather than posting sensitive details in public issues.

---

## License

Contributions are accepted under the license specified in this repository (see [LICENSE](LICENSE) when available). By submitting a pull request, you agree that your contribution may be used under that license and you accept the guidelines and rules defined in this repository.