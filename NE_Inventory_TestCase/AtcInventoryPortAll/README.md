## Summary Table

| Field                     | Description                                                                                                                                        |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Create Port List                                                                                                                                   |
| **Summary**               | A Mistral workflow that retrieves all hardware ports via RESTCONF GET (up to 1000 items) and renders admin and operational state in an HTML table. |
| **Purpose**               | Demonstrate port inventory reporting with `resultFilter` to limit response fields and reduce Workflow Manager log payload.                           |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF GET, `resultFilter`, YaQL, JavaScript (`std.js`), HTML.                                          |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                              |

---

## Introduction

**Short description:** This ATC workflow queries port hardware components across all managed network elements and produces an HTML inventory table showing administrative and operational states. It uses the RESTCONF gateway and a `resultFilter` for efficient data handling.

**Problem statement:** Port inventories can be large (default RESTCONF limits may truncate results). This example raises the query limit to 1000 and filters the response to only the fields required for a concise port status report.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager and RESTCONF gateway.
- **Access and roles:** Developer mode enabled; Developer or Operator role.
- **External systems:** Network elements with port inventory synchronized in NSP.
- **Tools or skills:** Mistral workflows and RESTCONF fundamentals.

---

## Solution Overview

- **High-level design:** `getPortInfo` performs a RESTCONF GET with `limit=1000` and filters to NE name, NE ID, component ID, admin state, and operational state. `displayHTMLTable` renders the HTML report.

- **Step-by-step guide:**
  1. Import `AtcInventoryPortAll.yaml` (or `.json`) into Workflow Manager.
  2. Verify port data is available in the NSP equipment model.
  3. Execute the `AtcInventoryPortAll` workflow.
  4. Review the HTML output for port admin and operational states.

**Key configuration:**

```yaml
getPortInfo:
  action: nsp.https
  input:
    url: https://restconf-gateway/restconf/data/nsp-equipment:network/network-element/hardware-component/port?limit=1000
    resultFilter: $.content["nsp-equipment:port"].select([$.get("ne-name"), $.get("ne-id"), $.get("component-id"), $.get("admin-state"), $.get("oper-state")])
    method: GET
```

**Output columns:** NE Name, NE ID, Slot, Admin State, Operational State.

---

## Conclusion

This workflow provides a full port inventory HTML report with efficient field filtering. For ports that are operationally up with extended details, see the related `AtcInventoryPortUp` example.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcInventoryPortUp`, `AtcTestSuite-InventoryReports`

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2022, Nokia.