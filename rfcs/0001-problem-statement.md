# RFC-0001: Problem Statement

- Status: Accepted
- Phase: Specification
- Reconstruction note: This is a new reconstruction based on documented project design history. It is not the original lost file and has not been accepted.

## 1. Summary

AI coding agents can be highly effective during uninterrupted development sessions, but they do not reliably preserve engineering execution state when those sessions end unexpectedly.

Interruptions such as usage limits, connectivity failures, application or terminal crashes, system shutdowns, context-window exhaustion, or switching agents or machines can force developers to reconstruct progress manually before work can continue. This uncertainty can lead to repeated investigation, duplicated implementation, forgotten decisions, unnecessary token consumption, and reduced engineering productivity.

Living Project Memory (LPM) is proposed as an open, AI-agnostic protocol for preserving engineering execution continuity. In this RFC, **engineering execution state** means the known facts, declared intent, outcomes, decisions, task progress, and unresolved operations needed to understand where engineering work stopped and how it can safely continue. It does not mean complete machine or IDE state, complete conversation memory, filesystem snapshots, general project knowledge, or guaranteed autonomous recovery.

LPM must preserve history incrementally as meaningful engineering actions and observations occur so that humans and compatible tools can reconstruct the last successfully recorded execution point after an interruption.

## 2. Motivation

LPM originated from repeated interruptions during AI-assisted development on a laptop with unreliable battery power. A loss of electricity could shut down the machine and terminate an active coding session without warning.

The important insight was that the problem is not simply that an AI loses conversation context. The deeper problem is that, after an interruption, neither the developer nor a subsequent agent can confidently determine the project's execution state.

Typical questions include:

- Was a planned change actually made?
- Was an operation completed or only started?
- Which files were modified?
- Which commands and tests ran?
- Which decisions were made?
- Where did execution stop?
- What should happen next?

Conversation history alone is not a dependable, portable, or vendor-neutral answer to these questions.

## 3. Problem Statement

Current AI-assisted development workflows primarily depend on active session or conversation context. They lack a reliable, vendor-neutral mechanism for preserving factual engineering execution history across interruptions and using that history to reconstruct the state needed to resume work.

This creates an execution-continuity gap: work may have changed the project, but the evidence needed to understand the extent and outcome of that work may be incomplete, inaccessible, or mixed with unexecuted plans and predictions.

LPM addresses this gap. Its purpose is to make engineering work resumable after the loss of an AI session, machine, connection, or context without requiring the developer to reconstruct the entire prior conversation.

## 4. Real-world Scenario

An AI agent receives a multi-step engineering task. It explores a repository, makes several changes, creates a utility, runs commands, and begins another implementation step. The computer then shuts down unexpectedly.

When work resumes, the developer or a different agent cannot confidently determine:

- what was completed;
- which files changed;
- whether any change was only partial;
- which commands ran and with what outcome;
- which tests ran and with what outcome;
- which decisions were established; or
- the last known execution point.

Without durable execution history recorded during the work, recovery requires inference from the filesystem, version-control state, surviving messages, and the developer's memory. Those sources may help, but they do not consistently establish what happened or what remains incomplete.

## 5. Consequences

Loss of execution continuity can cause:

- repeated repository exploration;
- duplicated or conflicting implementation;
- inconsistent project state;
- forgotten or contradicted decisions;
- repeated commands and tests;
- wasted AI tokens and developer time;
- increased risk when partially completed operations are mistaken for completed work; and
- reduced confidence in resuming long-running tasks.

## 6. Requirements

A solution to this problem must satisfy these foundational requirements:

- record meaningful engineering activity incrementally as it occurs;
- make recording event-driven, in response to meaningful actions and observations, rather than primarily timer-driven or dependent on end-of-task receipts;
- survive unexpected interruption to the extent that previously recorded history remains available;
- distinguish recorded facts from declared intent, plans, predictions, and recommendations;
- ensure recorded historical events are not rewritten during normal operation;
- treat LPM history as the authoritative record of recorded engineering execution facts, without treating it as authoritative over the observable filesystem, repository, Git state, databases, or external systems;
- allow recovery to compare recorded history with observable current project state;
- make execution state reconstructable from durable recorded history;
- enable identification of completed and apparently incomplete work;
- remain independent of any particular AI model, agent, vendor, IDE, or hosting platform;
- provide representations suitable for both human and machine consumption;
- remain extensible without imposing unnecessary complexity.

These requirements describe desired protocol properties. They do not select an event schema, storage format, API, or implementation.

Integrating with normal development workflows and reducing duplicated investigation or implementation are desired outcomes, not strict protocol requirements.

## 7. Scope

This RFC defines the problem LPM intends to solve, the motivation for solving it, and the high-level outcomes a solution must support.

This RFC does not define:

- event structures or event types;
- storage formats or locations;
- ordering and validation rules;
- state-reconstruction algorithms;
- APIs or command-line interfaces;
- AI-agent interaction protocols;
- reference implementations; or
- product integrations.

Those decisions belong in later RFCs and specifications.

## 8. Non-Goals

LPM is not intended to:

- replace version control systems such as Git;
- replace AI conversation history;
- become a project-management system;
- automatically write source code;
- guarantee preservation or recovery of an action that was not successfully recorded before an interruption;
- infer that declared intent proves an action occurred; or
- depend on a proprietary AI platform.

## 9. Success Criteria

LPM should eventually allow developers and compatible agents to:

- resume interrupted work with greater confidence;
- reconstruct relevant execution state without requiring the original conversation;
- distinguish completed work from work that started but lacks recorded completion;
- identify the last known execution point and a defensible next step;
- reduce duplicated implementation and repeated investigation;
- transfer execution history between compatible agents and environments; and
- improve the reliability of long-running AI-assisted development.

The protocol succeeds when its recorded history provides enough trustworthy evidence for a human or compatible agent to explain what is known, what remains uncertain, and how work can safely continue after an interruption. Its durability boundary is the last successfully recorded execution point; later unrecorded activity may need to be inferred from observable current project state and cannot be guaranteed by LPM.

## 10. Open Questions

- What constitutes a meaningful execution event?
- How should facts and intent be represented and related?
- How should event identity, actors, tasks, sessions, and timestamps be modeled?
- What ordering guarantees are required?
- How should incomplete operations be identified without treating missing evidence as proof?
- How should event sequences be validated?
- How should current execution state be reconstructed from history?
- How should agents and humans interact with LPM during normal work and recovery?
- How should protocol compliance be tested?
- How should large histories be managed while preserving authoritative history?
- What durability guarantees can implementations provide across different failure modes?
