# Workflow Artifact: ne-rollback

> Copyright **2024 Nokia**  
> Licensed under the BSD 3-Clause License.  
> SPDX-License-Identifier: BSD-3-Clause

Mistral workflow definition for the **ne-rollback** LSO operation. This artifact is deployed to Workflow Manager as part of the NE Snapshot Rollback activity bundle.

**Full documentation:** See [../README.md](../README.md) for prerequisites, step-by-step instructions, and troubleshooting.

**Artifact files in this folder:**

| File | Purpose |
| ---- | ------- |
| `ne-rollback.yaml` | Mistral workflow — reads a stored backup and restores configuration via RESTCONF YANG PATCH. |

**Disclaimers:** For education/research only. MD-SROS devices only. Rollback requires matching NE software version. Not designed for production scale.
