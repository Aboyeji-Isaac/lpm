# RFC-0002: Core Principles

- Status: Accepted
- Phase: Specification

## 1. Summary

Living Project Memory (LPM) preserves engineering execution continuity across interruptions. This RFC defines the principles and invariants that an LPM protocol must follow to serve that purpose correctly.

LPM records meaningful changes in engineering execution state incrementally, preserves recorded history without rewriting it during normal operation, distinguishes intent from fact, and makes current execution state derivable from that history. Its guarantees stop at the durability boundary: LPM can establish what it successfully recorded, but it cannot guarantee knowledge of activity that occurred without a durable record.

These are protocol principles, not implementation decisions. This RFC does not select an event schema, storage format, API, command-line interface, integration, or recovery algorithm.

## 2. Motivation

An AI coding agent may explore a repository, modify files, create a utility, run commands, and begin another change before its session or machine disappears without warning. On return, the central problem is uncertainty about engineering execution state: what happened, what completed, what remains unresolved, and where execution stopped.

An end-of-task summary cannot solve this problem because the interruption may occur before the task ends. Conversation history alone is not sufficiently durable, portable, or vendor-neutral. Observable project state may reveal what currently exists, but it does not necessarily explain the actions, outcomes, decisions, or intent that produced it.

LPM therefore needs governing rules that make its recorded history useful and trustworthy under interruption while expressing uncertainty honestly. Those rules must promote recoverability without claiming perfect observation or turning LPM into a substitute for version control, conversation history, or project management.

## 3. Relationship to RFC-0001

RFC-0001 defines the execution-continuity problem and the outcomes LPM is intended to support. It answers why LPM exists.

This RFC builds on that accepted problem statement by defining the protocol properties required to address it. It answers what rules LPM must follow. It does not expand LPM into a general-purpose AI memory system: memory and history are mechanisms used to preserve engineering execution continuity.

If this RFC appears to broaden the purpose or guarantees established by RFC-0001, RFC-0001 governs the scope and this RFC must be corrected.

## 4. Terminology and Normative Language

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this RFC express protocol requirements:

- **MUST** and **MUST NOT** identify requirements necessary for conformance to these principles.
- **SHOULD** and **SHOULD NOT** identify strong recommendations that may be departed from only when the tradeoff is understood and justified.
- **MAY** identifies a permitted but optional choice.

These words are normative only when capitalized.

For this RFC:

- **Engineering execution state** is the known facts, declared intent, outcomes, decisions, task progress, and unresolved operations needed to understand where engineering work stopped and how it can safely continue.
- **Meaningful engineering event** is an action, observation, outcome, decision, or declaration that materially changes or clarifies the known engineering execution state. Meaningfulness is based on its value to execution continuity, not on elapsed time or volume of activity. Later protocol specifications and conformance profiles define objective criteria for evaluating this requirement.
- **Intent** is a statement about an action, outcome, or direction that an actor plans, expects, proposes, or recommends. Intent does not establish that the action occurred.
- **Fact** is a recorded assertion that an action, observation, outcome, or decision occurred or was established. A recorded fact is evidence of what LPM recorded; its relationship to external state may still require verification.
- **Historical event** is a durable record of a meaningful engineering event.
- **Recorded history** is a body of historical events whose ordering relationships are defined by the protocol.
- **Derived state** is a current interpretation produced from recorded history. It is not itself a replacement for that history.
- **Durability boundary** identifies the frontier or frontiers, relative to the recorded history available at the time of interruption, through which events are known to have been successfully recorded and made durable. It does not require one global latest durable execution point. Activity not represented within the durable recorded history may be unknown to LPM.
- **Observable project state** is state independently inspectable outside LPM, including filesystem, Git, database, deployed, or other external-system state.
- **Normal operation** is conforming use of the protocol, excluding explicitly governed administrative processes such as retention, migration, or repair, whose rules are outside this RFC.

## 5. Core Principles

### 5.1 Incremental Execution Recording

LPM history MUST be recorded incrementally around meaningful changes in engineering execution state. Recording MUST be event-driven rather than defined primarily by a fixed time interval, and the protocol MUST NOT depend on an end-of-task receipt as its sole or primary record.

A meaningful event is one whose absence could materially impair a later actor's ability to determine what was attempted, observed, changed, decided, completed, or left unresolved. Examples may include consequential repository observations, changes to project artifacts, command or test outcomes, established decisions, and transitions in task progress. These examples illustrate the principle; they do not define the event taxonomy.

Incremental does not mean recording every keystroke, tool-internal operation, conversational token, minute, or fixed number of minutes. Implementations SHOULD record near enough to meaningful events to reduce the amount of execution state exposed to interruption, while avoiding records that add no useful recovery evidence.

RFC-0003 will define the event model and the more precise boundaries of recordable events.

This RFC establishes the normative requirement to record meaningful engineering events. RFC-0003 and later conformance specifications will define objective criteria for evaluating which events must be recorded in particular protocol and conformance contexts.

### 5.2 Append-Only History

Normal operation MUST extend recorded history by appending historical events. It MUST NOT replace prior events merely because understanding has changed, a mistake was discovered, or newer information supersedes an earlier assertion.

This preserves the historical evidence that LPM recorded and makes changes in knowledge visible through whatever ordering relationships the protocol defines. Append-only history does not by itself define physical storage, global or total ordering, retention, compaction, or transport behavior.

### 5.3 Immutable Historical Facts

Once a historical event has been durably recorded, normal operation MUST NOT silently rewrite that event. Later information that corrects, qualifies, contradicts, or supersedes recorded information MUST be represented without altering the original historical record.

Immutability applies to the historical record, not to conclusions derived from it. Derived state MAY change when later events add evidence or correct prior understanding.

This RFC does not define the correction mechanism, event relationships, or validation rules. Those belong to the event model and related RFCs.

### 5.4 Intent and Fact Separation

LPM MUST distinguish declared intent from recorded fact. A plan to modify authentication middleware and a record that a particular middleware file was modified represent different execution states and MUST NOT be treated as interchangeable.

Intent MAY be valuable recovery evidence because it explains direction and possible next steps. It MUST NOT be treated as proof that the intended action started, completed, succeeded, or changed external state.

Likewise, the absence of a recorded result after an intent MUST NOT by itself be treated as proof that the action failed or did not occur. It establishes only that LPM lacks a corresponding durable record. Recovery may need to compare history with observable project state.

### 5.5 Recoverability

LPM history MUST preserve the information necessary for a human or compatible tool to reconstruct what is known within the durable recorded history, identify material uncertainty, and form a defensible basis for continuing work.

Recoverability is evidence-based, not a promise of perfect or autonomous recovery. A recovery result MUST distinguish supported conclusions from uncertainty and MUST NOT claim knowledge that the recorded history and any explicitly examined external evidence do not support.

LPM SHOULD make completed work, known outcomes, established decisions, unresolved operations, and relevant known execution frontiers distinguishable where the recorded evidence supports those distinctions. It MUST NOT infer completion or failure solely from silence.

This RFC establishes the normative requirement for sufficient recovery evidence. RFC-0003 and later conformance specifications will define objective criteria for evaluating whether particular records and histories satisfy that requirement.

### 5.6 Durability Boundary

LPM can guarantee recovery only from execution evidence that was successfully recorded and made durable. Its continuity guarantees MUST be expressed relative to the durable recorded history available at the time of interruption and MUST NOT imply that every action performed before an interruption was captured.

The durability boundary MAY comprise multiple acknowledged or durable frontiers when recorded history includes multiple actors, concurrent execution, independent event streams, or events with protocol-defined ordering relationships. This RFC does not define how such frontiers relate, how events are ordered, or how concurrency is represented.

An interruption may occur after an external action but before its corresponding event becomes durable. In that case, observable state may have changed while LPM has no durable knowledge of the action. Implementations and recovery processes MUST represent this possibility as uncertainty rather than conceal it.

Implementations SHOULD minimize the exposure between a meaningful event and its durable recording. This principle does not prescribe a persistence technology, transactional design, or durability guarantee; such guarantees must be specified by later RFCs or implementations.

### 5.7 Authoritative Recorded History

Recorded history is authoritative for the engineering execution facts that LPM has recorded: it establishes what assertions entered LPM history and in what preserved sequence or relationship the protocol defines.

LPM history is NOT automatically authoritative over observable project state. It MUST NOT be treated as replacing or controlling the filesystem, Git repository, database, deployed infrastructure, or any other external system. A recorded assertion about external state may be stale, incomplete, mistaken, or followed by unrecorded activity.

Recovery MAY compare recorded history with observable project state. A discrepancy MUST be surfaced as evidence requiring interpretation; neither source should be silently forced to agree with the other.

### 5.8 Agent and Vendor Neutrality

The LPM protocol MUST NOT require a particular AI model, coding agent, vendor, IDE, hosting platform, or proprietary conversation system. It MUST permit interoperable participation by different AI coding agents, tools, and humans.

Vendor-specific integrations MAY exist, but protocol meaning and recoverability MUST remain available without dependence on a single vendor's private context or service.

### 5.9 Human and Machine Readability

LPM records MUST have defined semantics that can be interpreted by both humans and machines. A developer should be able to inspect and reason about the execution evidence, while tools should be able to process it without depending on ambiguous prose alone.

Representations SHOULD support clear communication of evidence, intent, uncertainty, and relationships without requiring access to the original AI conversation. This principle does not select a serialization, storage format, presentation, or schema.

### 5.10 Derived State

Current execution state MUST be derivable from recorded history rather than maintained as a second independent source of truth. Any summary, projection, index, checkpoint, or other current-state view MUST remain traceable to the historical evidence from which it was derived.

Derived state MAY be cached or materialized, but it MUST NOT silently override or rewrite recorded history. If derived state can no longer be reconciled with its source history, the history governs what LPM recorded and the derived state must be treated as stale or invalid.

This RFC does not define a derivation or recovery algorithm, nor does it require one unique interpretation when the evidence is incomplete. Derived state must preserve relevant uncertainty.

## 6. Principle Interactions

The principles operate as a system rather than as independent preferences:

1. Incremental recording limits how much meaningful execution evidence is exposed when an interruption occurs.
2. Append-only and immutable history preserve the integrity and protocol-defined ordering relationships of durable evidence.
3. Separation of intent and fact prevents plans from being mistaken for outcomes.
4. The explicit durability boundary prevents missing evidence from being presented as certainty.
5. Authoritative recorded history identifies what LPM can establish while permitting comparison with external reality.
6. Derived state turns preserved evidence into a usable view of current execution without creating a competing history.
7. Human and machine readability and vendor neutrality allow that evidence to remain useful across actors, tools, sessions, and environments.

Together, incremental recording, append-only immutable history, and evidence-based recovery provide the foundation for engineering execution continuity. None is sufficient alone: frequent records that can be rewritten are unreliable; immutable history captured only at task completion is interruption-fragile; preserved history without honest interpretation does not make work safely resumable.

These principles do not eliminate uncertainty. They constrain LPM to make the known execution state reconstructable and the unknown state explicit.

## 7. Non-Goals

This RFC does not make LPM:

- a general-purpose AI memory or knowledge system;
- a replacement for Git or any other version-control system;
- a replacement for AI conversation history;
- a project-management, issue-tracking, or workflow-orchestration system;
- a complete audit of every machine, tool, or user action;
- a filesystem, database, deployment, or environment snapshot system;
- a guarantee of preservation for unrecorded activity;
- a guarantee of perfect, deterministic, or autonomous recovery; or
- an authority that overrides independently observable project state.

## 8. Constraints

A future LPM protocol design conforming to this RFC:

- MUST preserve the distinction between historical events and derived current state;
- MUST preserve the distinction between intent, fact, and absence of evidence;
- MUST support incremental recording without requiring timer-driven checkpoints or task completion;
- MUST preserve durably recorded events from silent mutation during normal operation;
- MUST expose, rather than erase, uncertainty created by the durability boundary;
- MUST keep protocol semantics independent of any particular agent or vendor;
- MUST allow both human inspection and deterministic machine processing of defined semantics;
- MUST NOT require LPM to become authoritative over external systems; and
- SHOULD avoid protocol requirements that add activity records without material value to execution continuity.

Specific implementations MAY provide stronger durability, validation, ordering, or recovery guarantees than these principles require, provided they do not contradict them or present implementation-specific guarantees as universal protocol properties.

## 9. Open Questions

The following questions remain intentionally unresolved:

- What exact event taxonomy captures meaningful engineering execution without excessive noise?
- At what boundaries should actions, observations, outcomes, decisions, and progress changes become separate events?
- How should facts, intent, outcomes, uncertainty, and their relationships be represented?
- How should corrections, qualifications, and contradictions refer to earlier history?
- What event identity and ordering guarantees are required, including across concurrent actors?
- How should event histories and their ordering relationships be validated?
- What durability guarantees and acknowledgement semantics should conforming implementations expose?
- How should apparently incomplete operations be represented without treating a missing result as proof of failure?
- How should derived state be reconstructed when evidence is incomplete, contradictory, or inconsistent with observable project state?
- How may histories be retained, migrated, compacted, or archived without violating append-only and immutability principles?
- How should protocol conformance and interoperability be tested?

These questions require later specification. Their presence does not weaken the principles; it marks where implementation-independent rules end.

## 10. Relationship to Future RFCs

RFC-0003, the Event Model, is expected to define what events exist, how meaningful engineering events are represented, how intent and fact are distinguished, and how corrections and relationships are expressed. It must preserve the principles in this RFC without treating the illustrative examples here as a complete taxonomy.

Later RFCs may define ordering and validation rules, persistence and durability semantics, state derivation and recovery, APIs, command-line behavior, storage representations, integrations, and conformance testing. Those RFCs MUST treat this document as a constraint: implementation choices may strengthen these principles but must not silently weaken or redefine them.
