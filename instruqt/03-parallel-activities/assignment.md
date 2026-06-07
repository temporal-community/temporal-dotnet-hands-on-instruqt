---
slug: parallel-activities
id: oyt5dofafhs3
type: challenge
title: 'Exercise 3: Parallel Activities'
teaser: Fan out independent checks concurrently and gate on both completing before
  payment proceeds.
notes:
- type: text
  contents: |-
    # Why run things sequentially when you don't have to?

    The fraud check and inventory confirmation are completely independent.
    Neither needs the other's result to start. Running them one after the
    other wastes time.

    Temporal makes concurrent activities straightforward: start both without
    awaiting, then wait for both together. Temporal replays the fan-out
    correctly from history — no locks, no race conditions.
- type: text
  contents: |-
    # What you'll build

    You'll modify the workflow to start CheckFraudAsync and ConfirmInventoryAsync
    at the same time, then use Task.WhenAll to wait for both before proceeding
    to payment.

    If either check fails permanently, the workflow fails before payment runs.
    Open the Temporal UI after a run and look at the event history — both
    ActivityTaskScheduled events will appear at nearly the same sequence number.
tabs:
- id: w3pwsaquih4k
  title: Terminal 1 - Worker
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: axofbdrcg3f4
  title: Terminal 2 - Starter
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: stiyxh7f15rt
  title: VS Code
  type: service
  hostname: workshop
  port: 8080
- id: oycyy523drsp
  title: Temporal UI
  type: service
  hostname: workshop
  port: 8233
difficulty: basic
timelimit: 2700
enhanced_loading: null
---

# Exercise 3: Parallel Activities

> [!NOTE]
> **Tabs:** [button label="Terminal 1 - Worker" background="#444CE7"](tab-0) · [button label="Terminal 2 - Starter" background="#444CE7"](tab-1) · [button label="VS Code" background="#444CE7"](tab-2) · [button label="Temporal UI" background="#444CE7"](tab-3)

## Your task

Open [button label="VS Code" background="#444CE7"](tab-2) and open `VehicleTransactionWorkflow.cs`.
There is one TODO in `RunAsync`.

Start both `CheckFraudAsync` and `ConfirmInventoryAsync` without awaiting either
immediately — this gives you two running `Task` objects. Then `await Task.WhenAll`
on both before continuing to payment.

Read the README for the full pattern and hints.

## Run it

Start the worker in [button label="Terminal 1 - Worker" background="#444CE7"](tab-0):

```bash,run
dotnet run -- worker
```

Start a workflow in [button label="Terminal 2 - Starter" background="#444CE7"](tab-1):

```bash,run
dotnet run -- starter
```

Open [button label="Temporal UI" background="#444CE7"](tab-3) and inspect a completed workflow's
event history. Look for the two `ActivityTaskScheduled` events for the checks —
they should appear at nearly the same sequence number, confirming they ran
concurrently.

Click **Check** when at least one workflow completes successfully.
