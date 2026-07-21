# Tech Support Files Generator (NeTsFileGenerator)

---

## Summary Table

| Field                     | Description                                                                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Tech Support files generator                                                                                                                |
| **Summary**               | A Mistral workflow that generates a tech-support dump on an NSP-managed SR OS router, uploads it to the NSP file server, and deletes the file from the device. |
| **Purpose**               | Demonstrate automating a support-driven diagnostic workflow: resolve NE metadata, ensure target directory exists, run CLI tech-support, async file transfer, and cleanup by device management mode. |
| **Technologies Involved** | Mistral workflows, `nsp.https`, `nsp.https_async`, `nsp.managed_cli`, `std.noop`, Network Supervision API, NSP file-service, file-transfer-service, Kafka (`nsp-file-service` topic). |
| **NSP Release**           | NSP 26.4 (per `workflow_meta.dependencies.platform.nspOS`).                                                                                 |

---

## Introduction

**Short description:** This activity walks through the `NeTsFileGenerator` workflow, which creates a tech-support binary on a named network element, transfers it to `/TS/{neName}` on the NSP file server, and removes the local copy from the router. The workflow branches cleanup commands based on whether the NE is classic (`nfmp`) or model-driven (`mdm`).

**Problem statement:** Collecting tech-support dumps for Nokia TAC analysis is a multi-step, error-prone manual process involving CLI access, file paths, and FTP storage. This example shows how Workflows orchestrate supervision lookup, directory management, managed CLI, asynchronous file transfer, and session-aware cleanup in one repeatable flow.

---

## Pre-requisites

- **NSP:** NSP 26.4; Workflow Manager, Network Supervision, file-service, and file-transfer-service available in-cluster.
- **Authorization:** Use only with explicit Nokia support direction — generates system core dumps.
- **Access and roles:** Rights to run workflows, invoke managed CLI on target NEs, and use file transfer APIs.
- **Network:** At least one discovered SR OS network element resolvable by name in Network Supervision.
- **Storage:** `/TS` purge policy on the FTP/file server as needed to cap retained dumps.
- **Tools or skills:** Basic YAML; familiarity with NSP Workflows UI or VS Code plugin.
- **Disclaimer:** Proof-of-concept for OSS development — not for production use as-is.

---

## Solution Overview

### High-level design

Eight tasks form a linear flow with error branches and mode-specific cleanup:

1. **getNeIP** — Query Network Supervision for `neId`, `ipAddress`, `sourceType`.
2. **checkTargetDir** — GET list directories for `/TS/{neName}`; on error → **createDir**.
3. **createDir** — POST create directory if missing.
4. **first_ts_file** — `admin tech-support` via `nsp.managed_cli`; session left open (`closeSession: false`).
5. **upload_ts_files** — `nsp.https_async` transfers file to file-service; Kafka topic `nsp-file-service`.
6. **deleteOnClassicNode** or **deleteOnMdNode** — Conditional on `sourceType`; reuses `sessionId`, `closeSession: true`.
7. **endWF** — `std.noop` error path.

```text
getNeIP → checkTargetDir → first_ts_file → upload_ts_files
              ↓ (on-error)                    ↓ on-complete
           createDir ─────────────→    deleteOnClassicNode (nfmp)
                                       deleteOnMdNode (mdm)
getNeIP / first_ts_file / upload / delete* on-error → endWF
```

### Variables and contract

| Name | Role |
| ---- | ---- |
| `neName` | Input — Network element name (e.g. `IXR1`). |
| `cfCard` | Input — Compact flash card (e.g. `cf3`). |
| `dirName` | Var — `/TS/{neName}` destination on file server. |
| `TsFileDate` | Var — UTC date stamp for filename (`get_UTC_time`). |
| `neId`, `mgmtIP`, `sourceType` | Published — From Network Supervision. |
| `sessionId`, `file_list` | Published — From tech-support CLI task. |
| `result` | Output — `success` on happy path. |

**Example input:**

```json
{
  "neName": "IXR2",
  "cfCard": "cf3"
}
```

Generated TS filename pattern: `{cfCard}:TS-file-{neName}-{TsFileDate}-TS1.bin`

### Step-by-step guide

#### Step 1 — Create and publish the workflow

Import [`NeTsFileGenerator.yaml`](./NeTsFileGenerator.yaml) via the Workflows UI or VS Code plugin. Validate, create, and publish the workflow.

#### Step 2 — Resolve network element metadata

`getNeIP` uses in-cluster Network Supervision with a `resultFilter` to limit stored fields:

```yaml
    getNeIP:
      action: nsp.https
      input:
        url: "https://network-supervision/NetworkSupervision/rest/api/v1/networkElements?filter=name='<% $.neName %>'"
        resultFilter: $.content.response.data.select({ ipAddress=>$.ipAddress, neId=>$.neId, sourceType=>$.sourceType })
      publish:
        mgmtIP: <% task().result.content[0].ipAddress %>
        neId: <% task().result.content[0].neId %>
        sourceType: <% task().result.content[0].sourceType %>
```

**Practice note:** `resultFilter` reduces workflow database payload (client-side filter pattern).

#### Step 3 — Ensure file-server directory exists

`checkTargetDir` lists `/TS/{neName}`. If the directory is missing, `createDir` POSTs to file-service, then both paths continue to `first_ts_file`.

```yaml
    checkTargetDir:
      action: nsp.https
      input:
        url: "https://file-service/nsp-file-service-app/rest/api/v1/directory/listDirectories?dirName=<% $.dirName %>&recursive=true"
        method: GET
      on-error:
        - createDir
```

#### Step 4 — Generate tech-support file on device

`first_ts_file` opens a managed CLI session, runs `admin tech-support`, and publishes `sessionId` for reuse:

```yaml
    first_ts_file:
      action: nsp.managed_cli
      input:
        neId: <% $.neId %>
        cmds:
          - admin tech-support <% $.cfCard %>:TS-file-<% $.neName %>-<% $.TsFileDate %>-TS1.bin
        sessionId: "new"
        closeSession: false
        stopOn: '[\r\n]+(Error|MINOR):'
      publish:
        sessionId: <% task().result.sessionId %>
        file_list: ["<% $.cfCard %>:TS-file-<% $.neName %>-<% $.TsFileDate %>-TS1.bin"]
```

#### Step 5 — Async upload to file server

`upload_ts_files` uses `nsp.https_async` with `with-items`, `concurrency: 1`, and Kafka completion via `jsonPathSuccess` / `jsonPathError`:

```yaml
    upload_ts_files:
      action: nsp.https_async
      with-items: TS_files in <% $.file_list %>
      concurrency: 1
      input:
        url: https://file-transfer-service/nsp-file-transfer-service-app/rest/api/v1/neservice/getFile
        method: POST
        kafkaTopic: nsp-file-service
      on-complete:
        - deleteOnClassicNode: <% $.sourceType = "nfmp" %>
        - deleteOnMdNode: <% $.sourceType = "mdm" %>
```

#### Step 6 — Device cleanup by management mode

Classic nodes use `file delete`; model-driven nodes use `file remove`. Both reuse `sessionId` and set `closeSession: true`.

#### Step 7 — Execute the workflow

```json
POST https://<NSP IP>/wfm/api/v1/execution
Content-Type: application/json

{
  "workflow_id": "NeTsFileGenerator",
  "input": {
    "neName": "IXR2",
    "cfCard": "cf3"
  },
  "params": { "env": "DefaultEnv" },
  "notifyKafka": true
}
```

On success, quick-view output shows `result: success`. Download the TS file from `/TS/{neName}` on the NSP file server.

For the full definition, see [`NeTsFileGenerator.yaml`](./NeTsFileGenerator.yaml).

---

## Conclusion

You should now understand how `NeTsFileGenerator` automates tech-support collection: supervision lookup, file-server directory handling, managed CLI generation, async NE-to-server transfer, and mode-specific cleanup. Extend with validation, richer error output, and purge policies for operational use.

---

## References

- [Workflows: best practices](https://network.developer.nokia.com/learn/26_4/artifact-development/programming/workflows/wfm-workflow-development/wfm-best-practices.html)
- [Workflow actions reference](https://network.developer.nokia.com/learn/26_4/artifact-development/programming/workflows/wfm-workflow-development/wfm-workflow-actions.html)
- [Mistral DSL v2](https://docs.openstack.org/mistral/ocata/dsl/dsl_v2.html)
- [Nokia Network Developer Portal](https://network.developer.nokia.com/)
- Workflow definition: [`NeTsFileGenerator.yaml`](./NeTsFileGenerator.yaml)

---

## Security and operations

This workflow generates tech-support dumps and transfers them off the device. Before running in any environment:

- Obtain explicit Nokia support authorization — this is not a routine operational workflow.
- Run first in a lab with a test NE; verify CF card free space and transfer completion.
- Configure a `/TS` purge policy on the file server to cap retained dumps.
- There is no `dryRun` path and no `output-on-error` contract — failures route to `endWF` with a generic message. Consider adding `nsp.assert` on inputs and structured error output for production hardening.
- File transfer sets `retries: 5` in the request body; HTTPS lookup tasks have no Mistral-level `retry` policy.
- URLs use in-cluster Kubernetes service names (`network-supervision`, `file-service`, `file-transfer-service`).
