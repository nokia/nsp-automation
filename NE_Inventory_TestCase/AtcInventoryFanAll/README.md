## Summary Table

| Field                     | Description                                                                                                                                         |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Create Equipment FAN List                                                                                                                           |
| **Summary**               | A Mistral workflow that retrieves fan hardware inventory via RESTCONF GET and renders an HTML table, using `resultFilter` to limit logged data.   |
| **Purpose**               | Demonstrate RESTCONF GET for fan components and the use of `resultFilter` on `nsp.https` to reduce workflow log volume while building HTML reports. |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF GET, `resultFilter`, YaQL, JavaScript (`std.js`), HTML.                                           |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                               |

---

## Introduction

**Short description:** This ATC workflow queries fan hardware components across the network and produces an HTML inventory report. It uses the Kubernetes `restconf-gateway` service endpoint and applies a `resultFilter` to keep only the fields needed for the report.

**Problem statement:** Large RESTCONF responses can inflate Workflow Manager logs and slow troubleshooting. This example shows how to query fan inventory with a result limit and filter the HTTP action output before publishing test data for HTML rendering.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager and RESTCONF gateway deployed.
- **Access and roles:** Developer mode enabled; Developer or Operator role.
- **External systems:** Network elements with fan hardware inventory present in NSP.
- **Tools or skills:** Basic Mistral workflow and RESTCONF knowledge.

---

## Solution Overview

- **High-level design:** `getCardInfo` performs a RESTCONF GET with `limit=1000` and a `resultFilter` that selects NE name, NE ID, component ID, name, and operational state. `displayHTMLTable` builds the HTML output.

- **Step-by-step guide:**
  1. Import `AtcInventoryFanAll.yaml` (or `.json`) into Workflow Manager.
  2. Confirm fan inventory is available under the NSP equipment model.
  3. Execute the `AtcInventoryFanAll` workflow.
  4. Review the HTML output for fan operational status across NEs.

**Key configuration:**

```yaml
getCardInfo:
  action: nsp.https
  input:
    url: https://restconf-gateway/restconf/data/nsp-equipment:network/network-element/hardware-component/fan?limit=1000
    resultFilter: $.content["nsp-equipment:fan"].select([$.get("ne-name"), $.get("ne-id"), $.get("component-id"), $.name, $.get("oper-state")])
    method: GET
```

**Output columns:** NE Name, NE ID, Slot, Fan Name, Operational State.

---

## Conclusion

This workflow provides a concise fan inventory HTML report while minimizing logged payload size through `resultFilter`. You can reuse the same pattern for other hardware component types.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcInventoryPwrAll`, `AtcTestSuite-InventoryReports`

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2024, Nokia.