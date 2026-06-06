# ABOUTME: Sandbox image for the Temporal .NET Hands-On workshop.
# Bakes in: .NET 10 SDK, Temporal CLI, code-server + C# Dev Kit,
# all five exercise source trees (practice + solution), NuGet packages
# pre-restored for all ten projects.
#
# Build context: repo root. The GHA workflow checks out
# temporal-community/temporal-dotnet-hands-on alongside this Dockerfile
# at ./temporal-dotnet-hands-on/ before docker build.
#
# The track-level setup-workshop script stages /opt/workshop -> /root/workshop,
# starts Temporal dev server and code-server, and waits for health.

FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV WORKSPACE=/opt/workshop
ENV PYTHONUNBUFFERED=1

# ── 1. System packages ────────────────────────────────────────────────────────
RUN echo "deb http://us-central1.gce.archive.ubuntu.com/ubuntu jammy main restricted universe multiverse" > /etc/apt/sources.list \
    && echo "deb http://us-central1.gce.archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list \
    && echo "deb http://us-central1.gce.archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list \
    && echo "deb http://security.ubuntu.com/ubuntu jammy-security main restricted universe multiverse" >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        curl \
        wget \
        git \
        unzip \
        build-essential \
        ca-certificates \
        gnupg \
        bash-completion \
        apt-transport-https \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ── 2. .NET 10 SDK ────────────────────────────────────────────────────────────
RUN wget -q https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb \
    && dpkg -i packages-microsoft-prod.deb \
    && rm packages-microsoft-prod.deb \
    && apt-get update \
    && apt-get install -y dotnet-sdk-10.0 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV DOTNET_CLI_TELEMETRY_OPTOUT=1
ENV DOTNET_NOLOGO=1

# ── 3. Temporal CLI ───────────────────────────────────────────────────────────
RUN curl -sSf https://temporal.download/cli.sh | sh \
    && ln -sf /root/.temporalio/bin/temporal /usr/local/bin/temporal

ENV PATH="${PATH}:/root/.temporalio/bin"

# ── 4. Workshop source ────────────────────────────────────────────────────────
# GHA workflow checks out temporal-community/temporal-dotnet-hands-on (main)
# into ./temporal-dotnet-hands-on/ in the build context.
COPY temporal-dotnet-hands-on/ /opt/workshop/

# ── 5. Pre-restore NuGet packages for all ten projects ───────────────────────
# Restoring at build time means participants never wait for package downloads.
# Each exercise has an independent practice/ and solution/ project.
RUN for dir in \
        /opt/workshop/exercises/01_converting/practice \
        /opt/workshop/exercises/01_converting/solution \
        /opt/workshop/exercises/02_child_workflows/practice \
        /opt/workshop/exercises/02_child_workflows/solution \
        /opt/workshop/exercises/03_parallel_activities/practice \
        /opt/workshop/exercises/03_parallel_activities/solution \
        /opt/workshop/exercises/04_signals_queries/practice \
        /opt/workshop/exercises/04_signals_queries/solution \
        /opt/workshop/exercises/05_saga/practice \
        /opt/workshop/exercises/05_saga/solution; do \
        echo "Restoring $dir..." && dotnet restore "$dir" -q || true; \
    done

# ── 6. code-server + C# Dev Kit ──────────────────────────────────────────────
RUN curl -fsSL https://code-server.dev/install.sh | sh \
    && code-server --install-extension ms-dotnettools.csdevkit \
    && mkdir -p /root/.local/share/code-server/User \
    && printf '{\n\
    "workbench.colorTheme": "Default Dark+",\n\
    "workbench.startupEditor": "none",\n\
    "security.workspace.trust.enabled": false,\n\
    "editor.fontSize": 14,\n\
    "editor.tabSize": 4,\n\
    "editor.formatOnSave": true,\n\
    "workbench.activityBar.visible": false,\n\
    "workbench.statusBar.visible": true,\n\
    "dotnet.inlayHints.enableInlayHintsForParameters": false\n\
}\n' > /root/.local/share/code-server/User/settings.json

# ── 7. Shell environment ──────────────────────────────────────────────────────
RUN echo '' >> /etc/bash.bashrc \
    && echo '# Workshop environment' >> /etc/bash.bashrc \
    && echo 'export PATH=$PATH:/root/.temporalio/bin:/usr/local/bin' >> /etc/bash.bashrc \
    && echo 'export TEMPORAL_ADDRESS="127.0.0.1:7233"' >> /etc/bash.bashrc \
    && echo 'export DOTNET_CLI_TELEMETRY_OPTOUT=1' >> /etc/bash.bashrc \
    && echo 'export DOTNET_NOLOGO=1' >> /etc/bash.bashrc \
    && echo '[ -f /usr/share/bash-completion/bash_completion ] && . /usr/share/bash-completion/bash_completion' >> /etc/bash.bashrc \
    && echo '' >> /root/.bashrc \
    && echo '[ -f /etc/bash.bashrc ] && . /etc/bash.bashrc' >> /root/.bashrc

WORKDIR /root
