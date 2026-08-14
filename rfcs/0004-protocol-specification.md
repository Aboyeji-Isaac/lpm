# RFC-0004: Protocol Specification

- Status: Draft
- Phase: Specification

## 1. Abstract

This RFC defines the first normative protocol contract for Living Project Memory (LPM). It specifies the externally observable semantic guarantees required of a conforming LPM protocol implementation while remaining independent of representation, storage, interfaces, and implementation technology.

RFC-0001 defines the execution-continuity problem, RFC-0002 defines the governing principles, and RFC-0003 defines the event model. This RFC makes those accepted decisions enforceable as producer, durable-history, and consumer conformance requirements.

## 2. Scope and Conformance

This RFC governs how meaningful engineering execution evidence becomes durable history and how that history may be interpreted. It defines protocol conformance, not an implementation architecture.

A **producer** creates candidate records from meaningful occurrences. A **history conformance responsibility** determines whether candidate records are eligible to become durable events and preserves conforming durable history. A **consumer** interprets durable history and may derive execution state. These are conformance responsibilities, not required architectural components; one implementation MAY perform multiple responsibilities.

An implementation conforms to this RFC only if every protocol role it claims to implement satisfies the applicable MUST and MUST NOT requirements. Technology-specific behavior MAY vary, but externally observable protocol semantics MUST remain interoperable and consistent with this RFC and RFC-0001 through RFC-0003.

## 3. Normative Language

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** express protocol requirements:

- **MUST** and **MUST NOT** identify requirements necessary for conformance.
- **SHOULD** and **SHOULD NOT** identify strong recommendations that may be departed from only when the tradeoff is understood and justified.
- **MAY** identifies a permitted optional choice.

These words are normative only when capitalized.

## 4. Protocol Objects and Lifecycle

This RFC uses the protocol objects established by RFC-0003:

- **Occurrence:** a happening in engineering execution, whether or not LPM represents it.
- **Candidate record:** a pre-durability representation of one asserted occurrence. It is not an event or part of durable history.
- **Durable event:** the immutable event created when an eligible candidate record satisfies the applicable durability requirement and enters durable history. This RFC uses **event** to mean durable event.
- **Durable history:** the body of durable events and their preserved protocol-defined relationships, ordering evidence, provenance, and durability context.
- **Operation:** a bounded unit of attempted engineering activity, conceptually grouped through the execution context of related events. It is not a mutable historical object.
- **Relationship:** one of the defined semantic connections between durable events.
- **Derived execution state:** an interpretation computed from durable history. It is not historical evidence.

The protocol lifecycle is:

1. an occurrence happens;
2. a producer MAY create a candidate record representing one meaningful assertion about it;
3. an eligible candidate record MAY satisfy an applicable durability requirement and thereby become a durable event;
4. the durable event enters durable history; and
5. consumers MAY derive execution state from durable history.

These stages MUST NOT be conflated. An occurrence is not proof that a candidate record exists. A candidate record is not a durable event. A candidate record MUST NOT participate in guaranteed recovery history before it becomes durable. Only durable events belong to the history from which LPM guarantees recovery.

## 5. Incremental Recording Contract

Producers MUST create candidate records incrementally around meaningful changes or clarifications in engineering execution state when omission could materially impair later recovery. Recording MUST be driven by recovery-relevant occurrences rather than primarily by elapsed time, periodic snapshots, task completion, or conversation retention.

Producers MUST preserve independently recovery-relevant transitions as separate candidate records. They MUST NOT require every tool invocation, file read, keystroke, conversational token, internal reasoning step, or incidental observation to be represented.

Recording behavior SHOULD minimize the interval in which a meaningful occurrence has happened but no corresponding durable event exists. This requirement does not guarantee that every occurrence will be recorded or made durable, and it does not define a timing threshold or persistence mechanism.

## 6. Event Conformance

A candidate record is eligible to become a conforming durable event only if a consumer can determine, without relying on private conversation context:

- a stable event identity;
- exactly one normative assertion;
- exactly one semantic role;
- the actor and immediate source at the conceptual level required by this RFC;
- sufficient execution context to interpret the assertion;
- provenance necessary to distinguish the evidentiary source;
- any operation association or subject reference necessary for interpretation;
- every applicable core relationship and its direction;
- known, relevant ordering evidence without unsupported ordering claims; and
- the assertion's bounded meaning and uncertainty.

A conforming event MUST be meaningful under RFC-0003's execution-continuity criterion. Supporting explanation MAY accompany the assertion, but it MUST NOT introduce another normative assertion or make essential semantics depend on ambiguous prose.

A core relationship is **applicable** when an event's semantic assertion and context satisfy that relationship's applicability condition in Section 9. Applicability does not require relationships between arbitrary event pairs.

An event asserting or observing external state MUST remain a bounded assertion. It MUST NOT claim that LPM owns, controls, or remains authoritative over that external state.

Once an event enters durable history, its historical content, identity, provenance, execution context, subject references, and relationships MUST remain immutable during normal operation.

## 7. Semantic Role Conformance

Every event MUST express exactly one of the semantic roles defined by RFC-0003. Correction and qualification are independently distinguishable variants of one role category.

### 7.1 Intent

An intent event records a planned, proposed, expected, or recommended action or direction. It MUST NOT establish that the action was requested, started, attempted, performed, completed, or externally effective.

### 7.2 Action or Attempt

An action-or-attempt event MUST identify the execution stage it establishes:

- **started** establishes initiation;
- **attempted** establishes an effort whose completion is not established;
- **progressed** establishes intermediate execution;
- **performed** establishes that the bounded action itself was carried out; or
- **requested** establishes that execution was requested but not that another actor or system performed it.

None of these stages alone establishes the intended external effect, successful completion, correctness, or validation. A validation start MUST be distinguishable from a validation outcome.

### 7.3 Observation

An observation event records evidence perceived by an actor or source. It MUST identify the observed subject sufficiently for recovery and MUST preserve the provenance necessary to interpret the evidence. It MUST NOT by itself establish who caused an observed state, that the state remains current, or that LPM is authoritative over it.

### 7.4 Outcome

An outcome event records a terminal resolution of an identified operation as:

- successful completion;
- unsuccessful completion or failure; or
- cancellation.

A terminal outcome MUST itself be a durable outcome-role event whose assertion explicitly establishes the relevant terminal state. That assertion and its required provenance constitute the minimum protocol-level affirmative durable evidence. A separate observation event is not universally required.

An actor assertion is sufficient at the base protocol level only when the actor is legitimately the source of the asserted outcome. When a terminal outcome depends on an observed external result, a conformance profile MAY require supporting observation evidence. Trust, authentication, authorization, and stronger evidence-quality policies remain deferred.

Successful completion establishes only the success meaning of the identified operation. It MUST NOT by itself establish correctness or successful validation. Absence of a durable terminal outcome event is never affirmative evidence of success, failure, cancellation, or completion.

Intermediate results are not outcome events merely because they are related to an operation. An operation without an applicable durable terminal outcome MUST remain unresolved or outcome unknown.

### 7.5 Validation

A validation event records an evaluation and MUST identify the bounded assertion, work, artifact, outcome, or other subject evaluated through execution context or a subject reference. It MUST preserve the validation meaning supported by its evidence. It MUST NOT imply that the underlying action and validation are the same occurrence or that unrelated work was validated.

When a validation event makes a normative claim about the validity, correctness, or acceptability of an assertion or work evidenced by a prior durable event, `validates` is applicable and MUST connect the validation event to that durable event. An artifact, file, command, work item, or other contextual subject is not itself a `validates` relationship target. If no prior durable event represents the evaluated assertion or work, the validation event MAY identify only its bounded subject and context without a `validates` relationship.

Execution success of a validation activity MUST NOT be interpreted as validation success unless the event's evidence establishes that conclusion. Validation success MUST NOT be generalized beyond the evaluated subject and criteria supported by the event.

### 7.6 Decision

A decision event records an established choice, constraint, or direction relevant to subsequent work. It MUST NOT establish that later work followed or implemented the decision.

### 7.7 Correction or Qualification

A correction event asserts that an earlier event's assertion is materially erroneous and supplies corrective evidence. A qualification event narrows or contextualizes an earlier assertion without necessarily rejecting it. The event MUST identify which variant it expresses and MUST relate to the affected event using `corrects` or `qualifies` as applicable.

Neither variant modifies, erases, or replaces the referenced historical event.

## 8. Operation Conformance

An operation MUST be represented through the execution context of durable events concerning the same bounded attempt. It MUST NOT be maintained as a mutable historical record whose state can override those events.

An operation MUST have a stable conceptual identity distinct from all event identities. Events concern the same operation only when their execution context identifies the same bounded attempt. Shared actors, tasks, commands, targets, or intended effects are insufficient by themselves. Repeated and concurrent attempts MUST remain distinct operations.

An operation lifecycle MAY be derived from intent, action-or-attempt, observation, outcome, validation, correction, and qualification events associated with it. A **result event** is only a descriptive term for an event that records a result; it is not an additional semantic role. An intermediate result MUST use the applicable existing role: observation for observed output or state, or action-or-attempt for execution progress. A terminal result MUST use the outcome role. Intermediate progress or results MUST NOT be interpreted as terminal merely because they concern the operation.

An operation is resolved only when an applicable durable outcome event establishes successful completion, unsuccessful completion or failure, or cancellation. Each terminal conclusion requires the affirmative durable evidence defined in Section 7.4. If durable history establishes that an operation started but contains no applicable durable terminal outcome, consumers MUST derive unresolved or outcome unknown.

Absence of a terminal outcome MUST NOT be interpreted as success, failure, cancellation, completion, non-execution, or an interruption event.

## 9. Relationship Conformance

The mandatory core relationships are `follows`, `result-of`, `validates`, `corrects`, and `qualifies`. They connect durable events to durable events, have defined direction, and MUST NOT modify either endpoint.

### 9.1 `follows`

`follows` is applicable when a producer asserts a known, material ordering relationship between two durable events within a stated ordering domain. It MUST NOT be added merely because two events coexist in history.

`follows` asserts that one event follows another within a stated, relevant ordering domain. Both endpoints MAY have any semantic role for which that ordering claim is meaningful.

Consumers MAY rely only on the stated ordering domain. `follows` MUST NOT imply causation, dependency, immediate adjacency, ordering in another domain, or a global total order.

### 9.2 `result-of`

`result-of` is applicable when an event normatively asserts a result of an operation evidenced by a prior durable action-or-attempt event. The source and target MUST concern the same operation.

A **result event** is a descriptive term, not a semantic role. It connects to the action-or-attempt event evidencing the associated operation. An intermediate result MUST have the observation role when it records observed output or state, or the action-or-attempt role when it records execution progress. A terminal result MUST have the outcome role.

`result-of` itself MUST NOT imply completion, success, failure, cancellation, validation, or fulfillment of a broader intent. Terminality comes only from the result event's outcome role and applicable outcome semantics.

### 9.3 `validates`

`validates` is applicable when a validation event makes a normative claim about the validity, correctness, or acceptability of an assertion or work evidenced by a prior durable event. It connects the validation event to that durable event. Its source endpoint MUST be a validation event, and its target MUST be the event evidencing the evaluated assertion or work.

Artifacts, files, commands, work items, and other contextual subjects MAY be identified through execution context or subject references, but MUST NOT be `validates` relationship targets. If no prior durable event represents the evaluated assertion or work, `validates` is not applicable and the validation event MAY remain conforming by identifying its bounded subject and context.

`validates` MUST NOT rewrite the target, imply that unrelated work was evaluated, or imply that the validation and target describe the same occurrence.

### 9.4 `corrects`

`corrects` is applicable to every correction-variant event and MUST identify the durable event being corrected.

`corrects` connects a correction event to the earlier event it identifies as materially erroneous. Its source endpoint MUST express the correction variant.

Consumers MAY use the correction as later evidence when deriving state, but the relationship MUST NOT modify or erase the target event.

### 9.5 `qualifies`

`qualifies` is applicable to every qualification-variant event and MUST identify the durable event being qualified.

`qualifies` connects a qualification event to the earlier event whose assertion it narrows or contextualizes. Its source endpoint MUST express the qualification variant.

The relationship MUST NOT modify or erase the target event and MUST NOT be interpreted as rejecting it unless separate evidence establishes rejection.

No additional relationship is mandatory under this RFC. Extensions MAY be defined later only for demonstrated execution-continuity needs and MUST NOT alter the meaning of these core relationships.

## 10. Ordering and Concurrency

A conforming durable history MUST preserve known ordering relationships that are available and material to correct execution interpretation. It MUST distinguish, where relevant:

- execution order;
- observation order;
- recording order;
- durability order; and
- causal or other protocol-defined relationships.

One kind of order MUST NOT be inferred from another without supporting evidence. Recording order does not necessarily establish execution order, and ordering does not establish causation.

The protocol MUST support partial ordering, concurrent events, concurrent actors, independent event streams, unknown ordering, and histories formed by combining independent histories. It MUST NOT require or fabricate a global total order or one global latest execution point.

An implementation is not required to represent every ordering kind for every event. It MUST preserve only known, available, and relevant ordering evidence. Preserving unknown ordering means not inventing an ordering relationship where none is supported; it does not require explicit unknown-order relationships between every unordered pair.

Newly learned ordering information MUST NOT mutate an existing durable event or be silently attached to it as though it had always been recorded. It MUST be represented through a permitted additive event and relationship that preserve the original events unchanged.

When histories are combined, event and operation identities, operation associations, relationships, provenance, established ordering, unknown ordering, and applicable durability context MUST retain their meanings. Combination MUST NOT collapse distinct events or operations or invent ordering. This RFC does not define how histories are combined.

## 11. Identity, Duplicates, and Uniqueness

Event identity MUST remain stable after entry into durable history, MUST support unambiguous reference, MUST remain independent of presentation or storage position, and MUST NOT imply ordering. A durable event identity MUST NOT be reused for a different event.

Operation identity MUST remain stable for its derived lifecycle and distinct from event identity. It MUST distinguish repeated and concurrent attempts and MUST survive history combination without collapsing unrelated operations.

If the same occurrence is represented more than once, consumers MUST NOT interpret duplicate representations as multiple executions. Independent assertions about the same state remain distinct events with distinct identities and provenance. Corroborating observations remain distinct observation events; their agreement MAY strengthen a derived conclusion but does not merge their identities.

This RFC defines identity and duplicate semantics, not identifier formats or duplicate-detection procedures.

## 12. Provenance and External Evidence

Every event MUST preserve the minimum provenance necessary to distinguish:

- an assertion made by an actor;
- evidence produced by a tool; and
- evidence reported by an external system.

Provenance MUST retain the attribution and execution context necessary to avoid presenting one category as another. Actor and source MAY differ and MUST remain distinguishable when conflating them would misstate who acted or what supplied the evidence.

An event about a filesystem, version-control system, database, deployed environment, or external service records a bounded assertion or observation about that system. It establishes that LPM durably recorded the assertion, not that the external state remains true or that LPM controls the system.

Later external observations are new evidence. They MUST NOT retroactively establish an earlier operation's outcome unless their own evidence specifically supports that historical claim.

## 13. Durability Contract

Conforming implementations MUST distinguish an occurrence, a candidate record, and a durable event. A candidate record becomes an event only after satisfying the applicable durability requirement. Before that transition it MUST NOT be exposed or interpreted as part of durable history.

LPM's guaranteed recovery knowledge is limited to durable events. An occurrence MAY affect external state without a candidate record becoming durable. The protocol MUST represent resulting uncertainty honestly and MUST NOT claim knowledge of unrecorded or non-durable activity.

A durable history MAY have multiple durability frontiers across actors or independent streams. Implementations MUST preserve the applicable durability context and MUST NOT invent a single global frontier.

This contract does not define how durability is achieved, detected, or acknowledged, or the strength of a particular durability guarantee.

## 14. Append-Only History and Immutability

Normal operation MUST extend durable history by adding durable events. Once a durable event enters history:

- its identity and historical assertion MUST NOT be rewritten or reused;
- its semantic role, provenance, execution context, subject references, and operation association MUST NOT be silently changed; and
- its relationships and supported ordering claims MUST NOT be silently added, removed, or altered as though they had always been part of the event.

New knowledge, relationships, corrections, qualifications, and conflicting evidence MUST be represented by new durable events and applicable relationships. Derived interpretations MAY change as new durable events enter history, but durable history MUST remain authoritative for what LPM recorded.

Administrative retention, migration, repair, and compaction remain outside this RFC. Nothing in those deferred processes may be assumed to permit silent historical rewriting.

## 15. Candidate Validity and Rejection

The history conformance responsibility MUST determine universal protocol-level eligibility before a candidate record becomes a durable event. Universal eligibility covers the structural and semantic rules in this RFC. Conformance profiles MAY define objective sufficiency thresholds for meaningfulness, context, provenance, and evidence within particular engineering domains. A base implementation MUST NOT claim that those profile-specific thresholds are universally determined by this RFC. A candidate record MUST NOT become a conforming event if it has any of the following defects:

- no meaningful execution-continuity assertion;
- more than one normative assertion;
- no semantic role or an ambiguous role;
- a role assertion that violates the semantics in Section 7;
- missing or unusable identity, required provenance, or necessary execution context;
- an identity collision or reuse that would make distinct events indistinguishable;
- an operation association that would collapse distinct attempts;
- a relationship with invalid direction, endpoints, or semantics;
- an unsupported ordering or causal claim;
- an assertion of authority over external state that LPM does not possess; or
- content that would require mutation of an existing durable event to become valid.

A rejected candidate record MUST NOT enter durable history and MUST NOT be treated as recovery evidence. Rejection does not establish that the represented occurrence did not happen. It establishes only that the candidate was ineligible to become a conforming durable event. Rejection does not permanently consume an event identity. A producer MAY create and submit a corrected candidate representing the same occurrence, provided any resulting durable event satisfies the identity, eligibility, and immutability requirements of this RFC.

This RFC does not prescribe error codes, rejection interfaces, or validation procedures.

## 16. Contradictions and Uncertainty

Contradictory durable events remain historical evidence that their assertions were recorded. A producer or consumer MUST NOT resolve a contradiction by rewriting, deleting, or silently overriding an event.

A correction or qualification MAY add evidence that changes the supported derived interpretation. Mere recency MUST NOT determine truth. Where durable history does not justify one conclusion, consumers MUST preserve the conflict and material uncertainty.

Absence of evidence MUST remain distinguishable from affirmative evidence. Unknown ordering, unobserved external effects, and operations without terminal outcomes MUST remain explicit where material to safe continuation.

## 17. Derived Execution State

Derived execution state MUST be computed from durable events and their protocol-defined context and relationships. Every material derived conclusion MUST remain traceable to the durable events that support, contradict, correct, qualify, or leave it uncertain.

Derived state MUST:

- preserve unresolved operations and material uncertainty;
- distinguish absence of evidence from affirmative evidence;
- preserve known and unknown ordering;
- avoid inventing historical events or unsupported outcomes;
- avoid rewriting or silently overriding durable history; and
- avoid claiming authority over external systems.

Derived state MAY change when new durable events enter history. Different views MAY be derived where evidence is incomplete, provided every view remains consistent with protocol semantics, exposes material uncertainty, and remains traceable to evidence. Derived state MUST NOT become an independent historical source of truth.

## 18. Recovery Conformance

A conforming consumer MUST be capable of interpreting durable history sufficiently to identify, where supported:

- known intent;
- known action-or-attempt stages;
- meaningful observations and their provenance;
- intermediate results and terminal outcomes;
- validation status and evaluated subjects;
- established decisions;
- unresolved operations;
- relevant corrections, qualifications, contradictions, and uncertainty;
- known and unknown ordering relationships; and
- applicable durability frontiers.

A consumer MUST NOT fabricate or infer success, failure, cancellation, completion, external effect, validation, or interruption events without affirmative durable evidence as defined for terminal outcomes in Section 7.4. When an operation is known to have started and no applicable durable terminal outcome exists, its recovery state MUST be unresolved or outcome unknown unless later durable evidence establishes a terminal outcome.

Recovery MAY compare durable history with observable external state. Resulting observations are new evidence and MUST NOT silently rewrite historical events or retroactively establish claims beyond what the new evidence supports.

## 19. Normative Interruption Scenario

Consider an agent assigned a task to modify `auth.ts`:

1. a durable intent event establishes the assigned task context;
2. a durable intent event records the intended modification;
3. a durable action-or-attempt event establishes that editing began;
4. a durable observation event records meaningful modification evidence;
5. a durable action-or-attempt event establishes that validation started;
6. the computer shuts down; and
7. no candidate record of a validation outcome becomes a durable event.

A conforming consumer MUST be able to establish from this history:

- the task context;
- the recorded intent;
- that editing began;
- the recorded modification evidence and provenance;
- that validation began;
- that no durable terminal validation outcome exists; and
- that the validation operation is unresolved or outcome unknown.

The consumer MUST NOT conclude that validation succeeded, failed, was cancelled, or never started. It MUST NOT fabricate an interruption event.

After restart, an inspection or new validation MAY produce a candidate record that becomes a new durable event. That new evidence MUST NOT retroactively rewrite the original history or establish that the original validation completed unless it specifically supports that historical claim.

## 20. Implementation-Neutral Conformance Boundary

Protocol conformance is determined by externally observable semantic behavior, not by internal technology. An implementation MAY use any representation, storage, language, transport, interface, or integration technology provided that its protocol behavior satisfies this RFC.

Two conforming implementations MUST be able, through later-defined concrete specifications, to preserve and interpret the same event assertions, roles, operation associations, relationships, provenance, ordering uncertainty, durability context, and derived-state constraints without relying on private implementation meaning.

This RFC does not define a conformance test suite or concrete interoperability profile.

## 21. Requirements Summary

A conforming LPM protocol implementation:

- MUST preserve the occurrence, candidate-record, durable-event, durable-history, and derived-state lifecycle;
- MUST record meaningful evidence incrementally without requiring exhaustive activity capture;
- MUST admit only conforming candidate records into durable history;
- MUST preserve exactly-one-assertion and exactly-one-role event semantics;
- MUST preserve intent, action, observation, outcome, validation, decision, correction, and qualification distinctions;
- MUST derive operation lifecycles from immutable events and require affirmative evidence for terminal outcomes;
- MUST preserve the five mandatory relationship meanings without adding implied terminality, causation, mutation, or total ordering;
- MUST preserve event and operation identity across concurrency and history combination;
- MUST preserve minimum provenance and the external-authority boundary;
- MUST support partial, concurrent, independent, merged, and unknown ordering;
- MUST limit guaranteed recovery to durable history;
- MUST preserve append-only immutable history and represent corrections as new evidence;
- MUST preserve contradictions, unresolved operations, and material uncertainty; and
- MUST keep derived state traceable and subordinate to durable events.

## 22. Non-Goals and Deferred Decisions

This RFC does not define:

- a concrete event schema or exhaustive taxonomy;
- serialization, including JSON or JSONL;
- a database, storage format, persistence technology, or filesystem layout;
- an event or operation identifier format;
- timestamp, sequence-number, or logical-clock strategies;
- ordering, concurrency, synchronization, or history-combination algorithms;
- durability acknowledgement mechanisms or particular durability guarantees;
- recovery or state-derivation algorithms;
- APIs, command-line interfaces, transports, or programming languages;
- AI-agent or vendor-specific integrations;
- indexing, caching, replication, retention, compaction, migration, or archival;
- concrete conformance profiles, validation procedures, or test suites;
- authentication, authorization, or trust policy; or
- a general activity log, conversation store, project-management system, workflow engine, or knowledge graph.

These matters require later specifications. Any later choice MUST preserve this RFC's externally observable semantics.

## 23. Open Questions

The following protocol-level questions remain unresolved:

- Which operation boundaries and meaningful-recording thresholds belong to universal conformance versus domain-specific profiles?
- What minimum validation evidence must concrete profiles require for particular engineering activities?
- What interoperability rules should concrete profiles use to recognize duplicate representations without merging independent assertions?
- What additional relationship extensions, if any, demonstrate sufficient execution-continuity value for future standardization?
- How should conformance tests evaluate uncertainty preservation and combined-history behavior without prescribing implementation technology?

These questions MUST NOT reopen the accepted event semantics, history invariants, durability boundary, authority boundary, or ordering neutrality established by RFC-0001 through RFC-0003.

## 24. Relationship to Prior RFCs

RFC-0001 establishes why LPM exists and bounds it to engineering execution continuity. RFC-0002 establishes the principles that all protocol and implementation work must preserve. RFC-0003 defines event, role, operation, relationship, ordering, provenance, durability, and derived-state semantics.

This RFC defines the conformance contract that applies those accepted decisions. If it conflicts with RFC-0001, RFC-0002, or RFC-0003, the accepted RFCs govern and this RFC must be corrected.
