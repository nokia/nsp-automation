# README Template

Use this template for every exercise or example in the repository. Copy the structure below into your activity’s `README.md` and fill in each section. You may add or remove subsections as needed (e.g. “Troubleshooting”, “Variations”) as long as the required elements are present.

---

## Summary Table

Place this table at the **top** of the README, before the main sections.


| Field                     | Description                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------ |
| **Title**                 | Short, descriptive name of the activity (e.g. “Wait for Kafka event with nsp.https_async”) |
| **Summary**               | One or two sentences describing what the example does.                                     |
| **Purpose**               | Why this example exists: what problem it illustrates or what skill it teaches.             |
| **Technologies Involved** | List (e.g. Mistral workflows, NSP REST API, Kafka, Python, YANG).                          |
| **NSP Release**           | NSP version this example was created or validated for (e.g. NSP 24.8, 25.8).               |


**Example:**


| Field                     | Description                                                                                                                 |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Wait for Kafka event with nsp.https_async                                                                                   |
| **Summary**               | A workflow that uses the `nsp.https_async` action to wait asynchronously for a Kafka event without sending an HTTP request. |
| **Purpose**               | Demonstrate event-driven workflow design and the async pattern for external or asynchronous notifications.                  |
| **Technologies Involved** | Mistral workflows, nsp.https_async action, Kafka.                                                                           |
| **NSP Release**           | NSP 24.8                                                                                                                    |


---

## Introduction

**Short description:** In one or two paragraphs, explain what this exercise or example is about and who it is for.

**Problem statement:** Clearly state the scenario or problem the example addresses (e.g. “Workflows often need to wait for an external event such as a device callback or completion notification before continuing. This example shows how to do that using a dedicated async action.”).

---

## Pre-requisites

List what the reader needs before starting:

- **NSP:** Version and any required features or licenses (e.g. workflow engine, intent framework).
- **Access and roles:** e.g. Developer mode enabled, Developer or Operator role.
- **External systems:** e.g. Kafka topic available, test device or simulator.
- **Tools or skills:** e.g. basic YAML, familiarity with NSP UI or CLI, optional: Postman or curl.
- **Other:** Any specific configuration, accounts, or lab setup.

Use bullet points or a short table for clarity.

---

## Solution Overview

Describe the solution at a high level, then provide a **step-by-step guide** so someone can reproduce the example.

- **High-level design:** Brief explanation of the approach (e.g. “The workflow has three tasks: trigger external process, wait for event with nsp.https_async, process result.”).
- **Step-by-step guide:**
  1. Step 1 (e.g. “Create or import the workflow definition”).
  2. Step 2 (e.g. “Configure the Kafka topic and timeout”).
  3. … continue as needed.

Include **code snippets** (workflow YAML, intent snippets, adapter config, etc.) with syntax highlighting. Keep snippets self-contained enough to follow along; for long files, you may reference a file in the same folder and show the relevant part.

Use **images** where they help (e.g. workflow diagram, UI screenshot, sequence diagram). Store images in the same activity folder and reference them with relative paths (e.g. `./workflow-overview.png`).

---

## Conclusion

Summarise what the reader should have learned or achieved (e.g. “You now have a workflow that waits for a Kafka event using nsp.https_async and can adapt the topic and timeout for your own use cases.”). Optionally mention possible next steps or related examples.

---

## References

List links and documents that support or extend this example:

- Official NSP or Network Developer Portal documentation (e.g. workflow guide, action reference).
- Relevant API or schema documentation.
- Related examples in this repository or other Nokia resources.

**Example:**

- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [Intent management framework](https://documentation.nokia.com/nsp/24-8/NSP_System_Architecture_Guide/Intent-management-framework.html)

---

## Optional Sections (use as needed)

- **Troubleshooting:** Common errors and how to resolve them.
- **Variations:** Alternative approaches or extensions (e.g. different actions, timeouts, or event sources).
- **Security and operations:** Notes on credentials, logging, or production considerations.
- **Changelog:** If you update the example for a new NSP release, note the change and version here.

