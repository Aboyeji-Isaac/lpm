# RFC-0003: Event Model

- Status: Accepted
- Phase: Specification

## 1. Abstract

Living Project Memory (LPM) preserves engineering execution continuity by recording meaningful execution evidence incrementally. This RFC defines the conceptual model for that evidence: immutable events, optional operation lifecycles reconstructed from events, and a minimal set of semantic relationships among events.

An LPM event records one meaningful assertion about engineering execution. It may express intent, an action or attempt, an observation, an outcome, validation, a decision, or a correction or qualification. Events do not form a general activity log, workflow engine, or knowledge graph. Their sole purpose is to preserve the evidence needed to understand what is known, what remains uncertain, and how interrupted engineering work may safely continue.

This RFC defines semantics, not representation. It does not select a schema, identifier format, storage or persistence mechanism, API, ordering algorithm, recovery algorithm, or vendor integration.

## 2. Motivation

An actor may be interrupted between any two meaningful steps. It may declare an intended change, start modifying a file, observe that the file changed, begin validation, and disappear before a candidate record of the validation result becomes a durable event. A task-level summary written afterward cannot preserve evidence that was never durably recorded.

LPM therefore needs an event model that preserves distinctions which recovery depends upon:

- intent is not execution;
- an attempted action is not proof of its external effect;
- an observed effect is not proof of correctness;
- validation starting is not validation completing;
- absence of a durable outcome is not evidence of failure or success.

The model must support humans, AI agents, and tools; concurrent and independent activity; correction without rewriting history; and evidence about external systems without making LPM authoritative over those systems.

## 3. Normative Language

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this RFC express protocol requirements:

- **MUST** and **MUST NOT** identify requirements necessary for conformance to this event model.
- **SHOULD** and **SHOULD NOT** identify strong recommendations that may be departed from only when the tradeoff is understood and justified.
- **MAY** identifies a permitted but optional choice.

These words are normative only when capitalized.

## 4. Terminology

- **Occurrence** is an action, declaration, observation, outcome, decision, or other happening in engineering execution, whether or not LPM represents it.
- **Candidate record** is a representation submitted to or held by an LPM recording process before it has become durable. It is not an event and is not part of durable recorded history.
- **Event** is an immutable LPM record that has become durable and asserts exactly one meaningful change or clarification in engineering execution state.
- **Assertion** is the semantic claim made by an event. An event records that the claim entered LPM history; the claim's relationship to external reality depends on its role, provenance, and supporting evidence.
- **Actor** is a human, AI agent, tool, or other participant to which an intent, action, observation, decision, or recorded assertion is attributed.
- **Source** is the immediate origin from which an assertion or its supporting information was obtained. A source may be an actor, tool, external system, or prior event.
- **Provenance** is the information needed to understand who or what asserted or supplied evidence, under what relevant circumstances, and with what relationship to the occurrence being recorded.
- **Execution context** is the task, operation, target, project, session, or other bounded setting needed to interpret an event's execution significance.
- **Operation** is a bounded unit of attempted engineering activity whose progress or outcome may matter to continuity.
- **Subject reference** identifies the bounded artifact, external-state referent, task, or other contextual subject about which an event makes its assertion. It is not an event relationship and does not turn that subject into LPM-managed state.
- **Relationship** is a defined semantic connection between events. A relationship adds interpretation without modifying either related historical event. Association between an event and an operation is execution context, not an event relationship.
- **Derived execution state** is an interpretation computed from durable events and their relationships. It is not historical evidence itself.

## 5. Event Model

An LPM event MUST represent exactly one immutable, meaningful assertion about engineering execution. A consumer MUST be able to determine the assertion's semantic role, relevant execution context, provenance, and any relationships necessary to interpret its continuity significance.

An event exists to answer a recovery-relevant question such as what was intended, attempted, observed, established, validated, decided, corrected, or left unresolved. It differs from:

- a **log line**, because inclusion is based on execution-continuity value rather than routine emission;
- **telemetry**, because its purpose is reconstructing engineering execution rather than measuring system behavior;
- **conversation history**, because protocol meaning cannot depend on access to a dialogue or private model context;
- **arbitrary agent output**, because an event carries defined semantics and provenance;
- a **filesystem or machine snapshot**, because an event records bounded execution evidence rather than complete state; and
- **generic activity tracking**, because activity without material recovery value is outside the model.

An event MAY contain human-readable explanation, but essential protocol meaning MUST NOT depend on ambiguous prose alone. Context or supporting explanation MUST NOT introduce additional normative assertions. Independently recovery-relevant transitions, including intent, initiation, observed effect, terminal outcome, and validation, MUST be separate events.

Events MAY participate in operation lifecycles and MAY relate explicitly to other events. Those capabilities MUST remain limited to engineering execution continuity and MUST NOT turn LPM into a general-purpose knowledge representation or workflow system.

## 6. Meaningful Event Boundary

An occurrence is meaningful when omitting its durable record could materially impair a later actor's ability to determine what was intended, attempted, observed, changed, decided, completed, validated, or left unresolved, or to choose a defensible continuation.

Conforming recording behavior MUST preserve occurrences that satisfy this criterion at boundaries where interruption would otherwise create material ambiguity. It MUST NOT rely solely on elapsed time, periodic snapshots, task completion, or conversation retention.

An occurrence SHOULD be recorded separately when it:

- changes the semantic role of what is known, such as moving from intent to attempt or from attempt to outcome;
- establishes independently useful evidence;
- can be interrupted before a later consequential stage;
- may succeed or fail independently of related activity;
- changes task direction, constraints, or an established decision; or
- corrects, qualifies, contradicts, or supersedes prior recorded evidence.

Related low-level occurrences MAY be represented by one event when they form one coherent assertion and separating them would not improve recovery. The model does not require every tool invocation, file read, keystroke, conversational token, internal reasoning step, or incidental observation to be recorded. Reading a file, for example, is ordinarily less important than a consequential discovery obtained from it.

Later conformance specifications MAY establish more specific recording thresholds for particular classes of work, but MUST preserve this recovery-centered criterion.

## 7. Semantic Roles

Every event MUST expose exactly one semantic role. The minimum roles are:

- **Intent:** an actor declares a planned, proposed, expected, or recommended action or direction. It does not establish that execution began.
- **Action or attempt:** an actor started, attempted, progressed, or performed a bounded activity. The assertion MUST identify which of these execution stages it establishes. **Started** establishes initiation; **attempted** establishes an effort whose completion is not established; **progressed** establishes an intermediate execution step; and **performed** establishes that the bounded action itself was carried out. None alone establishes the intended external effect, correctness, or successful validation.
- **Observation:** an actor or source reports evidence perceived about execution or external state. It does not by itself establish who caused that state or that the observation remains current.
- **Outcome:** evidence establishes a terminal resolution of an operation as successful completion, unsuccessful completion or failure, or cancellation. An outcome applies only to the identified operation and meaning supported by its evidence. Completion of an operation does not by itself establish that its effect was correct or successfully validated.
- **Validation:** evidence evaluates a prior action, effect, artifact, or outcome against stated or inferable criteria. It is distinct from performing the underlying action.
- **Decision:** an actor establishes a choice, constraint, or direction that governs relevant subsequent work. It does not prove that work followed the decision.
- **Correction or qualification:** new evidence revises how an earlier assertion should be understood without rewriting it. A correction asserts that the earlier assertion is materially erroneous; a qualification narrows or contextualizes it without necessarily rejecting it. The assertion MUST identify which variant it expresses.

These are semantic roles, not a required concrete event-type hierarchy. A later representation MAY express roles as types, assertions, or another unambiguous mechanism. Additional specialized roles MAY be defined only when their semantics cannot be represented clearly by these roles and they materially support execution continuity.

Every event MUST contain exactly one normative assertion with exactly one semantic role. Explanation and context MAY support that assertion but MUST NOT assert another role. Independently recovery-relevant claims with different roles or execution stages MUST be recorded as separate events.

### 7.1 Required distinctions

The model enforces the following inequalities:

- **intent is not action:** “I intend to modify `auth.ts`” does not establish that modification began;
- **action is not observed effect:** “I attempted to modify `auth.ts`” does not establish what content exists in the file;
- **observed effect is not validation:** “`auth.ts` was observed as modified” does not establish that the modification is correct;
- **validation start is not validation outcome:** “validation began” does not establish whether it completed or what it found; and
- **missing outcome is not failure:** absence of a durable terminal outcome establishes neither failure, success, nor cancellation.

A validation outcome MAY support a conclusion about a change only when the relationship between the validation evidence and that change is represented. Successful execution of a validation command MUST NOT automatically be interpreted as successful validation unless the recorded evidence supports that meaning.

## 8. Operations

An operation is a conceptual unit of attempted work for which distinguishing initiation, progress, and outcome may be necessary to resume safely. Examples may include a file modification, command execution, test run, dependency installation, migration, or external request. These examples do not define an operation taxonomy.

An operation is not a mutable historical object. Its lifecycle MUST be reconstructed from immutable events concerning it. Events MAY declare intent for an operation, establish that it started or progressed, report observations, and establish an outcome or validation.

Events concerning the same operation MUST be associable without relying only on adjacency or a total event order. An operation's derived lifecycle MUST NOT overwrite its constituent events. A later event MAY add an outcome, correction, or qualification, but it cannot retroactively change what an earlier event asserted.

An operation MUST have a stable conceptual identity sufficient to associate its events across interruption and concurrency. Two events concern the same operation only when their execution context identifies the same bounded attempt; sharing an actor, task, command, target, or intended effect is not sufficient by itself. Separate attempts are distinct operations even when they repeat the same activity. Operation identity MUST remain stable for the operation lifecycle, MUST distinguish concurrent attempts, and MUST NOT imply event ordering. This RFC does not select an operation identifier format or generation method.

Not every event belongs to an operation. A repository discovery, user redirection, decision, or correction MAY be meaningful independently. LPM MUST NOT force all execution evidence into operation lifecycles.

## 9. Operation Outcomes and Incomplete Operations

An operation is **resolved** only when durable history contains an applicable terminal outcome event. The minimum terminal outcome semantics are:

- **successful completion:** the operation completed with the success meaning asserted by the outcome evidence;
- **unsuccessful completion or failure:** the operation terminated without the asserted operation success; and
- **cancellation:** the operation was deliberately terminated without completion.

These outcomes are mutually distinguishable. They resolve only the identified operation. Successful completion MUST NOT be interpreted as correctness or successful validation unless a separate validation event supplies that evidence.

When durable history establishes that an operation started but contains no applicable durable terminal outcome, a consumer MUST derive that the operation is **unresolved** or its **outcome is unknown**. Unresolved is derived uncertainty, not a terminal outcome.

That derived condition MUST NOT be represented as evidence that the operation failed, succeeded, completed, or was cancelled. Those conclusions require affirmative durable evidence. An unexpected interruption need not and often cannot produce an interruption event.

A later actor MAY inspect relevant external state and record a new observation. That observation establishes only what its own assertion, context, and provenance support. It MUST NOT retroactively establish that the original operation completed or rewrite its historical outcome unless the evidence specifically establishes that claim. For example, a test observed to pass after restart does not by itself prove that the pre-interruption validation completed successfully.

## 10. Identity, Actor, Provenance, and Context

### 10.1 Event identity

Every event MUST have a stable identity that distinguishes it within the scope in which histories may be interpreted or combined. Event identity MUST:

- remain stable for the lifetime of the event;
- support unambiguous reference by later events and derived state;
- remain distinct from mutable presentation or storage position;
- avoid implying event order merely by identity; and
- support detection or safe handling of accidental identity reuse.

The protocol semantics MUST define the scope and meaning of identity. Recording the same occurrence twice produces duplicate representations of one asserted occurrence and MUST NOT be interpreted as two executions. Independent assertions about the same state remain distinct events with distinct provenance, even when their claims agree. Corroborating observations are independent observation events whose agreement may strengthen a derived conclusion but does not merge their identities. Producers and consumers MUST preserve these distinctions; this RFC does not define a deduplication mechanism, identifier format, or generation method.

### 10.2 Actor, source, and provenance

Events MUST identify or characterize the actor and immediate source to the extent necessary to interpret the assertion and assess its evidentiary meaning. At minimum, provenance MUST distinguish an actor assertion, evidence produced by a tool, and evidence reported by an external system, and MUST retain the attribution and relevant context needed to avoid presenting one as another. Actor identity MUST support humans, AI agents, tools, and other relevant participants without requiring a particular vendor or product.

The actor and source MAY differ. An AI agent may record an observation supplied by a test tool, or a human may record information reported by an external service. Provenance MUST preserve that distinction where conflating it would misstate who acted or what supplied the evidence.

Provenance establishes attribution and evidentiary context; it does not by itself establish truth, trustworthiness, authorization, or current external state.

### 10.3 Execution context

An event MUST carry or relate to enough execution context to make its significance understandable without the original conversation. Relevant context MAY include the project, task, operation, target, execution session, governing request, or affected artifact.

No single context category is universally required beyond what is necessary for unambiguous interpretation. Context MUST NOT turn LPM into a project-management or session-replay system.

## 11. Event Relationships

Event relationships connect durable events to other durable events. Association with an operation and reference to an artifact, task, or external state are execution context, not graph relationships. Relationships MUST have defined direction and semantics, MUST serve execution continuity, and MUST NOT mutate, erase, or silently reinterpret either event.

The required core relationship vocabulary is:

- **follows:** one event is known to follow another in a stated, relevant ordering domain. It does not imply causation, dependency, immediate adjacency, or any other ordering domain.
- **result-of:** a result event reports an intermediate or terminal result of the operation evidenced by an identified action event. The relationship itself does not imply completion, success, failure, cancellation, validation, or achievement of a broader intent. Terminality comes only from the result event’s outcome role and applicable outcome semantics.
- **validates:** a validation event evaluates the assertion or work evidenced by an identified event. It does not rewrite that event, validate unrelated work, or imply that the underlying operation and validation are the same occurrence.
- **corrects:** a later event asserts that an earlier assertion is materially erroneous and supplies corrective evidence. The earlier event remains unchanged.
- **qualifies:** a later event narrows, contextualizes, or limits an earlier assertion without necessarily rejecting it. The earlier event remains unchanged.

Operation association is required where multiple events concern the same operation, but it is part of execution context rather than this relationship vocabulary. Subject references MAY identify artifacts or external-state referents only to bound the execution evidence being asserted; they MUST NOT create arbitrary semantic links or make LPM authoritative over those subjects.

Relationships such as responds-to, depends-on, and supersedes are not core semantics. Later specifications MAY define narrowly scoped extensions when a demonstrated execution-continuity need cannot be expressed through operation context, the core relationships, or a new intent, decision, correction, or qualification event. Extensions MUST NOT change core relationship meanings or turn LPM into a general-purpose graph or workflow protocol.

Mere temporal proximity MUST NOT be treated as an implicit result, validation, correction, or qualification relationship.

## 12. Ordering and Concurrency

The event model MUST preserve ordering relationships that are known and material to execution continuity. It MUST support partially ordered histories, concurrent actors, independent event streams, and cases where the relative order of events is unknown. It MUST NOT require or fabricate one total global order. An implementation need not record every kind of ordering for every event; it MUST preserve only ordering evidence that is known, available, and relevant to correct interpretation.

Consumers MUST distinguish, where relevant:

- **execution order:** the known order in which occurrences happened;
- **observation order:** the order in which occurrences or states were perceived;
- **recording order:** the order in which assertions entered a recording process;
- **durability order:** the order or frontier relationships through which records became durable; and
- **causal relationship:** an evidentiary claim that one event or condition contributed to, prompted, or was required by another.

One of these relationships MUST NOT be interpreted as another without supporting protocol evidence. In particular, recording order does not necessarily prove execution order, and ordering alone does not prove causation.

Events from one actor or stream MAY have a known local order while remaining unordered relative to events in another stream. When independent histories are combined, event and operation identities, core relationships, provenance, established ordering, explicitly unknown ordering, and applicable durability context MUST retain their meanings. Combination MUST NOT invent a global latest execution point, collapse distinct operations, or convert unknown order into known order. This RFC does not define a merge algorithm.

This RFC defines ordering semantics only. It does not select timestamps, counters, logical clocks, sequence-number strategies, merge procedures, or concurrency algorithms.

## 13. Durability

The model distinguishes three conditions:

1. an occurrence happened in engineering execution;
2. a candidate record representing an assertion entered an LPM recording process; and
3. that candidate record became durable and thereby became an event.

These conditions MUST NOT be treated as interchangeable. A candidate record MUST NOT be treated as part of recorded history before durability. Consistent with RFC-0002, only durable events are historical evidence from which LPM guarantees recovery.

An occurrence may affect external state without any candidate record being created or becoming durable. A candidate record may also enter recording without becoming an event before interruption. Recovery MUST represent both possibilities as uncertainty when durable events do not resolve them.

Histories MAY have multiple durable frontiers across actors or independent streams. The event model MUST NOT require one global durability frontier. This RFC does not define persistence technology, acknowledgement mechanics, transactional behavior, or the strength of a durability guarantee.

## 14. Actions, Observations, and External State

An action or attempt event asserts the specific execution stage established: requested, started, attempted, progressed, or performed. An observation event separately asserts that an actor or source perceived evidence about something. An action MUST NOT automatically be treated as proof of its intended effect, and an observation MUST NOT automatically be treated as proof of who caused the observed state.

Events MAY refer to filesystems, Git repositories, databases, deployed infrastructure, or external services. Such an event MUST express a bounded action or observation with enough provenance and context to interpret the evidence. It MUST NOT make LPM authoritative over the referenced system.

For example, an event may establish that a command requesting a deployment was invoked, while another event may establish that a service reported a deployment state. Neither assertion alone proves the other, and the reported state may later become stale. Later conflicting external observations MUST be preserved as additional evidence rather than forced into agreement.

## 15. Corrections and Conflicting Evidence

A durable event MUST NOT be rewritten or removed during normal operation because it is later found to be mistaken, incomplete, stale, or contradicted. New evidence MUST be recorded as a new event and related to the earlier event using the applicable core relationship; a new intent, decision, observation, or validation remains a distinct assertion.

A correction changes the supported interpretation of history; it does not change the historical fact that the earlier assertion was recorded. Conflicting evidence MAY remain unresolved. Consumers MUST surface material conflict and MUST NOT silently choose one assertion merely because it was recorded later.

## 16. Derived Execution State

Current execution state MUST be derived from durable events and their defined relationships. A derived conclusion MUST remain traceable to the events that support, contradict, qualify, or leave it uncertain.

Derived state MAY identify recorded intent, known attempts, observed effects, established outcomes, validation status, decisions, unresolved operations, conflicts, and relevant durable frontiers. It MUST:

- preserve material uncertainty and unknown ordering;
- distinguish absence of evidence from affirmative evidence;
- avoid inventing historical events or unsupported outcomes;
- avoid rewriting or silently overriding history; and
- remain subordinate to the durable history as the record of what LPM recorded.

Derived state MAY change when candidate records become durable events and those events enter durable history. It MUST NOT become a second independent mutable source of truth. This RFC does not define a derivation or recovery algorithm and does not require one unique conclusion where evidence permits multiple interpretations.

## 17. Human and Machine Interpretation

The event model MUST enable both a human and a compatible machine consumer to determine, without access to the original AI conversation:

- the event's stable identity;
- its semantic role and substantive assertion;
- the actor, source, and material provenance;
- the execution context necessary to understand its significance;
- the operation or target concerned, when applicable;
- its defined relationships to relevant events;
- the ordering claims that are known and those that remain unknown;
- any outcome, validation result, correction, qualification, conflict, or uncertainty it establishes; and
- that it is part of durable history under an applicable durability boundary.

Human-readable explanation SHOULD make the event's recovery significance inspectable. Machine interpretation MUST rely on defined semantics rather than natural-language inference alone. This RFC does not prescribe concrete fields, syntax, or serialization.

## 18. Interruption Scenario

Consider an agent performing a complex ten-step task. Durable history establishes the task context, a repository discovery, an intent to modify `auth.ts`, a separate action event establishing that editing began, observation evidence establishing the meaningful modification, and an action event establishing the start of a validation operation. The computer then shuts down before any candidate record of a validation outcome becomes a durable event.

Under this model, recovery can determine:

- which task and relevant context were recorded;
- what the agent intended;
- that editing began and what modification evidence was recorded with its provenance;
- that validation was started;
- that no durable validation outcome is known;
- that validation success, failure, and cancellation are all unsupported; and
- which artifact and validation operation should be inspected or repeated before continuation.

The validation operation is derived as unresolved. No failure event, success event, or synthetic interruption event is invented. If the recovering actor examines the artifact or reruns validation, meaningful occurrences may produce candidate records, which become durable events only if they satisfy the applicable durability requirement. The complete interpretation is available without the original conversation.

## 19. Conformance Requirements

A protocol design conforming to this RFC:

- MUST represent meaningful execution evidence as immutable durable events;
- MUST distinguish intent, action or attempt, observation, outcome, validation, decision, and correction or qualification semantics;
- MUST preserve the required inequalities among intent, action, observed effect, validation, and missing outcomes;
- MUST reconstruct operation lifecycles from events rather than mutable historical operation state;
- MUST allow unresolved operations to be identified without inferring a terminal outcome;
- MUST provide stable event and operation identities, actor or source attribution, minimum provenance, and sufficient execution context;
- MUST support the five core event relationships and preserve their defined meanings;
- MUST support concurrency, independent streams, partial ordering, and unknown ordering without requiring a total global order;
- MUST distinguish occurrences, candidate records, and durable events;
- MUST limit LPM authority to what its durable history records;
- MUST preserve corrections and conflicting evidence without rewriting history;
- MUST make derived state traceable and uncertainty-preserving;
- MUST provide defined semantics suitable for human and machine interpretation; and
- MUST keep all of these capabilities focused on engineering execution continuity.

A conforming design SHOULD minimize event noise and relationship complexity. It MUST NOT require LPM to record arbitrary activity merely because that activity can be observed.

## 20. Non-Goals

This RFC does not define:

- JSON, JSONL, or any other serialization;
- a database, filesystem layout, or storage format;
- an API or command-line interface;
- a programming language or reference implementation;
- a concrete event schema or exhaustive event taxonomy;
- a concrete event or operation identifier format;
- a timestamp, sequence-number, or logical-clock strategy;
- an ordering, concurrency, synchronization, or merge algorithm;
- a persistence mechanism or durability acknowledgement mechanism;
- a recovery or state-derivation algorithm;
- indexing, caching, replication, retention, compaction, migration, or archival;
- authentication, authorization, or trust policy;
- a vendor-specific AI or tool integration;
- complete conversation, machine, IDE, filesystem, or external-system state;
- a generic activity, telemetry, knowledge-graph, project-management, or workflow protocol; or
- autonomous execution or guaranteed autonomous recovery.

## 21. Open Questions

The following semantic questions remain for review of this RFC or compatible conformance work:

- Which operation boundaries must all conforming producers recognize, and which may be profile-specific?
- When does an external observation provide sufficient evidence to resolve an earlier operation whose outcome was unknown?
- Which conflicts can be resolved by defined relationship semantics, and which must remain explicitly unresolved?
- What minimum validation rules are necessary for interoperable event histories?
- Which extension relationships, if any, demonstrate sufficient execution-continuity value to standardize later?

These questions MUST be resolved without weakening the core distinctions and invariants in this RFC.

## 22. Future Work and Deferred Decisions

Later RFCs or conformance specifications may define concrete representations, identifier generation, event and operation taxonomies, ordering mechanisms, durability guarantees and acknowledgements, persistence, synchronization, recovery and derivation procedures, external-system evidence profiles, conformance tests, and integrations.

Those specifications MUST preserve the semantic roles, immutability, uncertainty, relationship meanings, authority boundary, and concurrency neutrality established here. Implementation choices MAY provide stronger guarantees but MUST NOT present them as universal LPM guarantees.

## 23. Relationship to RFC-0001 and RFC-0002

RFC-0001 defines the execution-continuity problem: meaningful engineering history must survive interruption sufficiently to support safe resumption. This RFC supplies the conceptual event model needed to record that history.

RFC-0002 defines the governing principles: incremental recording, append-only immutable history, intent and fact separation, evidence-based recovery, explicit durability boundaries, limited authority, vendor neutrality, human and machine readability, and derived state. This RFC makes those principles concrete at the semantic level without selecting implementation mechanisms.

If this RFC conflicts with the purpose or guarantees of RFC-0001 or the principles of RFC-0002, the accepted RFCs govern and this RFC must be corrected.
