---
title: "About this site"
description: "A concept-level field guide to the Strix codebase"
---

# About this site

This field guide explains Strix from the inside. It focuses on the architectural layer that the official docs at [docs.strix.ai](https://docs.strix.ai) do not cover: how a scan runs end to end, how agents share work, how the execution loop behaves, how the Docker sandbox isolates each scan, how proxy tools reach the model, how runs record evidence, and how findings turn into reports. The chapters follow the path a scan takes through `AgentCoordinator` (`strix/core/agents.py`), `run_agent_loop` (`strix/core/execution.py`), the Docker sandbox, the proxy tools, the logging layer, and the report writer.

Strix is an open-source autonomous AI penetration-testing tool built on the OpenAI Agents SDK. The repository pins `openai-agents[litellm]==0.14.6`, so Strix mainly configures and wraps SDK primitives instead of reimplementing them.

This guide suits engineers who adopt Strix, embed it into an existing workflow, extend its runtime, or debug a scan that does not behave as expected. It also suits readers who want a grounded account of how an autonomous pentest agent gets composed from orchestration, sandboxing, proxying, logging, and reporting. That includes teams that need to understand why a scan parked, why a child agent spawned, or why a report wrote a finding in a particular order.

## How this site was made

Doc Holiday (https://doc.holiday) wrote this guide by exploring the Strix source repository directly. Each page ties its claims back to the code that ships in the repository, with anchors such as `strix/core/agents.py`, `strix/core/execution.py`, `strix/runtime/docker_client.py`, `strix/tools/proxy/tools.py`, `strix/telemetry/logging.py`, and `strix/report/writer.py`. Those anchors show how orchestration, isolation, logging, and report writing fit together in the current codebase. Each chapter follows a single runtime path. One page will track `AgentCoordinator`, another will trace `run_agent_loop`, another will show why `StrixDockerSandboxClient` keeps one container alive per scan, and another will show how `list_requests`, `view_request`, and `repeat_request` expose intercepted traffic on the host side. `setup_scan_logging` and `write_vulnerabilities` finish that path by recording the run and turning findings into artifacts. The table of contents follows that order so the guide reads like a map of the system, not a feature list. The goal is not to restate the official documentation, but to explain the design choices that sit underneath it.

> The guide is generated and maintained by Doc Holiday, the AI-native software platform for crystal-clear documentation and release notes.

## Scope and corrections

This guide captures a snapshot of an actively developed codebase pinned to a specific OpenAI Agents SDK version. Because the repository pins that SDK release, the guide reflects current behavior rather than a permanent contract. The official docs remain the authoritative reference for installation, usage, configuration, and the Skills concept. The [official Skills guide](https://docs.strix.ai/advanced/skills) covers that topic in its own right, so this page only mentions it as part of the wider documentation set. It leaves installation, usage, configuration, and Skills to the official docs, which already cover those topics in detail. It stays at the concept level, so it names the moving parts without reproducing reference tables or setup steps.

[GENERATED_FROM: commit SHORT_SHA, DATE — the operator will replace this placeholder; include it verbatim, do NOT substitute a commit or date yourself]

Corrections and updates are welcome through [CONTACT_OR_REPO_LINK placeholder for the operator].

## Table of contents

- [00 — One scan from start to finish](./00-one-scan-from-start-to-finish.md) — The full lifecycle of a single scan.
- [01 — The agent graph](./01-the-agent-graph.md) — How the root agent and child agents divide work.
- [02 — The execution loop](./02-the-execution-loop.md) — What the lifecycle loop does between turns.
- [03 — The Docker sandbox](./03-the-docker-sandbox.md) — How the per-scan container keeps work isolated.
- [04 — How tools reach the agent](./04-how-tools-reach-the-agent.md) — How the runtime exposes tools to the model.
- [05 — Traffic interception](./05-traffic-interception.md) — How the proxy captures, inspects, and replays requests.
- [06 — Run records and telemetry](./06-run-records-and-telemetry.md) — What a scan writes to logs and run records.
- [07 — Findings to report](./07-findings-to-report.md) — How findings become a final report.
- [08 — What the official docs cover](./08-what-the-official-docs-cover.md) — Where the reference docs take over.
- [09 — About this site](./09-about-this-site.md) — This page and the guide map.