## Summary Table

| Field                     | Description                                                                                                                                                          |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Create Equipment List with All Ports Operational Up                                                                                                                  |
| **Summary**               | A Mistral workflow that uses RESTCONF POST with an XPath filter to list ports in `enabled` operational state and renders detailed port information as an HTML table. |
| **Purpose**               | Demonstrate advanced filtered port inventory queries including port details (type, mode, MTU, rate) and conditional rendering when port details are absent (e.g. tunnels). |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF POST, `nsp-inventory:find`, YaQL, JavaScript (`std.js`), HTML.                                                     |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                                                |

---

## Introduction

**Short description:** This ATC workflow retrieves ports that are operationally up and builds a detailed HTML report including description, part/serial numbers, manufacturing date, and port details such as type, mode, MTU, and rate. It handles cases where `port-details` may be undefined.

**Problem statement:** Operational port health checks often require more than admin/oper states—engineers need rate, duplex/mode, and hardware identifiers. This example shows how to filter for enabled ports and enrich the report with nested `port-details` when available.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager and RESTCONF gateway.
- **Access and roles:** Developer mode enabled; Developer or Operator role.
- **External systems:** Network elements with port inventory synchronized in NSP.
- **Tools or skills:** Mistral workflows, RESTCONF POST, XPath filters, JavaScript.

---

## Solution Overview

- **High-level design:** Workflow variables define the `nsp-inventory:find` POST body with an XPath filter for `oper-state='enabled'`. `getPortInfo` executes the query. `displayHTMLTable` iterates results in JavaScript, conditionally including `port-details` fields and cleaning `null` values from the HTML.

- **Step-by-step guide:**
  1. Import `AtcInventoryPortUp.yaml` (or `.json`) into Workflow Manager.
  2. Verify the `nsp-inventory:find` operation is available.
  3. Execute the `AtcInventoryPortUp` workflow.
  4. Review the HTML table for operationally up ports and their detailed attributes.

**Inventory find request body:**

```yaml
HTTPbody1:
  input:
    xpath-filter: /nsp-equipment:network/network-element/hardware-component/port[oper-state='enabled']
    fields: ne-name;ne-id;component-id;oper-state;admin-state;port-details;part-number;serial-num;mfg-date;description;name
    include-meta: false
    sort-by: [ne-id, component-id]
    depth: 2
    limit: 1000
```

**Output columns:** NE Name, NE ID, Slot, Description, Admin State, Operational State, Part Number, Serial Number, Manufacturing Date, Port Type, Port Mode, MTU, Rate, Rate Units.

---

## Conclusion

You now have a workflow that reports operationally up ports with extended hardware and port-detail attributes. Adapt the XPath filter or JavaScript rendering for other port states or additional nested fields.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcInventoryPortAll`, `AtcTestSuite-InventoryReports`

---

## Troubleshooting

- **Missing port detail columns:** Some port types (e.g. tunnels) do not expose `port-details`; the workflow leaves those cells blank.
- **`null` in output:** The workflow replaces `null` strings with spaces in the final HTML.

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2022, Nokia.