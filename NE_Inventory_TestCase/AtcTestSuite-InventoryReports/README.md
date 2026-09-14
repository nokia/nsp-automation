## Summary Table

| Field                     | Description                                                                                                                                                              |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Title**                 | NSP Inventory Reports – Automation Test Batch                                                                                                                            |
| **Summary**               | A batch Mistral workflow that sequentially executes inventory report workflows and generates a consolidated HTML test report with action-execution troubleshooting links. |
| **Purpose**               | Demonstrate using Workflow Manager as a simple automated test runner for inventory workflows, with aggregated pass/fail visibility and downloadable HTML test results.     |
| **Technologies Involved** | Mistral workflows, sub-workflow execution, Workflow Manager REST API, `nsp.https`, Jinja2, YaQL, HTML.                                                                     |
| **NSP Release**           | Tested in NSP 25.8.                                                                                      |

---

## Introduction

**Short description:** This ATC batch workflow runs a suite of inventory report workflows in sequence, then queries the Workflow Manager action-execution API to build a consolidated HTML test report. Each sub-workflow returns results via its output variable, and the batch report includes links to individual action executions for troubleshooting.

**Problem statement:** Validating multiple inventory workflows manually is time-consuming. This example experiments with Workflow Manager as a lightweight test harness—executing a defined test list, collecting action execution metadata, and producing a single HTML report with execution state and timestamps.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager; validated on R23.11, R24.8, and R24.11.
- **Access and roles:** Developer mode enabled; Developer role recommended for workflow import and batch execution.
- **External systems:** All sub-workflows in the test list must be imported into Workflow Manager before running the batch.
- **Tools or skills:** Mistral workflows, sub-workflow invocation, Workflow Manager UI for reviewing action executions.

---

## Solution Overview

- **High-level design:** The batch workflow records a start timestamp, invokes each inventory workflow in sequence with a short `wait-after` delay, then calls the Workflow Manager API to retrieve action executions created after the batch start time. Jinja2 templates build an HTML report with test titles, task names, states, and links to action execution details. A downloadable `AtcTestSuite.html` file is included.

- **Step-by-step guide:**
  1. Import all individual inventory workflows listed above into Workflow Manager.
  2. Import `AtcTestSuite-InventoryReports.yaml` (or `.json`).
  3. Execute the `AtcTestSuite-InventoryReports` workflow.
  4. Review the consolidated HTML report in the workflow output.
  5. Use the action execution links in the report to drill into inputs and outputs of individual tasks.
  6. Download **AtcTestSuite.html** from the report link if an offline copy is needed.

**Test sequence (sub-workflows):**

| Order | Workflow                 | Test Title                              |
| ----- | ------------------------ | --------------------------------------- |
| 1     | `AtcInventoryNEAll`      | Create an NE inventory Table            |
| 2     | `AtcInventoryCardAll`    | Create a Card Table                     |
| 3     | `AtcInventoryCardUp`     | Create a Card Operational State Up Table |
| 4     | `AtcInventoryPwrAll`     | Create a Power Supply Table             |
| 5     | `AtcInventoryFanAll`     | Create a FAN Table                      |
| 6     | `AtcInventorySfpAll`     | Create a Transceiver Table              |
| 7     | `AtcInventoryPortUp`     | Create a Port Up Table                  |
| 8     | `AtcInventoryServElineAll` | Create a table with all E-Line Services |

**Action execution query:**

```yaml
getActExec:
  action: nsp.https
  input:
    url: https://workflow-manager/wfm/api/v1/action-execution?created_at=gt:<% url_encode($.testBatchStartTime) %>
```

---

## Conclusion

You now have a batch workflow that automates execution and reporting of inventory test cases in Workflow Manager. Extend `testList` to add workflows (e.g. `AtcInventoryPortAll`) or integrate additional assertions based on action execution state.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Individual inventory examples in this folder: `AtcInventoryNEAll`, `AtcInventoryCardAll`, and others listed above.

---

## Troubleshooting

- **Sub-workflow not found:** Ensure every workflow referenced in the batch is imported with matching workflow names.
- **Empty test report:** Verify the batch start time filter and that sub-workflows completed after `testBatchStartTime`.
- **Action execution links:** Use the Workflow Manager UI link format `/web/workflow-manager/action-executions/info?actionExecutionId=<id>` embedded in the report.

---

## Changelog

- **R24.11:** Regression issue fixed (validated 10/04/2025).
- **Version 1.0.1:** Batch report includes downloadable HTML and action execution troubleshooting links.

---

## Security and Operations

This workflow is intended for **POC or demo use only** and is **not intended for production execution**. Copyright (c) 2025, Nokia.