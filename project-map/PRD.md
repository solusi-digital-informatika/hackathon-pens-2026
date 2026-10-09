# Product Requirements Document (PRD)
## AI Production Workflow Assistant

| Field | Details |
|---|---|
| Product name | AI Production Workflow Assistant |
| Document type | Product Requirements Document (PRD) |
| Version | 1.0 |
| Status | Draft for hackathon MVP |
| Primary audience | Small AI VFX and creative production teams |
| Context | AI-assisted video/VFX production, including distributed teams such as Tokyo–Surabaya |
| Core principle | Adaptation over regeneration; human creative decisions remain authoritative |

---

## 1. Executive Summary

AI Production Workflow Assistant is a workflow coordination application for small creative teams producing AI-assisted video, animation, and VFX. It helps teams convert large and fragmented project briefs into structured production plans, maintain shared project knowledge, and respond to changing creative direction without unnecessarily discarding prior work.

The product addresses a recurring production challenge: a director changes a brief or visual direction late in the schedule, but the team cannot quickly determine which shots, prompts, references, and deliverables are affected. This creates misunderstandings, avoidable regeneration, rework, inconsistent output, and schedule risk.

The MVP will demonstrate a complete change-resilient workflow:

1. Import a project brief and public reference assets.
2. Extract requirements, constraints, creative direction, and unresolved questions.
3. Convert the brief into a structured shot list.
4. Record shot descriptions, prompts, assets, status, and version history.
5. Enter a new creative direction and compare it with the previous version.
6. Identify potentially affected shots and explain why they may be affected.
7. Recommend whether each shot should be retained, adapted, reviewed, or regenerated.
8. Allow a human to approve or override recommendations.
9. Produce an updated storyboard while preserving prior versions.

The MVP is not intended to replace creative judgment, generate a finished film autonomously, or serve as a complete professional editing/VFX suite.

## 2. Problem Statement

Creative production information is commonly spread across briefs, reference files, feedback, chat messages, prompts, and generated assets. These sources may change during production. When a direction changes, team members may not share the same understanding of the latest requirements or know which existing work remains usable.

This creates several connected problems:

- Large briefs take time to interpret and translate into executable tasks.
- Requirements and decisions are scattered across multiple sources.
- Direction changes do not automatically map to affected shots and assets.
- Previous versions and the reasons behind decisions are difficult to trace.
- Existing shots may become stale, while potentially reusable assets are overlooked.
- AI-generated outputs may not match the intended visual style or shot requirements.
- Dependencies between characters, visual style, shots, and deliverables are hidden.
- Production knowledge can be concentrated in a few experienced individuals.
- Rework and schedule impacts are difficult to estimate quickly.
- Individual shots may look acceptable but lack consistency as a sequence.

### Core problem statement

**How might we help small AI-assisted creative teams understand, trace, and respond to changing production direction while preserving useful work and maintaining a shared source of truth?**

## 3. Product Vision

Make creative production workflows more resilient to change by connecting project knowledge, creative decisions, shot-level work, assets, and deliverables in one traceable workspace.

### Product principles

1. **Shared project memory:** team members work from the same current project information.
2. **Traceability:** each shot and asset can be traced to the relevant requirements and direction version.
3. **Adaptation over regeneration:** assess whether work can be retained or adapted before recommending regeneration.
4. **Human-in-the-loop:** AI suggests and explains; authorized people make creative approvals.
5. **Evidence-aware AI:** recommendations should reference the requirements, changes, or dependencies that support them.
6. **Resilience over isolated features:** the complete workflow should remain understandable when direction changes or a key person is unavailable.
7. **Public-data-only demo:** the hackathon prototype must not require confidential client footage.

## 4. Goals and Non-Goals

### 4.1 Goals

- Reduce the time needed to understand a brief.
- Turn unstructured creative requirements into a structured shot plan.
- Make direction changes visible and traceable.
- Identify potentially affected shots and related assets.
- Preserve historical versions of briefs, prompts, and shot assets.
- Help users decide what to retain, adapt, review, or regenerate.
- Maintain a shared project view for distributed collaborators.
- Demonstrate measurable value with a controlled hackathon scenario.

### 4.2 Non-goals

- Replacing the director, artist, editor, or other creative decision-makers.
- Fully automating creative approval.
- Building a full video editor, compositing tool, or digital asset management platform.
- Guaranteeing that an AI-generated asset satisfies all artistic requirements.
- Building a production-grade multi-tenant enterprise platform during the hackathon.
- Using confidential client footage or restricted assets.
- Requiring a custom video-generation model as a condition for the MVP.

## 5. Target Users and Personas

### Persona A — Creative Director

**Needs**
- Communicate new creative direction clearly.
- Understand the consequences of a change before approving it.
- Review alternatives and maintain authority over creative decisions.

**Pain points**
- Repeating instructions across team members.
- Discovering too late that old shots no longer match the brief.
- Difficulty knowing which parts of the production plan are affected.

### Persona B — AI VFX / Creative Producer

**Needs**
- Translate the brief into tasks and a shot plan.
- Track shot status, versions, dependencies, and pending approvals.
- Identify schedule risks and prevent avoidable rework.

**Pain points**
- Manually reconciling documents and feedback.
- Tracking multiple generated alternatives.
- Coordinating changes across a small team.

### Persona C — AI Artist / Shot Creator

**Needs**
- Understand the current requirements for each shot.
- Access relevant references and previously successful prompts.
- Know what changed and what must be revised.

**Pain points**
- Working from outdated instructions.
- Repeating experiments because previous attempts are hard to find.
- Receiving feedback without clear links to the requirements.

### Persona D — Distributed Collaborator

**Needs**
- Access the current project state regardless of location.
- Understand the latest approved direction and shot versions.
- Continue work without relying on verbal handover alone.

**Pain points**
- Time-zone and communication gaps.
- Unclear source of truth.
- Missing context when a colleague is unavailable.

## 6. Primary User Journey

1. A producer creates a project.
2. The producer imports a brief and public reference assets.
3. The AI extracts requirements, constraints, visual direction, and open questions.
4. A human reviews and corrects the extracted information.
5. The system proposes a shot list and initial storyboard plan.
6. The team adds prompts and assets to individual shots.
7. A director submits a revised creative direction.
8. The system compares the new direction against the active version.
9. The system maps changed requirements to related shots and assets.
10. The system presents recommendations with explanations and supporting context.
11. An authorized user approves, rejects, or modifies each recommendation.
12. The team adapts or regenerates selected assets and saves new versions.
13. The storyboard reflects the latest approved state while older versions remain accessible.
14. The team reviews quality and deliverable readiness.

## 7. Functional Requirements

Priority definitions:

- **P0 — Must have:** essential to demonstrate the core value proposition.
- **P1 — Should have:** useful if implementation time allows.
- **P2 — Could have:** optional enhancement outside the core demo.

### FR-01 — Project Creation and Workspace

**Priority:** P0

The system shall allow a user to create a project with a name, short description, and project status.

**Acceptance criteria**
- A user can create and open a project.
- Each project has a stable identifier.
- Project data is separated from other projects.
- The workspace shows the active brief version and shot list.

### FR-02 — Brief and Reference Ingestion

**Priority:** P0

The system shall accept a brief and relevant reference materials for a project.

**MVP inputs**
- Plain text.
- PDF or DOCX if parsing support is available.
- Public reference images or image URLs.

**Acceptance criteria**
- The user can add or paste a brief.
- The system records the source name and import time.
- The original source remains accessible or its relevant content is preserved.
- Unsupported files produce a clear error rather than silently failing.
- The demo does not require confidential footage.

### FR-03 — AI Brief Analysis

**Priority:** P0

The system shall extract structured project information from the brief.

**Expected fields**
- Project objective and summary.
- Creative direction and visual style.
- Characters, locations, and key objects.
- Requirements and constraints.
- Deliverables.
- Acceptance criteria, when specified.
- Ambiguities and unresolved questions.
- Source references or excerpts supporting important extracted items, where feasible.

**Acceptance criteria**
- The result is displayed in editable fields.
- Users can correct the AI output.
- Uncertain or missing information is marked as unknown or requiring review.
- The system does not invent a requirement merely to fill a field.
- The reviewed information becomes the project's active structured brief.

### FR-04 — Shot List and Storyboard Planning

**Priority:** P0

The system shall create an editable shot list from the reviewed brief.

**Minimum shot fields**
- Shot ID.
- Title.
- Description.
- Relevant requirements.
- Reference assets.
- Prompt or generation instructions.
- Dependencies, if known.
- Current status.
- Current version.

**Acceptance criteria**
- The user can add, edit, reorder, and remove a shot.
- Every shot has a stable ID independent of its version.
- The shot list can be displayed as a storyboard or ordered list.
- AI-generated plans remain editable by a human.

### FR-05 — Shot Asset and Prompt Tracking

**Priority:** P0

The system shall associate prompts and generated/reference assets with the relevant shot.

**Acceptance criteria**
- A shot can have multiple assets or alternatives.
- An asset record can include filename or URI, asset type, source, prompt version, and status.
- The system distinguishes the current selected asset from alternatives.
- Existing assets are not deleted simply because a new direction is created.
- Actual file storage may be simplified for the hackathon, provided asset references remain reliable.

### FR-06 — Creative Direction Versioning

**Priority:** P0

The system shall preserve successive versions of the creative direction.

**Acceptance criteria**
- A new direction creates a new version rather than silently overwriting the old one.
- The system records version identifier, content, timestamp, and change note.
- The user can inspect the active and previous versions.
- Shots can reference the direction version used when they were created or last reviewed.

### FR-07 — Direction Comparison

**Priority:** P0

The system shall compare an updated direction with the currently active direction.

**Examples of changes**
- Realistic cinematic style to stylized animation.
- Cool lighting to warm lighting.
- Character design update.
- New location, framing, mood, or sequence requirement.
- A new restriction or acceptance criterion.

**Acceptance criteria**
- The system displays the main differences in understandable language.
- Changed, added, and removed requirements are distinguished where possible.
- The comparison does not automatically approve the new direction.
- The user can inspect the new direction before applying its consequences.

### FR-08 — Change Impact Analysis

**Priority:** P0

The system shall identify shots and assets that may be affected by a direction change.

**Recommendation categories**
- **Retain:** existing work appears to remain compatible.
- **Adapt:** existing work may be reusable after modification.
- **Regenerate:** regeneration may be necessary.
- **Needs review:** evidence is insufficient to recommend a confident action.

**Acceptance criteria**
- Each recommendation includes a reason.
- Recommendations reference changed requirements or known dependencies where feasible.
- The system can represent uncertainty.
- The user can override every recommendation.
- The system does not claim a shot is definitely valid based only on a superficial text match.
- Where dependency data is absent, the system marks the impact as uncertain rather than assuming no impact.

### FR-09 — Human Review and Approval

**Priority:** P0

The system shall require human review for consequential creative workflow decisions.

**Acceptance criteria**
- Users can accept, reject, or override a recommended action.
- The selected action and review status are recorded.
- The system distinguishes AI recommendation from human-approved status.
- Unreviewed changes remain visibly unreviewed.

### FR-10 — Version History and Traceability

**Priority:** P0

The system shall preserve the history of relevant shot and prompt changes.

**Acceptance criteria**
- Users can inspect earlier shot/prompt versions.
- The current version is clearly identified.
- The system records what changed and which direction version motivated the change, when known.
- A prior version can be selected as the working basis for a new revision without erasing history.
- The system does not falsely imply that all underlying files are recoverable if only metadata or links were stored.

### FR-11 — Updated Storyboard

**Priority:** P0

The system shall display the latest approved shot state in sequence.

**Acceptance criteria**
- The storyboard uses the latest selected version for each shot.
- Users can identify shots awaiting review or adaptation.
- The user can reorder shots.
- Previous versions remain accessible.
- The storyboard does not mark the entire project as complete while required approvals or checks remain outstanding.

### FR-12 — Project Knowledge and Decision Log

**Priority:** P1

The system should store reusable project knowledge and decisions.

**Examples**
- Why a visual direction was selected.
- Which prompts or references worked.
- Known limitations of a model or workflow.
- Why a shot was adapted or regenerated.
- Approval notes and relevant feedback.

**Acceptance criteria**
- Decisions can be associated with a project or shot.
- Users can search or browse decision records.
- Where feasible, AI responses can retrieve relevant project knowledge with source context.

### FR-13 — Quality Review Checklist

**Priority:** P1

The system should support a human-reviewed checklist for shot and storyboard readiness.

**Possible checks**
- Compliance with current direction.
- Character and visual continuity.
- Correct order and coverage of required shots.
- Missing assets or unresolved dependencies.
- Pending approval.
- Deliverable completeness.

**Acceptance criteria**
- Checks have explicit statuses such as pass, fail, or needs review.
- AI-generated assessments are identified as suggestions.
- Final approval remains with a human.

### FR-14 — Deadline and Rework Overview

**Priority:** P2

The system could summarize the number of affected shots and outstanding actions after a change.

**Acceptance criteria, if implemented**
- Show counts of retain, adapt, regenerate, and needs-review items.
- Show manually entered deadlines or estimates.
- Clearly label estimated effort as an estimate, not a measured fact.
- Do not claim cost or time savings without supporting measurements.

## 8. Non-Functional Requirements

### NFR-01 — Usability

- The main workflow should be understandable without AI or VFX engineering expertise.
- Key shot statuses and active versions should be visible.
- Recommendations should include plain-language explanations.

### NFR-02 — Reliability and Data Integrity

- A new version must not silently overwrite an existing version.
- Invalid or incomplete AI output must be validated before being saved as structured data.
- The system should provide understandable error messages for failed imports or AI calls.

### NFR-03 — Traceability

- Important extracted requirements should retain source context where feasible.
- Change recommendations should show their rationale.
- The user should be able to distinguish source facts, AI interpretations, and human decisions.

### NFR-04 — Performance

For the hackathon prototype, the interface should remain usable for a small project containing one brief and approximately 4–10 shots. AI processing time will depend on the selected model and provider. Avoid claiming a fixed response-time guarantee until measured.

### NFR-05 — Privacy and Security

- Use public sample assets or synthetic content in the demo.
- Do not upload confidential client footage.
- Keep API keys on the server side and out of client-visible code.
- Validate uploaded file types and sizes.
- Apply basic access separation if the prototype supports multiple users.
- Do not expose project data through predictable, unprotected endpoints.

### NFR-06 — Maintainability

- Separate project management, AI analysis, versioning, and asset metadata into distinct modules.
- Use structured schemas for LLM outputs.
- Keep the model provider replaceable where practical.
- Store identifiers and dependencies explicitly rather than relying only on free-text descriptions.

### NFR-07 — Accessibility and Collaboration

- Use readable labels and clear status indicators.
- Avoid using color alone to communicate status.
- Make the active direction and current shot versions explicit for distributed collaborators.

## 9. UX and Information Architecture

### 9.1 Main navigation

1. **Projects:** project list and creation.
2. **Project Overview:** summary, active direction, outstanding actions, and recent changes.
3. **Brief & Knowledge:** source documents, extracted requirements, constraints, and decisions.
4. **Shot Board:** ordered shots, thumbnails, prompts, dependencies, and status.
5. **Direction Changes:** compare versions and review impact analysis.
6. **Version History:** inspect prior versions of directions, prompts, and shot assets.
7. **Quality & Delivery:** checklist and outstanding approvals.

### 9.2 Key screen: Shot Board

Each shot card should display:
- Shot ID and title.
- Current thumbnail, if available.
- Current status.
- Relevant requirements.
- Active prompt/asset version.
- Direction version.
- Outstanding review indicator.

### 9.3 Key screen: Direction Change Review

The review screen should display:
- Previous direction.
- Proposed new direction.
- Summary of differences.
- A list of affected shots.
- Recommended action and explanation per shot.
- Evidence/dependency context, where available.
- Controls to accept, reject, or override recommendations.
- A clear confirmation step before the new direction becomes active.

### 9.4 Key screen: Version Comparison

The comparison screen should make it possible to inspect:
- Previous and current prompt.
- Previous and current asset, if available.
- Direction version associated with each.
- Change note and approval status.

## 10. Suggested Technical Architecture

The architecture below is a practical starting point, not a fixed requirement.

| Layer | Suggested technology | Responsibility |
|---|---|---|
| Frontend | Streamlit for fastest MVP; React if richer interaction is needed | Project dashboard, brief editor, shot board, direction review |
| Backend API | FastAPI | Project logic, import processing, change workflows, approvals |
| Database | PostgreSQL or Supabase | Projects, requirements, directions, shots, versions, decisions, dependencies |
| LLM service | Hosted API or suitable open-source model | Brief extraction, direction comparison, explanation and recommendations |
| Retrieval | Full-text search; embeddings if useful | Retrieve relevant passages and prior decisions |
| Asset storage | Local storage for demo or object storage | Images, thumbnails, and generated/reference assets |
| Optional vision service | Image similarity or vision-capable model | Support visual review where relevant |

### Architecture rules

- The LLM should not be the sole source of truth for project state.
- Persist important requirements, versions, and dependencies in structured data.
- Validate LLM output against a schema before updating project records.
- Use explicit relationships to trace changes from a requirement to affected shots.
- Treat semantic similarity as supporting evidence, not definitive proof that an asset is compliant.
- Keep creative approval in the application workflow.

## 11. Suggested Data Model

### Project
- `id`
- `name`
- `description`
- `status`
- `created_at`
- `updated_at`

### SourceDocument
- `id`
- `project_id`
- `name`
- `file_type`
- `source_uri` or stored content reference
- `version`
- `created_at`

### Requirement
- `id`
- `project_id`
- `source_document_id`
- `category`
- `description`
- `priority`
- `confidence` or review state
- `source_excerpt`, where feasible
- `status`

### DirectionVersion
- `id`
- `project_id`
- `version_number`
- `content`
- `change_note`
- `created_at`
- `created_by`, if user identity exists
- `approval_status`

### Shot
- `id`
- `project_id`
- `title`
- `description`
- `sequence_order`
- `status`
- `current_version_id`

### ShotVersion
- `id`
- `shot_id`
- `version_number`
- `direction_version_id`
- `prompt`
- `description`
- `change_note`
- `review_status`
- `created_at`

### Asset
- `id`
- `project_id`
- `shot_id`
- `shot_version_id`
- `asset_type`
- `file_uri`
- `source`
- `selection_status`
- `created_at`

### Dependency
- `id`
- `project_id`
- `source_type`
- `source_id`
- `target_type`
- `target_id`
- `relationship_type`
- `evidence_note`

### DecisionLog
- `id`
- `project_id`
- `shot_id`, nullable
- `decision`
- `reason`
- `actor_type` (human or AI)
- `direction_version_id`
- `created_at`

This schema can be simplified for the hackathon, but the prototype should preserve stable IDs and version history.

## 12. Core Workflow Logic

### 12.1 Brief analysis

1. Read or parse the input content.
2. Ask the LLM to return structured fields.
3. Validate the output schema.
4. Retain source context for important extracted items where feasible.
5. Present results for human correction.
6. Save the reviewed brief and requirements.

### 12.2 Direction change analysis

1. Save the proposed direction as a draft version.
2. Compare it with the active direction.
3. Identify added, removed, or modified requirements.
4. Query structured dependencies to find related shots and assets.
5. Use AI to assess semantic implications and generate explanations.
6. Combine dependency evidence and AI assessment into recommendations.
7. Mark low-confidence or weakly supported results as `needs_review`.
8. Ask an authorized user to approve the new direction and shot actions.
9. Save the approved direction and resulting decisions as new versions.
10. Update the storyboard based on approved shot versions.

### 12.3 Shot action rules

- **Retain:** evidence suggests the shot remains compatible; still allow review.
- **Adapt:** a portion of the shot can likely be preserved, but one or more requirements need updating.
- **Regenerate:** the current asset appears incompatible or cannot be adapted adequately.
- **Needs review:** evidence is missing, conflicting, or too uncertain.

The system should not infer that a shot is unaffected simply because no dependency record exists. Missing dependency data should lower confidence.

## 13. MVP Scope and Delivery Plan

### Must-have MVP

- Project creation.
- Brief input and AI extraction.
- Human editing of extracted requirements.
- Shot list/storyboard.
- Prompt and asset metadata per shot.
- Direction versioning.
- Direction comparison.
- Change impact recommendations with reasons.
- Human approval or override.
- Version history.
- Updated storyboard view.
- Public or synthetic demo assets.

### Optional enhancements

- Direct image/video generation integration.
- Visual similarity checks.
- Semantic search over project knowledge.
- Deadline and rework dashboard.
- Multi-user authentication and collaboration.
- Export to PDF/CSV or production tool integrations.

### Suggested implementation sequence

**Milestone 1 — Structured project data**
- Define data schema and seed a sample project with 4–6 shots.
- Implement project and shot CRUD operations.

**Milestone 2 — Brief understanding**
- Add brief ingestion and structured extraction.
- Build a review/edit interface for requirements.

**Milestone 3 — Versioned production plan**
- Add direction versions, shot versions, prompts, and asset metadata.
- Build the shot board and version history.

**Milestone 4 — Change analysis**
- Add direction comparison.
- Connect requirements to shots using explicit dependencies.
- Generate and display action recommendations.

**Milestone 5 — End-to-end demo**
- Add approval controls and update storyboard state.
- Test the complete change scenario.
- Record evaluation results and known limitations.

## 14. Success Metrics and Evaluation

The hackathon should evaluate workflow improvements rather than simply count AI calls.

| Metric | Definition | Evaluation method |
|---|---|---|
| Time to understand brief | Time required to identify key requirements and constraints | Compare manual and app-assisted review |
| Change impact identification | Share of truly affected shots correctly identified | Compare system results with a manually defined reference set |
| Recommendation quality | Share of suggested actions judged appropriate by reviewers | Review a fixed set of change scenarios |
| Asset reuse rate | Share of previous assets retained or adapted in the updated plan | Track asset status before and after changes |
| Traceability | Ability to find the requirement/direction that motivated a shot change | Run task-based user checks |
| Direction compliance | Degree to which revised shots satisfy the updated criteria | Human checklist or structured review |
| Workflow completion time | Time from direction change to approved updated plan | Compare manual and assisted workflows |

Do not state a percentage reduction in time, cost, or rework until it has been measured. Report the test setup, sample size, and limitations.

## 15. Risks and Mitigations

| Risk | Potential impact | Mitigation |
|---|---|---|
| LLM misinterprets a brief | Incorrect requirements or shot actions | Source-aware extraction, editable fields, schema validation, human review |
| Impact analysis misses a dependency | Affected shot remains outdated | Explicit dependency records, `needs_review` status, test cases |
| AI overstates visual quality | False confidence in output compliance | Treat visual assessments as advisory; human review |
| Version history overwrites data | Loss of previous work | Create new versions and preserve history |
| API latency or cost | Slow or unreliable demo | Limit context, cache safe results, use small models for simpler tasks |
| Scope expands too far | Core scenario remains unfinished | Prioritize the end-to-end change workflow |
| Confidential content is uploaded | Privacy or contractual risk | Use public/synthetic assets and communicate upload restrictions |
| Distributed team misunderstands state | Conflicting production actions | Clear active-version labels and approval status |

## 16. Demo Scenario and Acceptance Test

### Scenario

A six-shot project begins with a realistic cinematic style and cool blue lighting. After several assets have been created, the director changes the direction to stylized animation with warmer lighting while requiring the main character's identity to remain consistent.

### Test sequence

1. Create the sample project and import its brief.
2. Extract and review requirements.
3. Generate or enter six shots.
4. Attach public sample assets and prompt versions to selected shots.
5. Save the original direction as Version 1.
6. Enter the new direction as a proposed Version 2.
7. Display a comparison of the two directions.
8. Identify shots linked to the changed style, lighting, and character requirements.
9. Show recommendations with explanations and uncertainty where appropriate.
10. Have a user approve or override recommendations.
11. Create revised shot versions without deleting earlier versions.
12. Display the updated storyboard and outstanding review items.

### MVP acceptance criteria

The MVP is acceptable for demonstration when:
- A project can be created and populated with a brief and shots.
- The brief analysis is editable and does not require confidential data.
- A new direction can be compared with the previous direction.
- The system produces a shot-level impact list with explanations.
- The user can override recommendations.
- The application preserves earlier versions.
- The storyboard reflects the selected current versions.
- The demo clearly distinguishes AI suggestions from human approvals.
- Known limitations are documented rather than hidden.

## 17. Open Questions

These decisions can be finalized with the hackathon team:
- Will the first prototype accept PDF/DOCX uploads, or only pasted text?
- Which LLM provider or local model is available within the budget?
- Will assets be uploaded to local storage or stored as external URLs?
- Is direct image generation needed, or will prepared public assets be sufficient?
- Will the demo use a single-user workflow or basic multi-user access?
- What language should the interface support: Indonesian, English, or both?
- Which specific evaluation scenario and ground-truth affected-shot list will be used?

## 18. Final Product Positioning

**AI Production Workflow Assistant is a change-resilient production planning and knowledge system for AI-assisted creative teams.**

It helps a team turn a large brief into a structured plan, understand what changes when direction shifts, retain useful prior work, trace decisions, and update the storyboard with human oversight.

The MVP's central demonstration is not that AI can generate another image. It is that the workflow can survive a change in direction without losing context, silently discarding useful assets, or leaving the team unsure about what to do next.
