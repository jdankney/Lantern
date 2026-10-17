# Lantern

> **If someone is unresponsive, not breathing normally, or you think they are overdosing, call your local emergency number now.** Don't wait for an app.

Lantern is an offline harm-reduction assistant. It sits in front of a local, open-weight language model and answers drug-safety questions from archived source material instead of the model's memory, with safety gates enforced in code.

**Status: early development. Do not rely on Lantern for safety decisions.** It has not been evaluated or reviewed by harm-reduction practitioners or clinicians yet.

## What it does

- Answers drug-safety questions using retrieved, cited source material.
- Checks substance combinations against a deterministic risk table instead of asking the model.
- Shows fixed, region-specific emergency resources from a two-press "break glass" button, or automatically when a conversation signals an emergency.
- Runs fully offline after first setup, on your own machine, for one user.

## What it will never do

Lantern refuses to help with making or extracting substances, finding sellers, beating drug tests, or dosing someone without their consent. It never gives dose numbers as advice. See [SAFETY.md](SAFETY.md) for the full requirements.

## How it works

```
message → crisis gate → scope gate → name normalization → interaction lookup
        → hybrid retrieval → local LLM → numeric gate → citations → answer
```

The model only writes explanations. Anything life-critical (combination ratings, emergency content, numbers from sources) comes from code and verified data.

## Data

Source data is not stored in this repository. On first run, Lantern downloads a signed data bundle from this project's GitHub Releases into `data/`, verifies it, and works offline from then on. When a newer bundle is released, the UI offers a **Sync changes** button.

Planned sources, pending license and permission checks: DanceSafe, TripSit, and PsychonautWiki.

## Requirements

- Python 3.11+ and [uv](https://docs.astral.sh/uv/)
- A local model server with an OpenAI-compatible API, such as Ollama, llama.cpp server, or LM Studio

## Development

```sh
uv sync
uv run pytest                   # unit tests, including every gate-level eval
uv run python -m lantern.evals  # eval summary by category
```

## Project layout

```
src/lantern/
  gates/          crisis, scope, and numeric safety gates
  emergency/      fixed emergency content and break-glass flow
  interactions/   deterministic combination-risk lookup
  substances/     street name and alias normalization
  ingest/         local source files → cited chunks
  retrieval/      embeddings, vector store, BM25, hybrid ranking
  llm/            OpenAI-compatible local model client
  prompts/        versioned system prompts
  sync/           bundle download, verification, update checks
  logs/           local encrypted interaction logs
  ui/             local web UI
  evals/          safety eval cases (cases/*.toml) and runner
  harness.py      the pipeline
corpus/           source definitions; documents stay local until permission is confirmed
tools/build_bundle/  maintainer tooling to build data bundles
tests/
```
