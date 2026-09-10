# 7x50 CF CleanUp

## Summary Table

| Field                     | Description                                                                                                                                                             |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | 7x50 CF CleanUp (`cleanupCFlash`)                                                                                                                                       |
| **Summary**               | A workflow that removes files older than a configurable age from an SR OS network element compact flash directory, with dry-run support.                                |
| **Purpose**               | Demonstrate NE filesystem cleanup using Network Supervision lookups, managed CLI (MD and Classic), TextFSM parsing, and Python age calculation.                         |
| **Technologies Involved** | Mistral workflows, `nsp.https`, `nsp.managed_cli`, `nsp.textFSM`, `nsp.python`, schemaForm / Input Forms, Network Supervision REST API.                                 |
| **NSP Release**           | NSP 25.8                                                                                                                                                                |

---

## Introduction

**Short description:** This example provides a proof-of-concept Mistral workflow that cleans old files from the compact flash (CF) of an NSP-managed SR OS device. It supports both model-driven (MD) and Classic (NFM-P) managed devices and is intended for customers, partners, and engineers learning Workflow Manager patterns for NE maintenance.

**Problem statement:** Activity or logging directories on SR OS compact flash can fill with aged files. Operators need a safe, repeatable way to identify deletion candidates (with dry-run support), skip critical system files, and apply the correct CLI delete/remove commands depending on whether the NE is MD or Classic managed. This example shows how to do that with a single workflow.

---

## Pre-requisites

- **NSP:** NSP 25.8 with Workflow Manager (WFM) enabled.
- **Access and roles:** Developer mode enabled; ability to create, publish, and execute workflows (Developer or Operator role as required by your environment).
- **External systems:** At least one SR OS NE managed by NSP (MD via MDM and/or Classic via NFM-P) with a target CF directory such as `cf3:/act`.
- **Tools or skills:** Basic YAML and JSON; familiarity with the NSP Workflows UI or WFM REST API; optional: VS Code with the NSP workflows plugin, Postman or curl.
- **Other:** Lab or non-production NE recommended. Do not hardcode credentials; use NSP session authentication and placeholders in examples.

---

## Solution Overview

- **High-level design:** The `cleanupCFlash` workflow has these tasks: resolve the NE with `nsp.https` (`getNeIP`); list files with `nsp.managed_cli` on MD or Classic paths; parse the listing with `nsp.textFSM`; compute ages and build delete commands with `nsp.python` (honouring `dryRun` and an exception list); delete candidates when not dry-running; then close the CLI session.

- **Step-by-step guide:**

  1. **Create or import the workflow definition.** Use the VS Code NSP workflows plugin or the NSP UI (**Workflows** → **+ Workflow**). Paste or import [`7x50-CF-cleanup-workflow.yaml`](./7x50-CF-cleanup-workflow.yaml), then **Validate & Update Flow** and **Create**.

  2. **(Optional) Add a short README in the NSP UI.** From the workflow **⋮** menu choose **View info**, edit the **Readme** tab, and describe inputs (`neName`, `dir`, `deleteAge`) and `dryRun` support.

  3. **Publish the workflow.** On the workflow info page, use **Modify state** to set it to **Published**.

  4. **(Optional) Add an Input Form (schemaForm).** In the workflow **Input Form** section, enter:

     ```yaml
     type: object
     properties:
       - name: neName
         title: NE Name
         columnSpan: 2
         newRow: true
         readOnly: false
         required: true
         type: string
         component:
           input: autoComplete
         suggest:
           action: nspWebUI.neList
           name:
             - neName
       - name: dir
         title: DIR
         description: Target directory
         columnSpan: 4
         newRow: true
         readOnly: false
         required: false
         type: string
         default: cf3:/act
         validations:
           patterns: []
       - name: deleteAge
         title: DELETE AGE
         description: Minimum age of file in seconds
         columnSpan: 4
         newRow: true
         readOnly: false
         required: false
         type: number
         default: 3600
     ```

  5. **Execute the workflow.** Supply input such as:

     ```json
     {
       "neName": "<NE_NAME>",
       "dir": "cf3:/act",
       "deleteAge": 3600
     }
     ```

     Start with `dryRun: true`. From the UI use the Input Form, or call the WFM REST API:

     ```json
     POST https://<NSP_IP>/wfm/api/v1/execution
     content-type: application/json

     {
         "workflow_id": "cleanupCFlash",
         "input": {
             "neName": "<NE_NAME>",
             "dir": "cf3:/act",
             "deleteAge": 3600
         },
         "params": {
             "env": "DefaultEnv",
             "options": {
                 "dryRun": true,
                 "force": false,
                 "notifyKafka": true
             }
         },
         "output": {},
         "notifyKafka": true
     }
     ```

     Example successful output:

     ```yaml
     files:
       - NAME: four.txt
         SIZE: '5'
         TIMEDATE: 10/23/2025  03:50p
       - NAME: three.txt
         SIZE: '5'
         TIMEDATE: 10/23/2025  03:01p
     success: true
     ```

     Workflow inputs: `neName` (required), `dir` (default `cf3:/act`), `deleteAge` in seconds (default `3600`), plus the standard WFM `dryRun` option. Critical filenames in the Python `exceptList` (for example `BOF.CFG`, `CONFIG.CFG`, `NVRAM.DAT`, and selected TIM/boot images) are never deleted—review that list before any non–dry-run use.

---

## Conclusion

You now have a workflow that cleans aged files from SR OS compact flash using `cleanupCFlash`, with MD and Classic paths, dry-run preview, and an optional schemaForm Input Form. Adapt the directory, age threshold, and exception list for your own lab or OSS use cases.

---

## References

- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [Workflow artifact in this folder](./7x50-CF-cleanup-workflow.yaml)
- [Contribution template](../../template/Contribution_template.md)
- [Community Guidelines](../../GUIDELINES.md)

---

## Security and operations

This tutorial is a proof of concept and an example method for cleaning compact flash on an NSP-managed SR OS device. It is not designed to be implemented as-is in a production network. Prefer dry-run first, validate against your NSP release, review exception lists and target directories, and never commit credentials or environment-specific secrets into the repository.
