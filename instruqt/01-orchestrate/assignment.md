---
slug: orchestrate
id: gl85syn78jtj
type: challenge
title: 'Challenge 1: Orchestrating Distributed Work'
teaser: Turn three unreliable service calls into a durable workflow — then kill the
  worker and watch it survive.
notes:
- type: text
  contents: |-
    # Three calls, any of which can fail

    An auto-loan application has to clear three steps before anyone
    can rule on it: pull the applicant's credit file, screen them for
    fraud, and score the result for risk.

    Each call crosses a network. Each one can time out, return a 503,
    or succeed slowly. Written by hand, this becomes retry loops,
    backoff math, and a persistence layer to survive a crash.

    Temporal owns all of that. Your code describes the happy path.
- type: text
  contents: |-
    # What durable execution actually means

    Halfway through this challenge you will kill the worker while an
    application is in flight, then start a new one.

    The application finishes. Not restarts — finishes, from where it
    was. Its state was never in the worker's memory; it was in the
    event history on the server.

    That is the idea the rest of the workshop builds on.
tabs:
- id: yfoazsbhtogk
  title: Terminal 1 - Worker
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: lzmuke0f5ezm
  title: Terminal 2 - Starter
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: 2trbdn8da5cp
  title: VS Code
  type: service
  hostname: workshop
  port: 8080
- id: ekcqczjwsqo8
  title: Temporal UI
  type: service
  hostname: workshop
  port: 8233
difficulty: basic
timelimit: 1800
enhanced_loading: null
---

# Challenge 1: Orchestrating Distributed Work

> [!NOTE]
> **Tabs:** [button label="Terminal 1 - Worker" background="#444CE7"](tab-0) · [button label="Terminal 2 - Starter" background="#444CE7"](tab-1) · [button label="VS Code" background="#444CE7"](tab-2) · [button label="Temporal UI" background="#444CE7"](tab-3)

## Start here

Click [button label="VS Code" background="#444CE7"](tab-2). Your code is in `/root/workshop/exercise`.
The reference solution is in `/root/workshop/solution` if you want to compare.

You'll work in two files: `LoanUnderwritingActivities.cs` and
`LoanUnderwritingWorkflow.cs`. `README.md` in the same folder has the full
walkthrough, including every code snippet you need.

---

## Part A — Implement the activities (TODO 1–3)

Open `LoanUnderwritingActivities.cs`.

Inside an activity you can reach `ActivityExecutionContext.Current`, which
carries that activity's Temporal identity:

```csharp
var info   = ActivityExecutionContext.Current.Info;
var logger = ActivityExecutionContext.Current.Logger;
```

`info.Attempt` is the current attempt number — log it and you'll watch retries
happen live. `info.WorkflowId` and `info.ActivityId` together make a good
idempotency key:

```csharp
var idempotencyKey = $"{info.WorkflowId}:{info.ActivityId}";
```

**Why those two?** `ActivityId` is stable across every retry of the same call
but unique against every other call. Pass that key downstream and a retried
credit pull is recognised as a duplicate instead of charged twice. A
`Guid.NewGuid()` would change on every attempt and defeat the purpose.

For the fraud screen, a flagged applicant is a *business* outcome, not a
glitch. Throw `ApplicationFailureException(..., nonRetryable: true)` so Temporal
stops instead of retrying toward the same answer.

## Part B — Implement the workflow (TODO 1–2)

Open `LoanUnderwritingWorkflow.cs`.

Define `CreditOptions` for the flaky bureau — initial interval 5s, backoff
coefficient 3.0, max 5 attempts — and a plainer `DefaultOptions` for the other
two activities. Then write `RunAsync`: capture `Workflow.UtcNow`, log through
`Workflow.Logger`, and call the three activities in sequence.

> [!WARNING]
> **Determinism.** Inside workflow code never use `DateTime.Now`, `Random`,
> `Guid.NewGuid`, or `Task.Delay`. The workflow replays from history after any
> crash, and those four return something different each replay. Use
> `Workflow.UtcNow`, `Workflow.Random`, `Workflow.NewGuid`, and
> `Workflow.DelayAsync`. Activities have no such restriction — that is where
> the messy work belongs.

---

## Part C — Run it

Start the worker in [button label="Terminal 1 - Worker" background="#444CE7"](tab-0):

```bash,run
dotnet run -- worker
```

Submit an application in [button label="Terminal 2 - Starter" background="#444CE7"](tab-1):

```bash,run
dotnet run -- starter
```

Watch Terminal 1. The bureau fails twice and succeeds on the third attempt,
with the same idempotency key every time:

```
19:35:34 info: Pulling credit for applicant-4471 (attempt 1, idempotency key loan-APP-...:1)
19:35:34 warn: Credit bureau returned 503 on attempt 1 — Temporal will retry
19:35:39 info: Pulling credit for applicant-4471 (attempt 2, idempotency key loan-APP-...:1)
19:35:54 info: Pulling credit for applicant-4471 (attempt 3, idempotency key loan-APP-...:1)
```

---

## Part D — Kill the worker

This is the part that matters.

Run the starter again. The moment you see the **attempt 2** failure, press
`Ctrl+C` in Terminal 1 to kill the worker. Your retry policy puts the next
attempt about fifteen seconds later, so there's no rush.

Count to five, then start a new worker:

```bash,run
dotnet run -- worker
```

The application finishes. Nothing was lost, nothing was resubmitted, and you
wrote no recovery code. The state was never in the worker — it was in the event
history, and the new worker picked it up.

---

## Part E — Observe

Open [button label="Temporal UI" background="#444CE7"](tab-3) and find your workflow. Open the
`ActivityTaskStarted` event for `PullCredit` and look at `attempt`.

Notice what is *not* there: no event per failed attempt. Temporal records
retries on the started event rather than writing one event each, so a chatty
retry loop never bloats your history.

Click **Check** once an application has completed.
