# Bulk E-Line Provisioning using Workflows

---

## Summary Table

| Field                     | Description                                                                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Bulk E-Line provisioning using Workflows                                                                                                    |
| **Summary**               | A Mistral workflow that bulk-creates E-Line (epipe) services via the Service Management intent API, measures provisioning duration, and returns an HTML service-state report. |
| **Purpose**               | Demonstrate how Workflows automate repetitive service provisioning: resolve template and customer metadata, create services in parallel, and present results in a user-friendly format. |
| **Technologies Involved** | Mistral workflows, `nsp.https`, `nsp.python`, `std.noop`, NSP RESTCONF (`nsp-inventory:find`), Service Management intent API, Jinja2 (`serviceState` template). |
| **NSP Release**           | NSP 25.8 (per `workflow_meta.dependencies.platform.nspOS`).                                                                                 |

---

## Introduction

**Short description:** This activity walks through the `bulkEline` workflow, which provisions multiple E-Line services from a single execution. The workflow looks up the service template and customer IDs, posts intent payloads for each target service, captures execution timing from the Workflow Manager API, and renders a formatted HTML summary of each service's SLC state.

**Problem statement:** Manually provisioning many E-Line services through the NSP UI is slow and error-prone. Operations teams need a repeatable pattern that accepts a list of service targets, creates them in controlled parallel batches, and returns a consolidated status view. This example shows how Workflows integrate inventory find, Service Management intent creation, and Jinja2 output formatting.

---

## Pre-requisites

- **NSP:** NSP 25.8 with Workflow Manager enabled; ALED artifacts `service-mgt-artifacts-common` and `service-mgt-artifacts-unified` imported.
- **Access and roles:** Rights to create workflows, import Jinja templates, and invoke Service Management / RESTCONF APIs.
- **Network:** At least two SROS-based network elements discovered in NSP (classic or model-driven), MPLS topology with LSPs/paths, and hardware port `1/1/10` available on both nodes for Dot1Q access.
- **Service template:** A Service Template named `epipe` bound to intent type `epipe` (version 2) must exist in Service Management before execution.
- **Jinja template:** A workflow Jinja template named `serviceState` must be imported in the Workflows app from [`serviceState.jinja2`](./serviceState.jinja2) (see Step 4).
- **Tools or skills:** Basic YAML; familiarity with NSP Workflows UI or VS Code workflow plugin.
- **Disclaimer:** This workflow is a proof-of-concept for OSS development guidance — not intended for production use as-is.

---

## Solution Overview

### High-level design

The workflow has seven tasks in a linear chain with three `with-items` loops for bulk operations:

1. **setVars** — Publishes RESTCONF find URL and Service Management intent-base URL.
2. **getIntentType** — Resolves `intent-type` and `intent-type-version` from the named service template.
3. **getCustomerId** — Looks up customer IDs for each target (parallel, concurrency 10).
4. **createElineService** — POSTs epipe intent payloads for each target (parallel, concurrency 10).
5. **captureCreationTime** — Waits 5 seconds, then reads WFM action-execution timestamps for the create task.
6. **extractTimeDifference** — Python computes elapsed seconds between start and end.
7. **captureServiceState** — Queries each service's SLC state and renders HTML via the `serviceState` Jinja template.

```text
setVars → getIntentType → getCustomerId → createElineService
    → captureCreationTime → extractTimeDifference → captureServiceState
```

### Variables and contract

| Name | Role |
| ---- | ---- |
| `templateName` | Input — Service Management template to use (e.g. `epipe`). |
| `targets` | Input — List of service objects (custName, svcId, svcName, siteA/B, portA/B, sdpAB/BA). |
| `rcFind` | Published — In-cluster RESTCONF find endpoint. |
| `ibsfCreate` | Published — Service Management intent-base URL (in-cluster `restconf-gateway`). |
| `intentType` / `intentTypeVersion` | Published — From template lookup. |
| `customerIds` | Published — Customer IDs aligned with targets. |
| `taskId` | Published — WFM task ID for timing query. |
| `execTime` | Published — Provisioning duration in seconds. |
| `result` | Workflow output — HTML table from Jinja template. |

**Example input:**

```yaml
templateName: epipe
targets:
  - custName: Default customer
    svcId: '1001'
    svcName: epipe_1001
    siteA: 92.168.97.183
    portA: Port 1/1/10
    siteB: 92.168.97.34
    portB: Port 1/1/10
    sdpAB: '2'
    sdpBA: '1'
  - custName: Default customer
    svcId: '1000'
    svcName: epipe_1000
    siteA: 92.168.97.34
    portA: Port 1/1/10
    siteB: 92.168.97.183
    portB: Port 1/1/10
    sdpAB: '1'
    sdpBA: '2'
```

### Step-by-step guide

#### Step 1 — Import artifacts and create the service template

Import `service-mgt-artifacts-common` and `service-mgt-artifacts-unified` via the Artifacts page. Create a Service Template named `epipe` with intent type `epipe`, version 2, and the default config form.

#### Step 2 — Create the workflow definition

Create the workflow from [`bulkEline.yaml`](./bulkEline.yaml) using the Workflows UI or VS Code plugin. Validate and publish the workflow.

**Entry task and URL setup (`setVars`, `getIntentType`):**

```yaml
    setVars:
      action: std.noop
      publish:
        rcFind: https://restconf-gateway/restconf/operations/nsp-inventory:find
        ibsfCreate: https://restconf-gateway/restconf/data/nsp-service-intent:intent-base
      on-success:
        - getIntentType

    getIntentType:
      action: nsp.https
      input:
        method: POST
        url: <% $.rcFind %>
        body:
          input:
            xpath-filter: /service-template:templates/template[name='<% $.templateName %>']
            fields: name;description;intent-type;intent-version;state
            include-meta: false
        resultFilter: $.content.get("nsp-inventory:output")
      publish:
        intentType: <% task().result.content.data.first().get("intent-type") %>
        intentTypeVersion: <% task().result.content.data.first().get("intent-version") %>
```

**Practice note:** `rcFind` uses the in-cluster `restconf-gateway` service name. `getIntentType` applies server-side `fields` and `include-meta: false` to limit payload size.

#### Step 3 — Bulk customer lookup and service creation

`getCustomerId` and `createElineService` use `with-items` with `concurrency: 10` to process all targets in parallel batches.

```yaml
    getCustomerId:
      with-items: item in <% $.targets %>
      concurrency: 10
      action: nsp.https
      input:
        method: POST
        url: <% $.rcFind %>
        body:
          input:
            xpath-filter: /nsp-customer:customers/customer[name='<% $.item.custName %>']
            fields: name;id;description
            include-meta: false
        resultFilter: $.content.get("nsp-inventory:output")
      publish:
        customerIds: <% task().result.content.data.select($.id).flatten() %>
```

`createElineService` nests two `with-items` iterators (`cId` and `item`) to zip customer IDs with target objects and POST the full epipe intent payload to `ibsfCreate`.

#### Step 4 — Import the Jinja2 `serviceState` template

Before execution, register [`serviceState.jinja2`](./serviceState.jinja2) in the Workflows app as template name `serviceState`. The final task calls:

```yaml
result: <% resolve_jinja(fetch_jinja("serviceState"), {execTime => $.execTime, data => task().result.content.flatten()}) %>
```

The template renders an HTML table of service ID, name, SLC state, misaligned flag, and overall creation time.

#### Step 5 — Timing and result capture

`captureCreationTime` uses `wait-before: 5` so the create task completes before querying WFM:

```yaml
    captureCreationTime:
      wait-before: 5
      action: nsp.https
      input:
        method: GET
        url: https://workflow-manager/wfm/api/v1/action-execution/task/<% $.taskId %>
```

`extractTimeDifference` passes only timestamps into `nsp.python` context (not the full `$` workflow context):

```yaml
    extractTimeDifference:
      action: nsp.python
      input:
        context: <% [$.startTime, $.endTime] %>
        script: |
          from datetime import datetime
          date_format = "%Y-%m-%d %H:%M:%S"
          t1 = datetime.strptime(context[0], date_format)
          t2 = datetime.strptime(context[1], date_format)
          return round((t2 - t1).total_seconds());
      publish:
        execTime: <% task().result %>
```

#### Step 6 — Execute the workflow

Publish the workflow, then execute manually or via REST:

```json
POST https://<NSP IP>/wfm/api/v1/execution
Content-Type: application/json

{
  "workflow_id": "bulkEline",
  "input": {
    "templateName": "epipe",
    "targets": [ ... ]
  },
  "params": { "env": "DefaultEnv" },
  "notifyKafka": true
}
```

On success, the quick-view output shows the HTML `result` from `captureServiceState`.

For the full workflow definition, see [`bulkEline.yaml`](./bulkEline.yaml).

---

## Conclusion

You should now understand how `bulkEline` automates bulk E-Line provisioning: template and customer resolution, parallel intent creation, timing measurement, and HTML result formatting. Adapt the target list schema, concurrency, and Jinja template for your own service types and operational requirements.

---

## References

- [Workflows: best practices](https://network.developer.nokia.com/learn/26_4/artifact-development/programming/workflows/wfm-workflow-development/wfm-best-practices/)
- [Workflow actions reference](https://network.developer.nokia.com/learn/26_4/artifact-development/programming/workflows/wfm-workflow-development/wfm-workflow-actions/)
- [Mistral DSL v2](https://docs.openstack.org/mistral/ocata/dsl/dsl_v2.html)
- [Nokia Network Developer Portal](https://network.developer.nokia.com/)
- Workflow definition: [`bulkEline.yaml`](./bulkEline.yaml)
- Jinja template: [`serviceState.jinja2`](./serviceState.jinja2)

---

## Security and operations

This example provisions live services. Before running in a shared or production environment:

- Validate input parameters and target service definitions in a lab first.
- Consider lowering `concurrency` on `with-items` tasks if mistral pods are resource-constrained.
- The `ibsfCreate` URL uses the in-cluster `restconf-gateway` service name, consistent with `rcFind`.
- Register the `serviceState` Jinja template in the Workflows app before execution — the file in this folder is not imported automatically.
