---
slug: converting
id: ""
type: challenge
title: "Exercise 1: Converting a Workflow"
teaser: Replace a fragile retry-riddled pipeline with a durable Temporal workflow.
notes:
- type: text
  contents: |-
    # The problem with hand-rolled retry loops

    You have a vehicle purchase pipeline. It calls three services: fraud check,
    payment, and title transfer. Each has its own retry loop with different logic.

    If the process crashes mid-transaction, the order is gone. There's no way
    to resume. A payment that succeeded before the crash might never be followed
    by a title transfer.

    Temporal solves this by recording every step in an event history. If the
    process crashes, the next worker picks up exactly where it left off.
- type: text
  contents: |-
    # What you'll build

    You'll convert VehicleTransactionPipeline.cs — a realistic but fragile
    sequential method — into a proper Temporal workflow with activities.

    The retry loops, the Task.Delay backoff math, the if-attempt==N-throw
    guards: none of that belongs in your code. Temporal owns all of it.

    Your workflow will describe only the happy path.
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

# Exercise 1: Converting a Workflow

> [!NOTE]
> **Tabs:** [button label="Terminal 1 - Worker" background="#444CE7"](tab-0) · [button label="Terminal 2 - Starter" background="#444CE7"](tab-1) · [button label="VS Code" background="#444CE7"](tab-2) · [button label="Temporal UI" background="#444CE7"](tab-3)

## Start here

Click [button label="VS Code" background="#444CE7"](tab-2) and open `/root/workshop/exercise`.

Read `VehicleTransactionPipeline.cs` first — it's the before picture. Then open
`VehicleTransactionActivities.cs` and `VehicleTransactionWorkflow.cs` where your
TODOs are waiting.

The solution is in `/root/workshop/solution` if you want to compare at any point.

## Part A — Implement the Activities

Open `VehicleTransactionActivities.cs`. Implement the three `[Activity]`-annotated
methods. Each calls one of the simulated services already defined in
`VehicleTransactionPipeline.cs`.

For the fraud check: if the service returns `false`, throw
`ApplicationFailureException` with `nonRetryable: true` — this is a business rule
failure, not a transient error. Temporal should not retry it.

## Part B — Implement the Workflow

Open `VehicleTransactionWorkflow.cs`. Two TODOs:

**First:** define a static `ActivityOptions` with a `StartToCloseTimeout` and a
`RetryPolicy`. This replaces all the retry math in the pipeline.

**Second:** implement `RunAsync` by calling all three activities in sequence using
`Workflow.ExecuteActivityAsync`, then return a `TransactionResult`.

## Run it

Start the worker in [button label="Terminal 1 - Worker" background="#444CE7"](tab-0):

```bash,run
dotnet run -- worker
```

Start a workflow in [button label="Terminal 2 - Starter" background="#444CE7"](tab-1):

```bash,run
dotnet run -- starter
```

Open [button label="Temporal UI" background="#444CE7"](tab-3) and find your workflow. Click into it
and explore the event history. Notice each activity as a separate entry.

**Try this:** kill the worker with Ctrl+C while a workflow is in progress.
Restart it. What happens to the in-flight transaction?

Click **Check** when at least one workflow completes successfully.
