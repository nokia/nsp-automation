## Summary Table

| Field                     | Description                                                                                                                                                 |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Create Equipment Transceiver (SFP) List                                                                                                                     |
| **Summary**               | A Mistral workflow that retrieves transceiver details for equipped SFPs via RESTCONF GET and renders optical and connector attributes in an HTML table.     |
| **Purpose**               | Demonstrate transceiver inventory reporting, YaQL ordering by vendor OUI, and JavaScript parsing of model identifiers for SFP position display.             |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF GET, YaQL, JavaScript (`std.js`), HTML, regular expressions.                                                |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                                       |

---

## Introduction

**Short description:** This ATC workflow queries transceiver (SFP) details from the NSP equipment model and produces a styled HTML inventory report. Results are ordered by vendor OUI, and SFP position is extracted from the model identifier using a regular expression.

**Problem statement:** Optical transceiver inventory is critical for capacity planning and troubleshooting. This example shows how to retrieve `transceiver-details` via RESTCONF, filter null connector entries, and present optical compliance, wavelength, and slot position in a readable table.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager and RESTCONF on port `8545`.
- **Access and roles:** Developer mode enabled; Developer or Operator role.
- **External systems:** Network elements with equipped transceivers reported in NSP inventory.
- **Tools or skills:** Mistral workflows, RESTCONF, YaQL, basic JavaScript.

---

## Solution Overview

- **High-level design:** `getSFPInfo` performs a RESTCONF GET on the transceiver-details endpoint and orders results by vendor OUI. `displayHTMLTable` skips rows with null connector codes and uses regex to parse NE ID and SFP position from the `nsp-model:identifier` field.

- **Step-by-step guide:**
  1. Import `AtcInventorySfpAll.yaml` (or `.json`) into Workflow Manager.
  2. Verify transceiver details are available for ports with equipped optics.
  3. Execute the `AtcInventorySfpAll` workflow.
  4. Review the HTML output for transceiver optical and connector attributes.

**Key RESTCONF call:**

```yaml
getSFPInfo:
  action: nsp.https
  input:
    url: https://<% locate_nsp() %>:8545/restconf/data/nsp-equipment:network/network-element/hardware-component/port/transceiver-details?limit=100
    method: GET
```

**Output columns:** Connector Code, Vendor OUI, Laser Wavelength, Optical Compliance, Specific Type, Number of Lanes, Connector Type, Link Length Support, SFP Position.

---

## Conclusion

This workflow provides a transceiver inventory HTML report with vendor-ordered results and parsed SFP positions. Increase the `limit` parameter or extend the YaQL selection for larger networks.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcInventoryPortUp`, `AtcTestSuite-InventoryReports`

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2022, Nokia.