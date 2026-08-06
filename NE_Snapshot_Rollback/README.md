# NE Snapshot and Rollback — LSO Operation Manager Artifact Bundle

> Copyright **2024 Nokia**  
> Licensed under the BSD 3-Clause License.  
> SPDX-License-Identifier: BSD-3-Clause

## Summary Table

| Field | Description |
| ----- | ----------- |
| **Title** | NE Snapshot and Rollback — LSO Operation Manager Artifact Bundle |
| **Summary** | An importable NSP artifact bundle that snapshots MD-SROS network element configuration via RESTCONF and restores it on demand, without admin-save or FTP mediation. |
| **Purpose** | Demonstrate how to build paired LSO Operation Manager operations (snapshot and rollback) backed by Mistral workflows, YANG models, and operation profiles for MD-SROS devices. |
| **Technologies Involved** | LSO Operation Manager, Mistral workflows, YANG augmentation, NSP RESTCONF gateway, NSP File Service, MD-SROS RESTCONF |
| **NSP Release** | Tested in NSP 25.4 (validated for NSP 25.4 and later; see `metadata.json` for application compatibility) |

---

## Introduction

**Short description:** This example provides a complete **artifact bundle** for the Nokia Network Services Platform (NSP) that lets operators take a configuration snapshot of a managed MD-SROS network element and roll back to that snapshot later. It is intended for NSP engineers, partners, and customers who want to learn how LSO operations, workflows, and YANG models work together in a reusable, deployable package.

The bundle contains four coordinated artifacts: two **LSO operation types** (`ne-snapshot` and `ne-rollback`) with YANG models and operation profiles, and two **Mistral workflows** that implement the southbound logic against NSP REST APIs.

**Problem statement:** Operational changes to network elements carry risk. Before applying changes, operators often need a reliable way to capture the current configuration and restore it if something goes wrong. Traditional backup approaches may require admin-save or FTP mediation. This example shows how to snapshot and restore MD-SROS configuration using NSP's RESTCONF gateway and File Service, orchestrated through LSO Operation Manager and Mistral workflows—without rebooting the device for rollback.

---

## Pre-requisites

- **NSP:** Version 23.11 or later with Workflow Manager and LSO Operation Manager (`lsom-server-app`) enabled.
- **Access and roles:** Developer or Operator role with permission to import artifact bundles, execute LSO operations, and run workflows.
- **External systems:**
  - At least one **MD-SROS** network element (7750 SR, 7950 XRS, 7450 ESS, or 7250 IXR family) managed in NSP inventory with RESTCONF connectivity.
  - NSP **RESTCONF gateway** and **File Service** reachable from the workflow engine (internal cluster URLs are used by default).
- **Tools or skills:**
  - Basic familiarity with YAML, YANG, and Mistral workflow concepts.
  - Ability to import artifact bundles through the NSP UI or API.
  - Optional: `curl` or Postman for verifying RESTCONF and file-service calls.
- **Other:**
  - This example is for **education and research** purposes only.
  - Not designed for production scale or performance; review Workflow Manager best practices before operational use.
  - Rollback is only allowed when the NE software version matches the version recorded at snapshot time.

---

## Solution Overview

### High-level design

The solution is organized as a single importable artifact bundle (`metadata.json`) containing four artifacts:

| Artifact folder | Target application | Role |
| --------------- | ------------------ | ---- |
| `operation-snapshot/` | `lsom-server-app` | Defines the `ne-snapshot` LSO operation type (YANG model, profile, operation metadata). |
| `operation-rollback/` | `lsom-server-app` | Defines the `ne-rollback` LSO operation type with inputs linking to a prior snapshot. |
| `workflow-snapshot/` | `workflow-manager` | Mistral workflow that fetches NE config via RESTCONF, stores it on the File Service, and zips the backup. |
| `workflow-rollback/` | `workflow-manager` | Mistral workflow that reads a stored backup and applies it via YANG PATCH to the NE. |

**Snapshot workflow (`ne-snapshot`)** stages:

1. Resolve NE details from NSP inventory.
2. Retrieve `nokia-conf:configure` via RESTCONF GET.
3. Create a timestamped directory on the File Service.
4. Upload the configuration as JSON and zip the backup folder.

**Rollback workflow (`ne-rollback`)** stages:

1. Resolve NE details and locate backup metadata (from a linked snapshot operation or explicit path/filename inputs).
2. Verify NE software version matches the snapshot version.
3. Read the backup JSON from the zipped file on the File Service.
4. Apply configuration via RESTCONF YANG PATCH (`replace` on `nokia-conf:/configure`).

### Repository layout

```
ne_snapshot_rollback_artifactBundle/
├── README.md                    # This file (activity documentation)
├── metadata.json                # Artifact bundle manifest for NSP import
├── operation-snapshot/          # LSO operation type: ne-snapshot
│   ├── ne-snapshot.yang
│   ├── ne-snapshot.yaml
│   └── operation_types.json
├── operation-rollback/            # LSO operation type: ne-rollback
│   ├── ne-rollback.yang
│   ├── ne-rollback.yaml
│   └── operation_types.json
├── workflow-snapshot/             # Mistral workflow for snapshot
│   ├── ne-snapshot.yaml
│   └── ne-snapshot.json           # Workflow UI input schema (neId autocomplete)
└── workflow-rollback/             # Mistral workflow for rollback
    └── ne-rollback.yaml
```

### Step-by-step guide

#### 1. Review the artifact bundle

Inspect `metadata.json` to confirm artifact names, versions, and target applications before import:

```json
{
  "meta-data-header": {
    "title": "Operation Manager Snapshot/Rollback NE Config",
    "description": "Take device snapshot and rollback"
  }
}
```

#### 2. Import the bundle into NSP

1. Package the `ne_snapshot_rollback_artifactBundle` folder contents (or the full folder) as an artifact bundle acceptable to your NSP import mechanism.
2. Import via the NSP UI or API into the target NSP instance (23.11+).
3. Verify that four artifacts are registered:
   - `operation-snapshot` → LSO Operation Manager
   - `operation-rollback` → LSO Operation Manager
   - `workflow-snapshot` → Workflow Manager
   - `workflow-rollback` → Workflow Manager

#### 3. Execute a configuration snapshot

1. Open **LSO Operation Manager** and create or run the `ne-snapshot` operation.
2. Provide the target **NE ID** (the workflow UI schema in `workflow-snapshot/ne-snapshot.json` supports NE autocomplete).
3. On success, the operation execution state includes:
   - `backupFilename` — zipped backup file name (timestamp-based)
   - `filePath` — directory path on the File Service
   - `neSoftwareVersion` — SR OS version at snapshot time

Key workflow input/output (see `workflow-snapshot/ne-snapshot.yaml`):

```yaml
input:
  - neId

output:
  lsoInfo: <% $.lsoInfo %>
  backupFilename: <% $.backupFilename %>
  filePath: <% $.filePath %>
  neSoftwareVersion: <% $.neSoftwareVersion %>
```

#### 4. Execute a configuration rollback

1. Run the `ne-rollback` operation against the same NE.
2. Link the rollback to a prior snapshot by providing `backup_operation`, or supply `backupFilename` and `backupFilePath` directly.
3. The workflow verifies that the current NE software version matches the snapshot version before applying the configuration.

Rollback applies configuration with YANG PATCH (see `workflow-rollback/ne-rollback.yaml`):

```yaml
rollbackConfig:
  action: nsp.https
  input:
    url: https://restconf-gateway/restconf/data/network-device-mgr:network-devices/network-device=<% $.neId %>/root
    method: PATCH
    contentType: application/yang-patch+json
    body:
      ietf-yang-patch:yang-patch:
        edit:
        - operation: replace
          target: nokia-conf:/configure
          value: <% json_parse($.jsonConfig) %>
```

#### 5. Verify results

- Confirm the snapshot operation reports `lsoInfo: "Successfully done backup operation"`.
- Confirm the rollback operation reports `lsoInfo: "Restore successful"`.
- Validate NE configuration in the NSP UI or via RESTCONF after rollback.

---

## Conclusion

You now have a working example of an **LSO Operation Manager artifact bundle** that pairs snapshot and rollback operations with Mistral workflows for MD-SROS devices. The pattern—YANG augmentation for operation state, operation profiles for NE family mapping, and workflows calling NSP internal REST APIs—can be extended to other device types or backup stores.

**Possible next steps:**

- Adapt the RESTCONF URLs and YANG paths for NETCONF/gRPC-managed devices or third-party vendors.
- Add error-handling, retry, or notification tasks per Workflow Manager best practices.
- Integrate snapshot execution as a pre-change step in a larger orchestration workflow.

---

## References

- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- LSO Operation Manager and Workflow Manager product documentation on the [Nokia Documentation Portal](https://documentation.nokia.com/nsp/)

---

## Troubleshooting

| Symptom | Likely cause | Resolution |
| ------- | ------------ | ---------- |
| `Failed: fetching node details` | NE not found in inventory or invalid `neId` | Verify the NE is discovered and communication state is healthy. |
| `Failed: getting config from MD-SROS node` | Device not MD-SROS or RESTCONF path unavailable | Confirm NE family is supported (7750 SR, 7950 XRS, 7450 ESS, 7250 IXR). |
| `Failed: creating backup directory on file-server` | File Service unreachable | Check File Service pod health and internal DNS resolution. |
| `Rollback is not allowed for snapshots taken with a different SR OS release` | NE upgraded or downgraded since snapshot | Take a new snapshot on the current software version. |

---

## Security and Operations

- Workflows use **internal cluster service URLs** (`restconf-gateway`, `file-service`); do not expose these externally.
- No credentials are hardcoded; authentication is handled by the NSP platform for `nsp.https` actions.
- Backup files are stored on the NSP File Service under `/lsom/neBackup/<vendor>/<family>/<ne-name>/<version>/`.
- Review retention and access controls for backup files before using in operational environments.

---

## Changelog

| Version | NSP release | Notes |
| ------- | ----------- | ----- |
| 1.0.1 | 23.11+ | Initial release; reorganized into `ne_snapshot_rollback_artifactBundle` with contribution-template documentation. |
