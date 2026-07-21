# The graph of agents

## Overview

Strix models multi-agent work as a tree with a single root agent, `strix`, whose `parent=None`. Each child agent gets a short 8-hex id under that root, and each agent id carries an `AgentRuntime` with `session`, `task`, and `wake` handles. `AgentCoordinator` in `strix/core/agents.py` owns the non-LLM control plane for that tree; it is not itself an agent. The root agent `strix` speaks in natural language and steers the branches.

See also [The agent loop](./03-the-agent-loop.md), which explains the lifecycle rules that this page describes. For broader product framing, use [Skills](https://docs.strix.ai/advanced/skills) and [scan modes](https://docs.strix.ai/usage/scan-modes).

The diagram below shows the root, three children, and the flows that matter most: spawn, message, and completion.

```mermaid
flowchart TD
  Root[strix]
  A[9a1b2c3d]
  B[4e5f6a7b]
  C[c8d9e0f1]

  Root -->|create_agent| A
  Root -->|create_agent| B
  Root -->|create_agent| C
  A -->|send_message_to_agent| B
  B -->|agent_finish| Root
  C -->|agent_finish| Root
```

## The coordinator owns the tree

The coordinator keeps the tree real. `AgentCoordinator` stores the live status for every node, maps each child to its parent, and keeps the runtime snapshots that let Strix rebuild a scan later. It does this because a multi-agent scan needs a single control plane that can answer questions about membership, ancestry, and liveness without asking the LLM itself. The consequence is simple: the tree has a stable source of truth, and every other subsystem reads from it.

## Spawning stays detached

Child creation splits across `create_agent` in `strix/tools/agents_graph/tools.py` and `spawn_child_agent` in `strix/core/execution.py`. The graph tool asks the coordinator to register the new child, and the execution layer starts the child as a detached asyncio task so the parent keeps moving. This keeps the parent free to orchestrate while the child works in parallel, and it prevents the tree from turning into a single blocking call chain.

## Messages travel through agent sessions

Strix does not route agent communication through a separate message bus. `send_message_to_agent` and the coordinator's `send` path append the new message into the target agent's SDK session, update pending counts, and wake the waiting agent; `wait_for_message` and `consume_pending` pick that work back up. That choice keeps inter-agent traffic inside the same session model the SDK already uses, so the runtime avoids a second delivery system. The consequence is direct and predictable: a message becomes the next thing the target agent sees when it resumes.

## Children inherit context at birth

`child_initial_input` in `strix/core/inputs.py` packages the parent's relevant history, the child's identity, and the assigned task into one initial user message. The child starts with that inherited context instead of a blank slate because the parent already knows which details matter for the subtask. The result is a narrower prompt surface, less duplicated explanation, and a child that begins with the right framing for its branch of work.

## Termination stays tool gated

`agent_finish` ends a child agent, and `finish_scan` in `strix/tools/finish/tool.py` ends the root scan. Both tools reject premature completion while children still run, and `finish_scan` keeps that guard at the top level so the root cannot claim success before the tree finishes. That constraint exists because the scan only completes when the coordinator can account for every live descendant. The next page, [The agent loop](./03-the-agent-loop.md), covers the loop-level enforcement that keeps this guard in place.

## Resume rebuilds the live branches

`respawn_subagents` in `strix/core/execution.py` restores non-terminal descendants from the persisted coordinator snapshot when a scan resumes. The runtime rebuilds the same tree shape instead of starting over because a paused branch still has identity, state, and unfinished work. The consequence is continuity: finished branches stay finished, live branches come back, and the coordinator can pick up the same ancestry map it had before the pause.

## The tree is real control state

Subtree walks and cancellation stay first-class because `AgentCoordinator` exposes `cancel_descendants` and `cancel_descendants_graceful`. Those methods show that the tree carries load-bearing control state: Strix can inspect or stop one branch without touching the rest of the scan.

## Where to look in the code

- `strix/core/runner.py` — creates the root agent `strix`, installs `spawn_child_agent`, and respawns non-terminal children on resume.
- `strix/core/agents.py` — holds `AgentCoordinator`, `AgentRuntime`, status maps, parent links, pending counts, wake events, snapshots, and shutdown state.
- `strix/core/execution.py` — starts detached child tasks, opens child sessions, and rebuilds children from persisted state with `respawn_subagents`.
- `strix/core/inputs.py` — builds `child_initial_input`, which carries inherited context into a child's first user message.
- `strix/tools/agents_graph/tools.py` — exposes `view_agent_graph`, `create_agent`, `send_message_to_agent`, `wait_for_message`, `agent_finish`, and `stop_agent`.
- `strix/tools/finish/tool.py` and `strix/agents/factory.py` — enforce root-only scan completion and attach the lifecycle tools to root and child `SandboxAgent`s.