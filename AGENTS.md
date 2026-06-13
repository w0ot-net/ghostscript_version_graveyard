# AGENTS.md

## Workflow

Commit and push after every change. Don't batch multiple changes into a single commit — each logical change should be its own commit, pushed immediately. One version per commit when adding Dockerfiles.

## Project structure

Dockerfiles go in `versions/<version>/Dockerfile`. Docker images are tagged `gs-<version>`. Follow this convention — don't invent new layouts.

## Before committing

- Build the Docker image and verify `gs --version` outputs the expected version.
- Update the versions table in `README.md` when adding or removing a version.
- Never put credentials or secrets in Dockerfiles. Everything should build from public package repos or source tarballs.
