# Coordination T-box: a typed, searchable log for multi-agent work

This directory documents a coordination pattern for teams of AI coding agents
that edit the same repositories. The formal vocabulary lives in
[`coord.ttl`](./coord.ttl) (an OWL T-box in namespace `http://soypete.tech/coord/`),
and this README explains how to use the pattern so you can adopt it outside
this project.

## The problem

When several coding agents work in the same repo at the same time, each agent
is effectively blind to the others. Agent A edits `lib/parser.rs` believing it
owns that file; agent B edits the same file for the same reason. Neither can
see the other's intent, neither knows what has already been claimed, and the
result is merge conflicts, lost work, and decisions made twice in different
ways.

You cannot fix this by asking agents to talk to each other in their prompts.
A prompt is a private conversation between one orchestrator and one agent; it
dies when the session ends. What survives is a shared log that every agent can
search before it acts and append to when it does.

## The two-channel rule

There are exactly two channels in this pattern, and they carry different
things:

- **The wiki is the system of record.** Decisions, decision requests,
  blockers, claims, handoffs, acknowledgments, releases, and status live
  there, as typed, searchable, linkable entries.
- **Prompts carry instructions only.** A prompt says start, stop, resume, or
  read-this-wiki-entry. It never carries the *content* of a decision. When a
  decision is needed, the prompt points at the wiki entry that holds it.

The rule is enforced by the structure of the vocabulary: the wiki entry type
`decision` exists so a decision can be *stored* in the wiki, and the link
predicates `answers`, `acknowledges`, `derived_from`, and friends exist so
entries can point at each other. There is deliberately no entry type for
"decision stuffed inside a prompt."

The rationale is simple: any current or future agent — including one that was
not in the conversation where a decision was made — can find *why* something
was decided by searching the wiki. Prompts are ephemeral; the wiki is not.

## The entry types

The vocabulary is a closed T-box with ten entity types and eight link
predicates. A capture that uses an entity type or predicate not listed here is
rejected at the write boundary; nothing is silently coerced into a default.
These are the ten entry types:

- **source** — an immutable raw source document ingested into the wiki. A raw
  file plus a wiki page describing it. *Example: the git log of the repo being
  coordinated, ingested once at the start.*
- **claim** — a discrete assertion extracted from a source, a conversation, or
  an agent's pane. The unit of shared knowledge. *Example: "Agent A owns
  `lib/parser.rs` as of now."*
- **entity** — a named thing worth its own page: a person, project, tool, or
  concept. *Example: the repository itself, or the wiki plugin.*
- **contradiction** — a recorded conflict between two claims or pages. Making
  the conflict explicit is the point. *Example: claim X says the parser is
  owned by A, claim Y says it is owned by B.*
- **decision** — a recorded decision with its context and rationale. *Example:
  "The vocabulary namespace is `http://soypete.tech/coord/`."*
- **blocker** — a blocking issue that stalls the dependent task until
  resolved. *Example: "Cannot run the test suite; the parser does not
  compile."*
- **handoff** — a task or claim passed from one agent or context to another.
  *Example: "Ownership of the integration test passes from agent B to agent
  C."*
- **ack** — an acknowledgment that a handoff, request, or decision was
  received. Usually the first thing an agent writes after being resumed.
  *Example: "Ack: I will implement the Open status value."*
- **release** — a declared release of a resource, lock, or responsibility.
  The inverse of a claim. *Example: "Agent A releases `lib/parser.rs`."*
- **contract_change** — a change to an inter-agent contract, such as the
  coordination protocol itself. *Example: "The vocabulary gains a new link
  predicate."*

The eight link predicates, which are how entries point at one another:

- **derived_from** — this capture derives from the target source or page.
- **contradicts** — this claim or page contradicts the target.
- **supports** — this capture provides evidence supporting the target claim.
- **about** — this capture is about the target entity.
- **relates_to** — a general association with the target page, when nothing
  more specific applies.
- **answers** — this capture answers the target request or question.
- **acknowledges** — this capture acknowledges the target handoff, request, or
  decision.
- **blocks** — this capture blocks the target task until resolved.

## The R → D → ack protocol

The core workflow is the request → decision → acknowledgment cycle. It exists
because an agent must never resolve a coordination question on its own and
must never silently stop.

1. **A worker detects a stop condition.** While working, an agent hits
   something it is not authorized to decide (a conflict over scope, an
   ambiguous instruction, a decision the rules say belongs to a human). It
   writes a decision request to the wiki — a capture typed as a claim or
   decision, titled `requests/R-NNN-<slug>`, with the numbered decision
   request — then prints `BLOCKED: R-NNN` and stops.
2. **The orchestrator halts it explicitly.** The orchestrator sees the
   `BLOCKED` signal, confirms the worker is stopped (not churning), and
   records that halt so no other agent touches the same ground.
3. **The orchestrator adds analysis.** It reads the request, adds its own
   context and evidence to the wiki entry — alternatives, trade-offs,
   affected files or agents — and links the analysis to the request.
4. **The decision is surfaced to a human.** The orchestrator brings the
   request and the analysis to the person who owns the decision. No agent
   decides this; the human does.
5. **The human decides.** The human picks an option, and the orchestrator
   records it in the wiki as a `decision`, linked to the request with the
   `answers` predicate.
6. **The orchestrator resumes the worker by prompt.** The prompt does not
   repeat the decision's content. It says: read this wiki entry, then proceed.
7. **The worker writes an ack.** The resumed agent writes an `ack` entry that
   restates in one line what it will now do, and links it to the decision with
   `acknowledges`. This is what proves the decision was actually received, and
   it gives the orchestrator a place to check status.
8. **Cascade.** Any other worker whose work is affected by the decision gets
   the same treatment: halted explicitly, pointed at the wiki entry, resumed,
   and acked. The decision travels through the wiki, not through prompts, so
   every affected agent is updated by reference.

The state machine in this cycle is: a request is **open**, then becomes
**answered** when the orchestrator records the decision, then becomes
**acknowledged** when the worker writes its ack. The model in `coord.ttl`
represents this with a `RequestStatus` class and `Open`, `Answered`,
`Acknowledged` individuals, described further in that file.

## The claim rule

Before editing anything that other agents might also touch, search the wiki
for overlapping claims. The search is a lookup, not a guess: run it with the
exact words that would appear in a claim you might be about to duplicate. If a
claim already covers what you wanted to do, do not edit — write a `blocker`
entry instead, linked to the overlapping claim with `blocks`, and report it.
Claiming first is how ownership becomes visible to everyone; skipping the
search is how two agents edit the same file.

## Adopting it yourself

The vocabulary is a **closed T-box**. It is small by design — ten entity
types, eight predicates, one status model — and it is enforced at the write
boundary. Extending it (a new entity type, a new predicate, a new status
value) is a deliberate human edit, not something an agent invents while
working. If an agent needs a term that does not exist, that is a stop
condition: it writes a decision request rather than improvising. This is what
keeps the log uniformly searchable — every agent knows the full term list,
because there is a fixed list.

To adopt the pattern in your own project:

1. Copy the closed vocabulary (in this repository's original form it is
   defined in a TOML file that the wiki plugin loads; the RDF description in
   `coord.ttl` is the formal statement of the same terms).
2. Stand up a shared, searchable store that enforces the closed vocabulary at
   write time — entries whose type or link predicate is not in the list are
   rejected.
3. Teach your agents three habits: search before editing, write claims and
   decisions to the wiki instead of into prompts, and print `BLOCKED` and
   stop instead of improvising when the vocabulary or your rules do not cover
   the situation.
4. Make the human review step explicit in your orchestration. The protocol
   only works if decisions have an owner who is not an agent.

## Why RDF

There are two descriptions of the same vocabulary, and they serve different
consumers. The TOML file is what the wiki plugin actually loads at runtime; it
is compact, validated at startup, and cheap for the plugin to check every
capture against. The `.ttl` is the formal, linkable description: an OWL T-box
with classes, properties, domains, ranges, and a documented state model. Any
other system — a SPARQL endpoint, an inference engine, a future tool that
wants to reason over the log — can consume the RDF without knowing anything
about the plugin. Keeping both means the plugin code never changes when the
vocabulary is formalized or extended; the TOML and the TTL are two views of
one decision, and `coord.ttl` documents which TOML key each term maps to.

This directory is wired for documentation only: the `.ttl` describes the
vocabulary, and the plugin continues to load the TOML. Wiring the plugin to
load the RDF directly is a separate decision, not part of this one.
