## Summary Table

| Field                     | Description                                                                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Title**                 | Create Service List with All E-Line Services                                                                                                     |
| **Summary**               | A Mistral workflow that queries E-Line services via RESTCONF POST (`nsp-inventory:find`) and renders service name, description, and states in HTML. |
| **Purpose**               | Demonstrate service-layer inventory reporting for E-Line services, including sequential RESTCONF calls and JavaScript HTML formatting.           |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF POST, `nsp-inventory:find`, YaQL, JavaScript (`std.js`), HTML, NSP service model.              |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                            |

---

## Introduction

**Short description:** This ATC workflow retrieves E-Line service inventory from the NSP service model and produces an HTML table with service name, description, NE service ID, and administrative/operational states. It performs two sequential inventory find calls—one for deployer details and one for E-Line service attributes.

**Problem statement:** Service inventory is as important as equipment inventory for automation validation. This example shows how to query the `nsp-service` E-Line model using XPath filters and field projection, then format results for human-readable review or automated test reporting.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager, RESTCONF gateway, and service inventory enabled.
- **Access and roles:** Developer mode enabled; Developer or Operator role with access to service data.
- **External systems:** E-Line services deployed and visible in NSP service inventory.
- **Tools or skills:** Mistral workflows, RESTCONF POST, XPath filters on service models.

---

## Solution Overview

- **High-level design:** Two workflow variables define find requests—`HTTPbody1` for deployer info and `HTTPbody2` for E-Line service fields. `getServInfo1` and `getServInfo2` run sequentially; the second call publishes test data. `displayHTMLTable` renders the HTML report.

- **Step-by-step guide:**
  1. Import `AtcInventoryServElineAll.yaml` (or `.json`) into Workflow Manager.
  2. Ensure E-Line services exist in the NSP service inventory.
  3. Execute the `AtcInventoryServElineAll` workflow.
  4. Review the HTML output for E-Line service states.

**Primary service query:**

```yaml
HTTPbody2:
  input:
    xpath-filter: /nsp-service:services/service-layer/eline
    fields: name;description;ne-service-id;admin-state;oper-state
    include-meta: false
    depth: 2
    sort-by: [name]
```

**Output columns:** Name, Description, NE Service ID, Admin State, Operational State.

---

## Conclusion

You now have a workflow that reports E-Line service inventory using RESTCONF find operations. Extend the XPath filters or add deployer-state columns from the first query for richer service lifecycle reporting.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcTestSuite-InventoryReports`

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2024, Nokia.