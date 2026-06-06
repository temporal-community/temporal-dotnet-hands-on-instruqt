---
slug: environment-setup
id: ""
type: challenge
title: "Environment Setup"
teaser: Verify the sandbox is healthy before the first exercise.
notes:
- type: text
  contents: |-
    # Welcome to Temporal .NET Hands-On

    A Temporal dev server is running in your sandbox. Five progressive
    exercises take a fragile C# pipeline to a fully durable, production-grade
    Temporal workflow system.

    Each exercise builds on the last. You'll edit code in VS Code,
    run the worker and starter in the terminals, and observe your
    workflows in the Temporal Web UI.

    The blue buttons in these instructions are clickable — use them to
    jump between tabs without hunting for them.
tabs:
- title: Terminal
  type: terminal
  hostname: workshop
  workdir: /root/workshop
- title: Temporal UI
  type: service
  hostname: workshop
  port: 8233
- title: VS Code
  type: service
  hostname: workshop
  port: 8080
difficulty: basic
timelimit: 600
enhanced_loading: null
---

# Environment Setup

> [!NOTE]
> **Tabs:** [button label="Terminal" background="#444CE7"](tab-0) · [button label="Temporal UI" background="#444CE7"](tab-1) · [button label="VS Code" background="#444CE7"](tab-2)

## Check the Temporal server

Click the [button label="Terminal" background="#444CE7"](tab-0) tab and run:

```bash,run
temporal operator cluster health
```

You should see `SERVING`. If not, wait 10 seconds and try again.

## Verify your tools

```bash,run
dotnet --version && temporal --version
```

You should see .NET 10 and a recent Temporal CLI version.

## Explore the workshop

```bash,run
ls /root/workshop/exercises
```

Five exercise directories. Each has a `practice/` folder where you'll work
and a `solution/` folder for reference.

## Open VS Code

Click the [button label="VS Code" background="#444CE7"](tab-2) tab. The C# Dev Kit extension
is pre-installed. When you open a `.cs` file IntelliSense and autocomplete will be available.

## Check the Temporal UI

Click the [button label="Temporal UI" background="#444CE7"](tab-1) tab. An empty workflow list
means the server is healthy and no workflows have run yet. You'll see your
workflows appear here as you complete each exercise.

Click **Check** when you're ready to continue.
