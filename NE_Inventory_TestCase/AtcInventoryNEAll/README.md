## Summary Table

| Field                     | Description                                                                                                                                                  |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Title**                 | Create NE Equipment List                                                                                                                                     |
| **Summary**               | A Mistral workflow that retrieves all network elements via RESTCONF GET and generates an HTML report with Google Maps links and a downloadable HTML file.    |
| **Purpose**               | Demonstrate NE inventory reporting, geo-location linking, URL encoding for downloadable reports, and multi-task HTML assembly using JavaScript and Jinja2.   |
| **Technologies Involved** | Mistral workflows, `nsp.https` action, RESTCONF GET, YaQL, JavaScript (`std.js`), Jinja2, HTML, Google Maps links.                                           |
| **NSP Release**           | Tested in NSP 25.8.                                                                                                                        |

---

## Introduction

**Short description:** This ATC workflow builds a comprehensive network element (NE) inventory report in HTML format. When latitude and longitude are configured on an NE, the report includes clickable Google Maps links for the location field. A downloadable HTML attachment is also generated.

**Problem statement:** Network operators need a single view of managed NEs including management IP, product, version, and operational states. This example shows how to query the full NE list, enrich location data with map links, and provide both inline and downloadable report output.

---

## Pre-requisites

- **NSP:** NSP R23.11 or later with Workflow Manager and RESTCONF on port `8545`.
- **Access and roles:** Developer mode enabled; Developer or Operator role.
- **External systems:** Managed network elements registered in NSP; optional geo coordinates configured on NEs for map links.
- **Tools or skills:** Mistral workflows, RESTCONF, basic HTML and JavaScript.

---

## Solution Overview

- **High-level design:** `getNeInfo` retrieves all network elements with depth 2. `createHTMLBegin` uses JavaScript to build the HTML table and Google Maps anchor tags. `createHTMLstream` URL-encodes the HTML for download. `displayHTMLtable` assembles the final report with a download link.

- **Step-by-step guide:**
  1. Import `AtcInventoryNEAll.yaml` (or `.json`) into Workflow Manager.
  2. Ensure NEs are managed and synchronized in NSP.
  3. Optionally configure location, latitude, and longitude on NEs for map integration.
  4. Execute the `AtcInventoryNEAll` workflow.
  5. View the HTML output and use the **AtcInventoryNEAll.html** download link if needed.

**Key RESTCONF call:**

```yaml
getNeInfo:
  action: nsp.https
  input:
    url: https://<% locate_nsp() %>:8545/restconf/data/nsp-equipment:network/network-element?depth=2
    method: GET
```

**Output columns:** Name, Management IP, NE ID, Type, Product, Version, Managed State, Communication State, Operational State, Resync State, Location (with optional Google Maps link).

---

## Conclusion

You now have an NE inventory workflow that combines RESTCONF data retrieval, geo-location linking, and downloadable HTML reporting. Extend the YaQL selection or HTML template to add further NE attributes.

---

## References

- [Automate NSP Inventory Export Using WorkFlows](https://network.developer.nokia.com/tutorials/automate-inventory-export-using-workflows/)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- Related examples: `AtcTestSuite-InventoryReports`

---

## Troubleshooting

- **Google Maps links do not work:** Verify that `latitude` and `longitude` are configured on the NE in NSP.
- **Empty report:** Confirm network elements exist and RESTCONF equipment data is accessible.

---

## Security and Operations

This workflow is intended for **POC or demo use only**. Copyright (c) 2024, Nokia.