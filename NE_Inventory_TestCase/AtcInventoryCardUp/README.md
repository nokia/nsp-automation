## Summary Table

| Field                     | Description                                                                                                                                              |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Create Equipment Card List (Operational Up)                                                                                                              |
| **Summary**               | A Mistral workflow that uses RESTCONF POST with an XPath filter to list only cards in `enabled` operational state and renders them as an HTML table.      |
| **Purpose**               | Demonstrate filtered inventory queries with `nsp-inventory:find`, field selection, sorting, and HTML report generation for operational card monitoring.  |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF POST, `nsp-inventory:find`, YaQL, JavaScript (`std.js`), HTML.                                          |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                                    |

---

## Introduction

**Short description:** This ATC workflow retrieves only cards that are operationally up (`oper-state='enabled'`) and formats the result as an HTML inventory table. It illustrates the POST-based inventory find operation, which is more efficient than retrieving all cards when filtering is required.

**Problem statement:** Full card inventory reports can be large and include decommissioned or failed hardware. This example shows how to apply an XPath filter and field projection at query time, then present a focused operational-up card list for troubleshooting or health checks.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager and RESTCONF gateway (`restconf-gateway` service) available.
- **Access and roles:** Developer mode enabled; Developer or Operator role with workflow execution and RESTCONF access.
- **External systems:** Managed network elements with card inventory synchronized in NSP.
- **Tools or skills:** Familiarity with Mistral workflows, RESTCONF POST operations, and XPath filters.

---

## Solution Overview

- **High-level design:** Workflow variables define the `nsp-inventory:find` request body with an XPath filter, field list, sort order, and depth. `getCardInfo` executes the POST call. `displayHTMLTable` transforms the filtered JSON into HTML.

- **Step-by-step guide:**
  1. Import `AtcInventoryCardUp.yaml` (or `.json`) into Workflow Manager.
  2. Verify the `nsp-inventory:find` operation is available on your NSP release.
  3. Execute the `AtcInventoryCardUp` workflow.
  4. Review the workflow output HTML for cards with operational state `enabled`.

**Inventory find request body:**

```yaml
HTTPbody1:
  input:
    xpath-filter: /nsp-equipment:network/network-element/hardware-component/card[oper-state='enabled']
    fields: card;ne-name;ne-id;component-id;name;oper-state;mfg-date;part-number;serial-num;mfg-assembly-number
    sort-by: [ne-id, component-id]
    depth: 2
```

**Output columns:** NE Name, NE ID, Slot, Card, Operational State, Manufacturing Date, Part Number, Serial Number, Manufacturing Assembly Number.

---

## Conclusion

You now have a workflow that reports only operationally up cards using filtered RESTCONF inventory queries. Adapt the XPath filter or field list for other operational states or card attributes.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcInventoryCardAll`, `AtcTestSuite-InventoryReports`

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2024, Nokia.