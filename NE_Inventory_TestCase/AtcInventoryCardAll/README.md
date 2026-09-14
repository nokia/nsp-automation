## Summary Table

| Field                     | Description                                                                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Create Equipment Card List                                                                                                                  |
| **Summary**               | A Mistral workflow that retrieves all network cards via RESTCONF GET and renders the inventory as an HTML table.                            |
| **Purpose**               | Demonstrate how to query NSP equipment card inventory using RESTCONF and format the response into a readable HTML report with JavaScript.    |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF GET, YaQL, JavaScript (`std.js`), HTML.                                                     |
| **NSP Release**           | Tested in NSP 25.8.                                                                                |

---

## Introduction

**Short description:** This automation test case (ATC) workflow queries the NSP inventory for all hardware cards across managed network elements and produces an HTML report. It is intended for engineers learning how to combine RESTCONF northbound APIs with in-workflow JavaScript to build simple inventory dashboards.

**Problem statement:** Operations and lab teams often need a quick, human-readable view of installed cards (slot, part number, serial number, operational state) without writing a separate application. This example shows how a workflow can call the NSP RESTCONF equipment model and transform JSON results into a styled HTML table.

---

## Pre-requisites

- **NSP:** NSP 23.11 or later with Workflow Manager and RESTCONF gateway enabled.
- **Access and roles:** Developer mode enabled; Developer or Operator role with permission to execute workflows and call RESTCONF APIs.
- **External systems:** At least one managed network element with card inventory present in NSP.
- **Tools or skills:** Basic understanding of Mistral workflow YAML, RESTCONF, and optional familiarity with YaQL expressions.

---

## Solution Overview

- **High-level design:** The workflow has two tasks. `getCardInfo` performs a RESTCONF GET against the card hardware-component endpoint and selects relevant fields with YaQL. `displayHTMLTable` uses `std.js` to build an HTML page with a summary header and a data table. The final HTML is returned as workflow output.

- **Step-by-step guide:**
  1. Import the workflow definition from `AtcInventoryCardAll.yaml` (or `AtcInventoryCardAll.json`) into Workflow Manager.
  2. Ensure the RESTCONF equipment model is available and network elements are synchronized.
  3. Execute the `AtcInventoryCardAll` workflow.
  4. Open the workflow output (`result`) in a browser or viewer to inspect the generated HTML card list.

**Key RESTCONF call:**

```yaml
getCardInfo:
  action: nsp.https
  input:
    url: https://<% locate_nsp() %>:8545/restconf/data/nsp-equipment:network/network-element/hardware-component/card?depth=2
    method: GET
```

**Output columns:** NE Name, NE ID, Slot, Card, Operational State, Manufacturing Date, Part Number, Serial Number, Manufacturing Assembly Number.

---

## Conclusion

After running this workflow, you have an HTML inventory report of all cards in the NSP network. You can adapt the YaQL `select` expression or the JavaScript table builder to include additional card attributes or export formats.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples in this repository: `AtcInventoryCardUp`, `AtcTestSuite-InventoryReports`

---

## Security and Operations

This workflow is intended for **POC or demo use only** and is not recommended for production execution. Do not hardcode credentials; the workflow relies on NSP platform authentication. Copyright (c) 2024, Nokia.
