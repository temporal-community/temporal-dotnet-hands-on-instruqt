---
slug: child-workflows
id: pjbtapbqqrrr
type: challenge
title: 'Exercise 2: Child Workflows'
teaser: Extract document generation into a child workflow with its own lifecycle and
  retry boundary.
notes:
- type: text
  contents: |-
    # When a sub-process deserves its own history

    Your vehicle transaction now calls fraud check, payment, and title transfer
    durably. But document generation — the bill of sale, the title document —
    is a multi-step process that another team might own, or that might need to
    run independently.

    Child workflows give a sub-process its own event history, its own retry
    boundary, and its own identity in the Temporal UI. The parent just waits
    for the result.
tabs:
- id: xfamifw7zumc
  title: Terminal 1 - Worker
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: zrks7jq8tnfc
  title: Terminal 2 - Starter
  type: terminal
  hostname: workshop
  workdir: /root/workshop/exercise
- id: oxn2xz5wydm7
  title: VS Code
  type: service
  hostname: workshop
  port: 8080
- id: vefdu8xdzsrg
  title: Temporal UI
  type: service
  hostname: workshop
  port: 8233
difficulty: basic
timelimit: 2700
enhanced_loading: null
---

# Exercise 2: Child Workflows

> [!NOTE]
> **Tabs:** [button label="Terminal 1 - Worker" background="#444CE7"](tab-0) · [button label="Terminal 2 - Starter" background="#444CE7"](tab-1) · [button label="VS Code" background="#444CE7"](tab-2) · [button label="Temporal UI" background="#444CE7"](tab-3)

Open [button label="VS Code" background="#444CE7"](tab-2) and read the README for this exercise,
then work through the TODOs in the source files.

Start the worker in [button label="Terminal 1 - Worker" background="#444CE7"](tab-0):

```bash,run
dotnet run -- worker
```

Start a workflow in [button label="Terminal 2 - Starter" background="#444CE7"](tab-1):

```bash,run
dotnet run -- starter
```

In [button label="Temporal UI" background="#444CE7"](tab-3), notice that the document generation
workflow appears as a separate entry with its own event history.

Click **Check** when at least one workflow completes successfully.
