## Summary Table

| Field                     | Description                                                                                                                                    |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Create Power Supply Inventory List                                                                                                             |
| **Summary**               | A Mistral workflow that retrieves all power supply hardware via RESTCONF GET and renders NE, shelf, name, and operational state in HTML.      |
| **Purpose**               | Demonstrate power supply inventory reporting using RESTCONF GET with `resultFilter` for efficient log handling and HTML table generation.      |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF GET, `resultFilter`, YaQL, JavaScript (`std.js`), HTML.                                        |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                          |

---

## Introduction

**Short description:** This ATC workflow queries power supply components across the network and produces an HTML inventory report. It follows the same RESTCONF GET and `resultFilter` pattern used for fan and port inventory examples.

**Problem statement:** Power supply health is a common operational check in lab and demo environments. This example shows how to retrieve power supply inventory from the NSP equipment model and present it in a simple HTML table.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager and RESTCONF gateway.
- **Access and roles:** Developer mode enabled; Developer or Operator role.
- **External systems:** Network elements with power supply hardware inventory in NSP.
- **Tools or skills:** Basic Mistral workflow and RESTCONF knowledge.

---

## Solution Overview

- **High-level design:** `getCardInfo` performs a RESTCONF GET with `limit=1000` and a `resultFilter` selecting NE name, NE ID, component ID, name, and operational state. `displayHTMLTable` builds the HTML output.

- **Step-by-step guide:**
  1. Import `AtcInventoryPwrAll.yaml` (or `.json`) into Workflow Manager.
  2. Confirm power supply inventory is available under the equipment model.
  3. Execute the `AtcInventoryPwrAll` workflow.
  4. Review the HTML output for power supply operational status.

**Key configuration:**

```yaml
getCardInfo:
  action: nsp.https
  input:
    url: https://restconf-gateway/restconf/data/nsp-equipment:network/network-element/hardware-component/power-supply?limit=1000
    method: GET
    resultFilter: $.content["nsp-equipment:power-supply"].select([$.get("ne-name"), $.get("ne-id"), $.get("component-id"), $.get("name"), $.get("oper-state")])
```

**Output columns:** NE Name, NE ID, Shelf, Power Supply Name, Operational State.

---

## Conclusion

This workflow provides a power supply inventory HTML report using filtered RESTCONF GET responses. Reuse the pattern for other hardware component types in the `nsp-equipment` model.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcInventoryFanAll`, `AtcTestSuite-InventoryReports`

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2024, Nokia.