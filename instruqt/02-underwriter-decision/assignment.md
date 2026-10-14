---
slug: underwriter-decision
id: yn9qqfdcjm7u
type: challenge
title: 'Challenge 2: Human in the Loop'
teaser: Pause an application for an underwriter, expose its state with a query, and
  escalate if nobody rules in time.
notes:
- type: text
  contents: |-
    # Workflows that wait for people

    The automated checks are done. Now a person has to rule on the
    application — and that person might answer in thirty seconds or
    next Tuesday.

    Without Temporal you would persist the half-finished application,
    write a scheduler to find stalled rows, build a callback endpoint,
    and reconstruct state when the answer arrives.

    Here the workflow simply waits. It holds no thread and no memory
    on any worker while it does.
- type: text
  contents: |-
    # Updates, queries, and a timer

    An update carries the underwriter's ruling in and hands back a
    response, so they learn whether it was accepted. A validator can
    reject a bad ruling before it ever reaches the event history.

    A query reads the application's current state without disturbing
    it or appearing in history at all.

    And a timeout on the wait lets you escalate a stalled application
    without taking the decision away from the human.
tabs:
- id: 2jxuwxy4eeho
  title: Terminal 1 - Worker
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: 5pjiidf3pyyo
  title: Terminal 2 - Starter
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: ysbkfyaxlacm
  title: Terminal 3 - Underwriter
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: 04v6ulskisxy
  title: VS Code
  type: service
  hostname: workshop
  port: 8080
- id: bxtf0jrvtonw
  title: Temporal UI
  type: service
  hostname: workshop
  port: 8233
difficulty: basic
timelimit: 1800
enhanced_loading: null
---

# Challenge 2: Human in the Loop

> [!NOTE]
> **Tabs:** [button label="Terminal 1 - Worker" background="#444CE7"](tab-0) · [button label="Terminal 2 - Starter" background="#444CE7"](tab-1) · [button label="Terminal 3 - Underwriter" background="#444CE7"](tab-2) · [button label="VS Code" background="#444CE7"](tab-3) · [button label="Temporal UI" background="#444CE7"](tab-4)

## Start here

Click [button label="VS Code" background="#444CE7"](tab-3) and open
`LoanUnderwritingWorkflow.cs` in `/root/workshop/exercise`.

The three activity calls from Challenge 1 are already wired up. Your job is the
human-in-the-loop part: four TODOs. `README.md` has the full walkthrough.

---

## Part A — The update handler (TODO 1)

Decorate `SubmitDecisionAsync` with `[WorkflowUpdate]` and record the ruling in
`_decision`.

**Update or signal?** A signal is fire-and-forget — the sender learns nothing.
An update is a round trip: the caller waits for your handler and receives its
return value. An underwriter submitting a ruling should find out whether it was
accepted, so this is an update.

## Part B — The validator (TODO 2)

Add a method decorated with `[WorkflowUpdateValidator(nameof(SubmitDecisionAsync))]`.

A validator runs *before* the update is accepted. Throw from it and the update
is rejected, the caller is told why, and **nothing is written to the event
history** — the workflow never sees the bad input.

Reject anything that isn't `approve` or `decline`, and reject a second ruling on
an application that already has one. Validators must not mutate state and must
not `await`.

## Part C — The query handler (TODO 3)

`GetStatus` is already written. It needs the `[WorkflowQuery]` attribute.

Queries read state without changing it and never appear in the event history,
so you can poll one as often as you like. They must be synchronous.

## Part D — Wait, and escalate (TODO 4)

`Workflow.WaitConditionAsync` takes a predicate and an optional timeout. With a
timeout it returns `bool` — `true` if the condition came true, `false` if the
timeout won:

```csharp
var decided = await Workflow.WaitConditionAsync(
    () => _decision is not null,
    EscalationThreshold);
```

If it timed out, set `_escalated`, log a warning, then **wait again with no
timeout**.

That second wait is the whole design. Don't fail the workflow and don't
auto-decline — an application nobody looked at is not a declined application.
The timer raises a flag; it does not make the decision for the desk.

---

## Run it

Start the worker in [button label="Terminal 1 - Worker" background="#444CE7"](tab-0):

```bash,run
dotnet run -- worker
```

Submit an application in [button label="Terminal 2 - Starter" background="#444CE7"](tab-1):

```bash,run
dotnet run -- starter
```

It prints a workflow id like `loan-APP-976545DF` and stops at
`awaiting-decision`. **Copy that id** — the next commands need it.

In [button label="Terminal 3 - Underwriter" background="#444CE7"](tab-2), read its state without
touching it:

```bash
dotnet run -- status loan-APP-XXXXXXXX
```

Try an invalid ruling. Your validator should refuse it:

```bash
dotnet run -- decide loan-APP-XXXXXXXX maybe
```

```
[Decide] Rejected: Decision must be 'approve' or 'decline', got 'maybe'
```

Now rule on it properly:

```bash
dotnet run -- decide loan-APP-XXXXXXXX approve
```

Terminal 2 unblocks immediately and prints the result.

---

## See the escalation fire

`EscalationThreshold` is two minutes. Start another application and leave it
alone. After two minutes the worker logs:

```
warn: No decision on APP-XXXXXXXX after 2 minutes — escalating to the senior desk. Still waiting.
```

Query it again: `Escalated` is now `True` and `Stage` reads `escalated`. The
application is still live — rule on it whenever you like and it is accepted
exactly as before.

Impatient? Set `EscalationThreshold` to `TimeSpan.FromSeconds(20)` and restart
the worker.

---

## Observe

Open [button label="Temporal UI" background="#444CE7"](tab-4) and find your application. You'll see
`WorkflowExecutionUpdateAccepted` and `WorkflowExecutionUpdateCompleted` around
your ruling.

What you will *not* see is your status query, or the rejected `maybe` ruling.
Queries never touch history, and a validator rejection is turned away before it
gets there.

Click **Check** once an application has been decided.
