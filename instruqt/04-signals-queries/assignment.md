---
slug: signals-queries
id: ""
type: challenge
title: 'Exercise 4: Signals and Queries'
teaser: Pause a high-value transaction for manager approval, then resume it with a
  signal.
notes:
- type: text
  contents: |-
    # Workflows that wait for humans

    Not every workflow can run straight through. High-value vehicle sales
    require a manager to approve the transaction before payment proceeds.

    Signals let external systems push events into a running workflow.
    The workflow pauses durably — no polling, no timeouts — and resumes
    the moment the signal arrives. Days later if necessary.
- type: text
  contents: |-
    # Queries: read state without changing it

    While the workflow is waiting, you can ask it what it knows: what stage
    is it in, has approval been requested, has it arrived?

    Queries are synchronous reads of workflow state. They don't modify
    anything and can be called at any time from outside the workflow —
    from a dashboard, a CLI, or another service.
tabs:
- title: Terminal 1 - Worker
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- title: Terminal 2 - Starter
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- title: VS Code
  type: service
  hostname: workshop
  port: 8080
- title: Temporal UI
  type: service
  hostname: workshop
  port: 8233
difficulty: basic
timelimit: 2700
enhanced_loading: null
---

# Exercise 4: Signals and Queries

> [!NOTE]
> **Tabs:** [button label="Terminal 1 - Worker" background="#444CE7"](tab-0) · [button label="Terminal 2 - Starter" background="#444CE7"](tab-1) · [button label="VS Code" background="#444CE7"](tab-2) · [button label="Temporal UI" background="#444CE7"](tab-3)

## Your task

Open [button label="VS Code" background="#444CE7"](tab-2) and open `VehicleTransactionWorkflow.cs`.
There are three TODOs:

**1. Signal handler** — add an `ApproveTransactionAsync` method decorated with
`[WorkflowSignal]` that sets `_managerApproved = true`.

**2. Query handler** — add a `GetStatus` method decorated with `[WorkflowQuery]`
that returns a `WorkflowStatus` snapshot of the three state fields. No `await`,
no state mutation.

**3. Approval gate** — in `RunAsync`, after setting `_stage = "awaiting-approval"`,
use `Workflow.WaitConditionAsync(() => _managerApproved)` to pause until the
signal arrives.

Read the README for the full pattern.

## Run it

Start the worker in [button label="Terminal 1 - Worker" background="#444CE7"](tab-0):

```bash,run
dotnet run -- worker
```

Start a high-value workflow in [button label="Terminal 2 - Starter" background="#444CE7"](tab-1)
(it will pause waiting for approval):

```bash,run
dotnet run -- starter
```

Query the workflow status — note the workflow ID printed by the starter:

```bash,run
dotnet run -- query vehicle-tx-VIN-2026-COXAUTO-004
```

Send the approval signal to resume the workflow:

```bash,run
dotnet run -- approve vehicle-tx-VIN-2026-COXAUTO-004
```

Watch the starter terminal — the workflow resumes and completes immediately.

Open [button label="Temporal UI" background="#444CE7"](tab-3) and find the `WorkflowSignalReceived`
event in the history. Notice the workflow was paused between that event and the
`ActivityTaskScheduled` for payment.

Click **Check** when at least one workflow completes after receiving an approval signal.
