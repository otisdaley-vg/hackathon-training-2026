FROM ghcr.io/otisdaley-vg/hackathon-csharp:latest

USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends postgresql-client \
    && rm -rf /var/lib/apt/lists/*

USER vscode
