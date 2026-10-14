---
slug: saga
id: cqzh9xgmqpnl
type: challenge
title: 'Challenge 3: Saga and Compensation'
teaser: Unwind committed steps in reverse when a later step fails — and learn why
  compensations must be idempotent.
notes:
- type: text
  contents: |-
    # Where Temporal's guarantee stops

    Temporal guarantees your workflow runs to completion. It cannot
    un-ring a bell in someone else's system.

    Place a credit hold, request documents, then have the funding
    partner decline the loan — and that hold is real, still sitting
    there, with no money moved and no one coming to clean it up.

    Undoing committed work is your job. The saga pattern is how you
    structure it.
- type: text
  contents: |-
    # Register the undo first

    After each step with a side effect, push a compensating action
    onto a stack. If a later step fails, unwind the stack — newest
    first.

    The subtle part: push each compensation BEFORE running the action
    it undoes. Push it afterwards and there is a window where the
    action succeeded but the undo was never registered, and the money
    stays moved.

    The cost is that you will sometimes undo something that never
    happened. That is why every compensation must be idempotent.
tabs:
- id: xzktrntgnd1q
  title: Terminal 1 - Worker
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: ifvicxgku84k
  title: Terminal 2 - Starter
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: oj5eeufxnudk
  title: VS Code
  type: service
  hostname: workshop
  port: 8080
- id: mvglt9ztazks
  title: Temporal UI
  type: service
  hostname: workshop
  port: 8233
difficulty: intermediate
timelimit: 1800
enhanced_loading: null
---

# Challenge 3: Saga and Compensation

> [!NOTE]
> **Tabs:** [button label="Terminal 1 - Worker" background="#444CE7"](tab-0) · [button label="Terminal 2 - Starter" background="#444CE7"](tab-1) · [button label="VS Code" background="#444CE7"](tab-2) · [button label="Temporal UI" background="#444CE7"](tab-3)

## Start here

Click [button label="VS Code" background="#444CE7"](tab-2) and open
`LoanUnderwritingWorkflow.cs`. The body of `RunAsync` is one TODO.

Funding an approved application takes three steps, each touching a different
external system:

| Forward step | Compensation |
|---|---|
| `PlaceCreditHoldAsync` | `ReleaseCreditHoldAsync` |
| `RequestDocumentsAsync` | `CancelDocumentRequestAsync` |
| `FundLoanAsync` | `ReverseFundingAsync` |

All six activities are already written. Your job is the orchestration.

---

## Your task

```csharp
var compensations = new Stack<Func<Task>>();
try
{
    compensations.Push(() => Run(a => a.ReleaseCreditHoldAsync(app)));
    await Run(a => a.PlaceCreditHoldAsync(app));

    // ...documents, then funding, each the same way...
}
catch
{
    while (compensations.TryPop(out var undo)) await undo();
    throw;
}
```

`Run` is a helper already defined in the class so your saga reads as the pattern
rather than as Temporal boilerplate.

> [!IMPORTANT]
> **Push each compensation before you run the action it undoes.**
>
> Push it afterwards and there is a window — the action succeeded, the push had
> not happened yet, the worker died — where the undo is lost forever and the
> money stays moved. Registering first closes that window, at the cost of
> sometimes undoing something that never happened. That is why every
> compensation here is idempotent.

A `Stack` pops in reverse order, which is what you want: release the funds
before cancelling the documents before releasing the hold.

Re-throw at the end of the `catch`. A compensated failure is still a failure.

---

## Run it

Start the worker in [button label="Terminal 1 - Worker" background="#444CE7"](tab-0):

```bash,run
dotnet run -- worker
```

Fund a loan that succeeds, in [button label="Terminal 2 - Starter" background="#444CE7"](tab-1):

```bash,run
dotnet run -- starter
```

Now one the funding partner refuses — $72,000, over its $50,000 per-loan limit:

```bash,run
dotnet run -- starter large
```

The hold and the documents commit; funding is declined. Watch Terminal 1:

```
Placed credit hold HOLD-504D6811 for $72,000.00 on APP-504D6811
Requested document package DOC-504D6811 for APP-504D6811
Funding partner declined APP-504D6811: $72,000.00 exceeds the $50,000.00 limit
APP-504D6811 failed — running 3 compensation(s) in reverse
[Compensation] No funding for APP-504D6811 to reverse (no-op)
[Compensation] Cancelled document package for APP-504D6811
[Compensation] Released credit hold on APP-504D6811
```

## Read those last three lines

They are the whole challenge.

The compensations ran newest-first. And the first one was a **no-op** — you
registered `ReverseFundingAsync` before calling `FundLoanAsync`, funding never
happened, so the undo had nothing to undo and said so.

That line is the price of the push-before-action rule, and exactly why these
compensations must be idempotent. An implementation that assumed the funding
existed would have thrown there, in the middle of cleanup, leaving the credit
hold stranded — the precise failure the saga exists to prevent.

Note too that the decline is thrown as a **non-retryable**
`ApplicationFailureException`. A business refusal is not a transient fault:
retrying it would reach the same answer more slowly while the hold sat open.

---

## Observe

Open [button label="Temporal UI" background="#444CE7"](tab-3) and find the failed application. The
history shows three forward activities, the failure, then the compensating
activities scheduled after it — all inside one workflow execution.

No orchestrator service coordinated that. No cleanup cron, no dead-letter queue.
The unwind is ordinary code in your `catch` block, made reliable because
Temporal keeps the workflow alive until the cleanup finishes.

Click **Check** once you have run both the successful and the declined paths.
