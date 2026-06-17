# AGENTS.md

## Workflow

Commit and push after every change. Don't batch multiple changes into a single commit — each logical change should be its own commit, pushed immediately. One version per commit when adding Dockerfiles.

## Project structure

Dockerfiles go in `versions/<version>/Dockerfile`. Docker images are tagged `gs-<version>`. Follow this convention — don't invent new layouts.

## Before committing

- Build the combined image (`scripts/build-combined-image <version>`) and verify
  both `gs --version` and `gs-debug --version` output the expected version. The
  combined image is the only published artifact; the normal and debug images are
  built as intermediate stages.
- Update the versions table in `README.md` when adding or removing a version.
- Never put credentials or secrets in Dockerfiles. Everything should build from public package repos or source tarballs.
