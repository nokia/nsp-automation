# Hello World workflow with std.echo

| Field                     | Description                                                                                                      |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Title**                 | Hello World workflow with std.echo                                                                               |
| **Summary**               | A minimal workflow that echoes a greeting message using `std.echo`, with input validation, publish, and output. |
| **Purpose**               | Teach workflow structure, DRAFT/PUBLISHED lifecycle, input/output handling, and error handling for beginners.    |
| **Technologies Involved** | Mistral workflows, `std.echo`, `nsp.assert`, YAQL.                                                             |
| **NSP Release**           | NSP 25.8                                                                                                         |

---

## Introduction

**Short description:** This tutorial walks you through creating your first NSP Workflow Manager (WFM) workflow. You build a simple "Hello World" workflow step by step, using the `std.echo` action to return a greeting message. No network devices or external systems are required.

**Problem statement:** Before automating network operations, you need to understand how workflows are structured, validated, published, and executed in NSP. This example introduces core concepts—tasks, actions, input, publish, output, and error handling—without the complexity of network actions or device connectivity.

**Who this is for:** Engineers, operators, and developers who are new to NSP Workflow Manager and want a clear starting point before moving on to NSP API calls or device automation.

---

## Pre-requisites

- **NSP:** NSP 25.8 (or compatible release) with Workflow Manager enabled.
- **Access and roles:** Developer or Operator role in the NSP WebUI.
- **External systems:** None. This example runs entirely inside Workflow Manager.
- **Tools or skills:** Basic familiarity with YAML; ability to navigate the NSP WebUI.
- **Other:** No lab topology or network elements are required.

---

## Solution Overview

### High-level design

The final workflow has two tasks:

1. **validate_message** — Uses `nsp.assert` to check that the input message is not empty.
2. **greet** — Uses `std.echo` to return the greeting message.

The workflow accepts a `message` input (default: `"Hello World!"`), publishes the echo result, and returns a clean JSON output on success or failure.

### Two ways to use this tutorial

**Option A — Build step by step (recommended for beginners)**  
Follow Steps 1–6 below. Each step adds one concept so you see how the workflow grows.

**Option B — Import the finished workflow**  
If you want to run the example first or compare your result against the reference:

1. Download [hello-world.yaml](./hello-world.yaml) from this folder (or clone the repository and open the file locally).
2. In Workflow Manager, open the **Workflows** view and use **Import Workflow from File** (Developer role required).
3. Select `hello-world.yaml`, then click **VALIDATE & UPDATE FLOW**.
4. **CREATE** the workflow, change status to **PUBLISHED**, and execute it with the default input.

You should get the success output shown in Step 6. Read Steps 1–6 afterward to understand each part of the YAML.

### Step-by-step guide

#### Step 1 — Create a minimal workflow

Open the NSP WebUI and start **Workflow Manager** from the launchpad under **CONTROL / FULFILL / OPTIMIZE**.

1. Click **Dashboard**, open the dropdown, and select **Workflows**.
2. Click **(+) CREATE WORKFLOW**. The editor opens with a skeleton workflow.
3. Replace the skeleton with the following YAML:

```yaml
version: '2.0'

helloWorld:
  type: direct

  tasks:
    greet:
      action: std.echo
      input:
        output: "Hello World!"
```

**What you are learning:**

- `version: '2.0'` — Mistral workflow language version.
- `helloWorld` — The workflow name (identifier).
- `type: direct` — Tasks run in a defined sequence (as opposed to `reverse`).
- `tasks` — The list of steps the workflow executes.
- `action: std.echo` — A built-in action that returns the value you provide, without calling external systems.

> **Bad vs good — hardcoded values**
>
> | Avoid (bad) | Prefer (good) |
> |-------------|---------------|
> | Hardcoding `"Hello World!"` in the task when the message should be configurable | Define an `input` section with defaults (see Step 3) |

4. Click **VALIDATE & UPDATE FLOW** and switch to **FLOW** view. Validation should succeed and show a single task named `greet`.
5. Click **CREATE**. The workflow is saved in **DRAFT** state.

#### Step 2 — Publish and execute the workflow

Workflows in DRAFT state cannot be executed. You must publish them first.

1. From the workflow context menu (three dots on the right), select **Modify Status**.
2. Select **PUBLISHED** and click **UPDATE**.
3. Execute the workflow.
4. Open **Quick View** to inspect the result.

At this stage, the execution output is empty (`{}`) because the workflow has not yet defined `input` or `output` sections. To see task-level details, switch to **FLOW** view in Quick View, where you can inspect task and action execution including run time.

> **Bad vs good — workflow metadata**
>
> | Avoid (bad) | Prefer (good) |
> |-------------|---------------|
> | Bare workflow with no description or tags | Add `workflow_meta`, `description`, and `tags: [Tutorial]` (see Step 6) |

#### Step 3 — Add input variables

To make the greeting configurable, move the message into the workflow input section.

1. Move the workflow back to **DRAFT** state (Modify Status).
2. Open the workflow editor and update the YAML:

```yaml
version: '2.0'

helloWorld:
  type: direct

  input:
    - message: "Hello World!"

  tasks:
    greet:
      action: std.echo
      input:
        output: <% $.message %>
```

The `input` section defines a variable called `message` with a default value. The expression `<% $.message %>` reads that variable at run time.

3. Validate, **PUBLISH** the workflow (this saves changes and sets status to PUBLISHED in one step), then execute it.

When you execute the workflow, the run form shows the `message` field pre-filled with `"Hello World!"`. You can change it before starting the run. Workflow Manager treats input variables as strings by default.

> **Bad vs good — input handling**
>
> | Avoid (bad) | Prefer (good) |
> |-------------|---------------|
> | Hardcoded string in the task `input` | `input` section with sensible defaults |

#### Step 4 — Publish the task result

Action results are available inside the task via `task().result`. To pass data to later tasks or to the workflow output, use the `publish` section.

Update the `greet` task:

```yaml
    greet:
      action: std.echo
      input:
        output: <% $.message %>
      publish:
        greeting_result: <% task().result %>
```

For `std.echo`, `task().result` is the echoed string (for example, `"Hello World!"`).

**Important:** `publish` runs only when the action succeeds. If the action fails, use `publish-on-error` instead (covered in Step 5).

> **Bad vs good — retrieving results**
>
> | Avoid (bad) | Prefer (good) |
> |-------------|---------------|
> | Navigating to action execution details to read the result | `publish` to expose the result in the workflow context |

#### Step 5 — Validate input and handle errors

A robust workflow checks parameters before doing work. Use `nsp.assert` instead of `std.fail` for validation—`std.fail` fails the entire workflow immediately, while `nsp.assert` lets you handle validation errors in a controlled way.

Add a validation task before `greet`:

```yaml
  tasks:
    validate_message:
      action: nsp.assert
      input:
        input: <% $.message %>
        expected: ""
        shouldFail: false
      on-success:
        - greet
      publish-on-error:
        validation_error: "Message cannot be empty"

    greet:
      action: std.echo
      input:
        output: <% $.message %>
      publish:
        greeting_result: <% task().result %>
      publish-on-error:
        greeting_error: <% task().result %>
```

**How validation works:**

- `shouldFail: false` (the default) means the task **fails when `input` equals `expected`**.
- If `message` is empty (`""`), validation fails and `greet` does not run.
- If `message` is not empty, validation succeeds and the flow continues to `greet`.

> **Tip:** Use `shouldFail: true` when you want the task to fail on a **difference** (for example, verifying that `count` equals `limit`). Use `shouldFail: false` when you want the task to fail on a **match** (for example, rejecting an empty string).

Test the error path by executing the workflow with an empty `message`. The workflow should fail with a clear validation message instead of a raw Mistral stack trace.

> **Bad vs good — parameter validation**
>
> | Avoid (bad) | Prefer (good) |
> |-------------|---------------|
> | `std.fail` to check parameters | `nsp.assert` with `publish-on-error` for controlled validation |
> | Failing mid-workflow after changes have started | Validate inputs in the first task before any work |

#### Step 6 — Control the workflow output

By default, all published variables appear in the workflow output. For readability, define explicit `output` and `output-on-error` sections.

If you followed Steps 1–5, your `tasks` section is already complete. **Add** the following sections to the workflow you built—do not replace the whole file:

```yaml
  workflow_meta:
    title: Hello World
    author: NSP Automation
    version: "1.0.0"
    deprecated: false

  description: Echoes a greeting message. Validates input before execution.

  tags:
    - Tutorial

  output:
    status: success
    greeting: <% $.greeting_result %>

  output-on-error:
    status: failed
    reason: <% $.validation_error %>
```

Place `workflow_meta`, `description`, and `tags` alongside `type: direct` at the top of `helloWorld`. Place `output` and `output-on-error` after `input` and before `tasks`.

The complete workflow is in [hello-world.yaml](./hello-world.yaml). Use it to verify your YAML matches the reference, or import it directly (see **Option B** above).

**Expected output — success (default input):**

```json
{
  "status": "success",
  "greeting": "Hello World!"
}
```

**Expected output — validation failure (empty message):**

```json
{
  "status": "failed",
  "reason": "Message cannot be empty"
}
```

Validate, publish, and execute the workflow to confirm both paths.

> **Bad vs good — workflow output**
>
> | Avoid (bad) | Prefer (good) |
> |-------------|---------------|
> | Raw Mistral error output with long stack traces | `output-on-error` with a short, meaningful `reason` |
> | Dumping all published variables into output | `output` section with only the fields callers need |

---

## Conclusion

You now have a working Hello World workflow that demonstrates:

- Workflow structure (version, name, type, tasks, actions)
- DRAFT and PUBLISHED lifecycle
- Input variables with defaults
- Publishing task results with `publish` and `publish-on-error`
- Input validation with `nsp.assert`
- Controlled success and error output with `output` and `output-on-error`
- Workflow metadata (`workflow_meta`, `description`, `tags`)

---

## References

- [NSP Workflow description](https://documentation.nokia.com/nsp/25-8/Network_Automation/wf_desc.html)
- [Nokia Network Developer Portal – NSP](https://network.developer.nokia.com/)
- [Mistral workflow language v2](https://docs.openstack.org/mistral/latest/user/wf_lang_v2.html)
- [Nokia NSP workflow repository on GitHub](https://github.com/nokia/nsp-workflow) — additional samples and tutorials

---

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---------|--------------|------------|
| Validation fails with default `"Hello World!"` message | `shouldFail: true` rejects any non-empty value | Set `shouldFail: false` (see [hello-world.yaml](./hello-world.yaml)) |
| Validation fails on create | YAML indentation or syntax error | Check that tasks are indented with two spaces under `helloWorld`; use **VALIDATE & UPDATE FLOW** for details |
| Workflow cannot be executed | Workflow is in DRAFT state | Modify Status to **PUBLISHED** |
| Output is `{}` | No `output` section defined yet | Complete Step 6 to add `output` and `output-on-error` |
| `reason` is empty on error | Validation error not published | Ensure `publish-on-error` is set on `validate_message` |
| Empty message not rejected | `shouldFail` set to `true` | Use `shouldFail: false` so the assert fails when `message` equals `""` |

---

## Changelog

| Version | NSP Release | Changes |
|---------|-------------|---------|
| 1.0.0   | 25.8        | Initial Hello World tutorial using `std.echo` and `nsp.assert` |
