---
slug: environment-setup
id: ""
type: challenge
title: Environment Setup
teaser: Verify the sandbox is healthy before the first challenge.
notes:
- type: text
  contents: |-
    # Welcome to Temporal .NET Hands-On

    A Temporal dev server is already running in your sandbox. Three
    challenges take an auto-loan underwriting pipeline from a set of
    unreliable service calls to a durable, self-healing workflow.

    1. Orchestrate three activities and survive a worker restart
    2. Pause for an underwriter, with updates, queries and escalation
    3. Unwind committed work with the saga pattern

    You'll edit C# in VS Code, run a worker and a starter in the
    terminals, and watch what happens in the Temporal Web UI.

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

You should see `SERVING`. If not, wait ten seconds and try again.

## Verify your tools

```bash,run
dotnet --version && temporal --version
```

You should see a .NET 9 SDK and a recent Temporal CLI.

## Look at the challenges

```bash,run
ls /root/workshop/exercises
```

Three directories, one per challenge. Each has a `practice/` folder where you
work and a `solution/` folder for reference. Each challenge's **Setup** step
copies the right one into `/root/workshop/exercise` for you, so that is the
folder you will actually edit.

## Warm up the build

The NuGet packages are already baked into this image, but a first build still
takes a few seconds. Get it out of the way now:

```bash,run
cd /root/workshop/exercises/01_orchestrate/practice && dotnet build
```

You should see `Build succeeded`.

## Open the editor

Click the [button label="VS Code" background="#444CE7"](tab-2) tab.

You get C# syntax highlighting, but **no IntelliSense** — the Microsoft C#
extensions aren't available in this editor's marketplace. Every snippet you
need is written out in the challenge instructions and in the `README.md`
alongside the code, so you won't need autocomplete.

## Check the Web UI

Click the [button label="Temporal UI" background="#444CE7"](tab-1) tab. An empty workflow list
means the server is healthy and nothing has run yet. Your workflows will appear
here as you work.

Throughout the workshop the Web UI is for *observing* — you'll drive everything
from the terminal and read the results here.

Click **Check** when you're ready to start.
