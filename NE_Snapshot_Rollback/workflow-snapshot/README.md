# Workflow Artifact: ne-snapshot

> Copyright **2024 Nokia**  
> Licensed under the BSD 3-Clause License.  
> SPDX-License-Identifier: BSD-3-Clause

Mistral workflow definition for the **ne-snapshot** LSO operation. This artifact is deployed to Workflow Manager as part of the NE Snapshot Rollback activity bundle.

**Full documentation:** See [../README.md](../README.md) for prerequisites, step-by-step instructions, and troubleshooting.

**Artifact files in this folder:**

| File | Purpose |
| ---- | ------- |
| `ne-snapshot.yaml` | Mistral workflow — fetches MD-SROS config via RESTCONF and stores a zipped backup on the File Service. |
| `ne-snapshot.json` | Workflow UI input schema — provides NE ID autocomplete for operation execution. |

**Disclaimers:** For education/research only. MD-SROS devices only. Not designed for production scale.
