# OpenSource Tech Ontologies

RDF/OWL ontologies for learning software development concepts. Published via GitHub Pages at [soypete.github.io/soypete_tech_ontologies](https://soypete.github.io/soypete_tech_ontologies).

## Directories

| Directory | Description |
|-----------|-------------|
| `software/` | Course/procedure ontologies (PKO-based learning paths) |
| `thesaurus/` | SKOS thesauri for testing RDF inference |
| `education/` | Learning ontologies (L2WS, edu) with TBOX definitions |
| `social/` | Topic SKOS thesauri (Twitch topics) |
| `coordination/` | Coordination T-box for multi-agent wiki workflows |

## Ontologies

### software/

**go-intro.ttl** - Introduction to Go Programming
- PKO-based course with 5 lessons covering Go basics
- Includes step types: Read, Exercise, Code, Quiz
- Competency questions for package system and main function

**go-basics-llm.ttl** - Learning Go Basics (LLM-generated)
- 5-step course: Hello World → Variables → Functions → Conditionals → Loops

**production-go-llm.ttl** - Production Go Programming (LLM-generated)
- 5-step advanced course: Scheduler → pprof → Worker Pools → Testing → Benchmarking

**self-hosted-ai-llm.ttl** - Self-Hosted AI (LLM-generated)
- 8-step course: Ollama → llama.cpp → Quantization → vLLM → Load Testing

### thesaurus/

**full_skos.ttl** - Complete SKOS Thesaurus
- Education concept scheme with broader/narrower relations
- Tests transitive inference, exactMatch/closeMatch
- Includes collections and ordered collections

**transitive_broader.ttl** - Transitive Broader Test
- Tests SKOS transitive inference: course → onlineCourse → selfPaced

**symmetric_related.ttl** - Symmetric Related Test
- Tests skos:related symmetric inference

**inconsistency_broader_narrower.ttl** - Consistency Test
- Tests detection of inconsistent broader/narrower relations

**cycle_detection.ttl** - Cycle Detection Test
- Tests handling of cycles: A → B → C → A

### education/

**TBOX_LEARNING_SOFTWARE.ttl** - Learning-to-Write-Software (L2WS) Ontology
- Comprehensive TBOX with 853 lines defining:
  - Difficulty levels (Beginner, Intermediate, Advanced)
  - Learning domains (Fundamentals, Web Dev, Systems, AI, etc.)
  - Competency questions and skill-to-role mappings
  - Properties: requiresPrerequisite, buildsToward, isUsefulFor, isAppropriateFor

**edu.ttl** - Simple Education Ontology
- Basic Person/Student/Instructor/Course/Lesson classes
- Example instances with blank node union

### social/

**twitch_topics.ttl** - Twitch Stream Topics Thesaurus
- SKOS concepts for Twitch streaming topics
- Programming languages, AI/ML, Infrastructure, Database categories

### coordination/

**coord.ttl** - Coordination T-box
- Closed OWL vocabulary for coordinating multiple AI coding agents through a shared, searchable, typed wiki log
- 10 entity types (Source, Claim, Entity, Contradiction, Decision, Blocker, Handoff, Ack, Release, ContractChange)
- 8 link predicates (derivedFrom, contradicts, supports, about, relatesTo, answers, acknowledges, blocks)
- Models the request -> decision -> ack protocol with a RequestStatus state machine (Open, Answered, Acknowledged)

**README.md** - Coordination pattern documentation
- The two-channel rule (wiki is the system of record; prompts carry only instructions)
- The R -> D -> ack protocol, the claim rule, and how to adopt the pattern yourself

## Competency Questions

The L2WS ontology supports these competency questions:

- What skills are required for a specific role?
- What topics should I learn first for a given skill?
- What roles can I pursue after learning a skill?
- What projects can I build with a skill?
- What is the learning path from beginner to a target role?

## Usage

Query with SPARQL endpoint or RDF library:

```bash
# Validate TTL syntax
riot --validate software/go-intro.ttl

# Query with SPARQL
sparql --query="SELECT ?s ?p ?o WHERE { ?s ?p ?o }" --service=https://example.org/sparql
```

## License

MIT