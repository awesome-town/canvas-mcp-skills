---
name: plan-module
description: Plan an entire Canvas module — module outcomes page, a sequence of lesson pages, and at least one summative assessment — then create the Canvas Module itself (unpublished) with all items added in order. Use when the teacher says "plan a module", "create a module", "design a module", "draft a unit", or describes a multi-lesson stretch of curriculum (often spanning several weeks). Enforces the school's module naming convention. Generic — works with whatever templates and competency framework the school config provides.
---

# Plan Module

Plan a complete Canvas module: the Module Outcomes page, the lessons inside it, the summative assessment at the end, and the Canvas Module container that holds them all. All drafts. Teacher publishes manually.

> **This skill never publishes anything.** Every page lands in Canvas as a draft. The Canvas Module itself is created in Canvas's default unpublished state. The teacher reviews everything in Canvas and clicks Publish themselves when ready.

## When to use

A teacher describes a multi-lesson stretch of curriculum they want to scaffold in Canvas. Common phrasings:

- "Plan a module on vibe coding — Weeks 24–36, course DSGN 9"
- "Design Module 4 for FSV 117 — pitch development, 4 weeks"
- "Draft a watershed unit, 6 lessons + a test"
- "I need to set up the next module — AI tools for designers, 8 weeks"

## When NOT to use

- **Single lesson** — that's `plan-lesson`.
- **Single assessment** — that's `plan-assessment`.
- **Adding to an existing module** — call `list_modules`, then `add_module_item` directly (or use `plan-lesson` / `plan-assessment` and then add the resulting page).

## Prerequisites

- canvas-mcp **v0.3.12 or later** (older versions don't have `create_module`).
- A school config with `lesson`, `assessment`, and `module_outcomes` page templates (Franklin: bundled by default).

## Workflow

### 1. Capture the module metadata

These pieces are required to build the module. Get them from the teacher's initial request; ask only for what's missing:

- **Canvas course** (code or numeric id — confirm via `list_courses` if there's any ambiguity)
- **Module number** (integer; e.g., 4)
- **Module short title** (the marketing name; e.g., "Vibe Coding")
- **Module subtitle** (the elaboration; e.g., "Transforming Text into Functional Software with AI")
- **Week range** (start–end integers, e.g., 24–36)
- **Topic / scope** (what the module is actually about — free-form from the teacher)
- **Approximate number of lessons** (the teacher tells you, or you propose based on week count; see "Default lesson count" below)

Build the **module name** to the exact Franklin convention:

```
Module {number}: {short_title}: {subtitle} (Weeks {start}-{end})
```

Example: `Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)`

Build the **Module Outcomes page title**:

```
Module {number}: {short_title} - Outcomes
```

Example: `Module 4: Vibe Coding - Outcomes`

Do NOT improvise these names — the formatting is part of how the school's Canvas pages are organized.

### Default lesson count

If the teacher didn't specify the number of lessons, propose based on the week range:

- 1–2 weeks: 2–3 lessons
- 3–4 weeks: 4–6 lessons
- 5–8 weeks: 6–10 lessons
- 9+ weeks: ~1 lesson per week, but ask the teacher

Always confirm with the teacher before committing — different teachers run modules at different cadences.

### 2. Pick a design approach

Ask the teacher:

> *Want to use **backwards design (UbD)** to think through the module's goals first, or **jump straight to lesson sequencing**?*
>
> *Backwards design takes 5–15 extra minutes but produces a stronger module — we start with what students should understand by the end, then design how we'll know they got there, then plan the lessons. Either way the Canvas artifacts end up the same.*

**If the teacher picks backwards design** → follow **Step 3 (UbD path)** below.
**If the teacher picks topic-first / quick scaffold** → skip to **Step 4 (topic-first path)** below.

Default to **asking** rather than assuming. UbD is pedagogically stronger but slower; topic-first is fast but inverts the design. Some teachers want one mode every time; some pick per-module.

### 3. UbD path (when backwards design is chosen)

Walk three stages, in order, before any lesson titles get committed.

#### Stage 1 — Desired Results

Before generating anything, identify:

**Enduring Understandings** (1–3 statements). The big ideas students should retain long after they forget the details. Full sentences, transferable, worth uncovering — not topics or facts. Example: *"Energy transformations drive all living systems and follow predictable patterns that humans can harness."* Distinguish for the teacher between a topic (`photosynthesis`), a fact (`plants convert sunlight to energy`), and an enduring understanding (the example above).

**Content Objectives** (2–4). What students will **know**. Use verbs like identify, describe, explain, define, compare. Specific and measurable. Example: *"Students will explain the relationship between light reactions and the Calvin cycle."*

**Skill Objectives** (1–3). What students will **do**. Use verbs like analyze, evaluate, create, apply, design, argue. Transferable across subjects. Example: *"Students will design an experiment to test variables affecting photosynthesis rate."*

**Essential Questions** (1–3). Open-ended, arguable questions that drive inquiry across the module. Written in student-friendly language. Connect to the enduring understandings. Example: *"How do we know what's worth understanding?"*

**Competency alignment.** Call `list_competencies`. Pick the **2–4 competencies** this module most clearly targets. These should connect naturally to the enduring understandings and skill objectives — don't shoehorn.

**For conventional subjects** (Algebra, US History, Biology, Spanish I): draft these for the teacher to react to. Claude has reasonable domain knowledge for established subjects.

**For unconventional subjects** (anything school-specific, designed by this teacher, or hard to find in a textbook catalog): don't assume. Pause at each decision, share initial ideas as conversation starters, let the teacher shape direction. Their expertise > Claude's pattern-matching.

**Stage 1 checkpoint** — show the teacher:

```
## Stage 1: Desired Results

**Enduring Understandings:**
1. [...]
2. [...]

**Content Objectives** (SWBAT):
- [...]
- [...]

**Skill Objectives** (SWBAT):
- [...]

**Essential Questions:**
1. [...]
2. [...]

**Competency Focus:** [Knowledge-Based Reasoning, Systems Thinking, ...]
```

Wait for the teacher to confirm or revise before moving to Stage 2. This is the foundation — getting it right matters more than speed.

#### Stage 2 — Evidence of Understanding

For each Stage 1 objective, design how you'll know students achieved it. Two kinds:

**Summative Assessment(s).** Evaluate learning at the end of the module. For each, specify:
- What it is (project / essay / presentation / portfolio / performance / test)
- Which objectives it measures (call back to Stage 1)
- Brief criteria — what distinguishes strong from weak performance

Default to one summative for a multi-week module; some modules have two (midpoint + final).

**Formative Assessment(s).** Ongoing checks for understanding during the module. For each:
- What it is (exit ticket / reflection / discussion / draft / peer feedback / observation)
- When it happens (which lesson or phase)
- What it tells the teacher

Formative assessments are usually lightweight and woven into individual lessons — they become part of the lesson's `tasks` slot in Canvas, not separate pages.

**Alignment check.** Every Stage 1 content + skill objective should be assessed by at least one summative. If an objective has no assessment, either the assessment plan is incomplete or the objective shouldn't be there. Flag misalignment to the teacher.

**Stage 2 checkpoint** — show the teacher:

```
## Stage 2: Evidence of Understanding

**Summative:**
| Assessment | Measures | Criteria sketch |
|---|---|---|
| [Name] | CO1, SO1 | [Brief criteria] |

**Formative:**
| Assessment | Timing | What it reveals |
|---|---|---|
| [Name] | Lesson 2 | [What teacher learns] |

**Alignment:**
✓ CO1 → assessed by [Summative]
✓ SO1 → assessed by [Summative]
[Flag any objective without an assessment]
```

Wait for confirmation before Stage 3.

#### Stage 3 — Lesson sequence (UbD-informed)

Now plan the lessons that get students from where they are to the Stage 1 understandings. Each lesson should serve specific objectives and have at least one assessment moment.

Generate lesson titles + 1-sentence summaries + objective alignment:

```
Proposed lesson sequence — Module 4: Vibe Coding (Weeks 24-36)

 1. Introduction to AI Coding Assistants
    Focus: CO1 (define AI assistants)
    Activity: Live demo + reflection
    Formative: Exit ticket — "What can AI coding tools do well and badly?"

 2. Prompt Engineering Basics for Code
    Focus: CO2, SO1
    Activity: Iterative prompt rewriting exercise
    Formative: Peer review of prompt iterations

 ... (one row per lesson)

13. Module Reflection & Showcase
    Focus: EU1, SO3
    Activity: Student presentations of their AI-built artifact
    Summative moment: Final project presentation
```

Sequencing principles:
- Build from concrete to abstract, simple to complex
- Front-load concepts later lessons depend on
- Place formative checkpoints at natural transition points
- Give students time to iterate — not just consume
- Include buffer time; ambitious pacing that requires perfection will fail
- End with synthesis, not just assessment — students should make meaning

Iterate with the teacher until the sequence is locked. Then jump to **Step 8 (preview)**.

The Stage 2 summative becomes the module's assessment page (see step 7d).
The Stage 1 enduring understandings + essential questions become the Module Outcomes page content (see step 7b — UbD path generates richer outcomes that reference the EUs and EQs).

---

### 4. Identify module-level competencies *(topic-first path only)*

Call `list_competencies`. If a framework is loaded (Franklin's TD Competencies, or another school's), spend a moment identifying which **2–4 competencies** this module targets. Modules span more than one competency — they're broader than a single lesson.

Hold these in mind as you draft outcomes and lesson titles. Surface them in the preview (step 8) and ask the teacher whether to mention competencies explicitly in the Module Outcomes page.

If no framework is configured, skip this step.

### 5. Propose the lesson sequence *(topic-first path only)*

Generate a list of lesson titles + 1-sentence summaries that flow through the module's scope. Show this to the teacher BEFORE drafting any pages:

```
Proposed lesson sequence for "Module 4: Vibe Coding" (Weeks 24-36):

 1. Introduction to AI Coding Assistants — what they are, how they work
 2. Prompt Engineering Basics for Code — clarity, context, iteration
 3. Building Your First AI-Assisted Function — a simple project
 4. Reading & Debugging AI-Generated Code — critical evaluation
 ...
13. Module Reflection & Showcase — students present their work

Plus a summative assessment at the end (see step 4).

Sound right? Adjust titles, drop lessons, or add more before I draft anything.
```

Iterate with the teacher until they're happy. Don't move on until the sequence is locked.

### 6. Propose the summative assessment *(topic-first path only — UbD path captured this in Stage 2)*

Ask the teacher about the culminating assessment for the module:

- **Type** — project / test / presentation / portfolio / essay / etc.
- **Total points** — for the grade boundaries table
- **Weighting** — % of trimester/term
- **Time/duration** — for tests this is minutes; for projects this is the timeline

Propose if the teacher doesn't have specifics yet. The default for a multi-week module is usually a project or portfolio (rarely a sit-down test).

### 7. Draft the Module Outcomes content

**If you took the UbD path (Step 3):** the Module Outcomes content is richer. Generate the `outcomes` slot HTML with three sections:

```html
<li>
  <strong>Enduring Understandings:</strong>
  <ul>
    <li>Energy transformations drive all living systems...</li>
    <li>...</li>
  </ul>
</li>
<li>
  <strong>Essential Questions:</strong>
  <ul>
    <li>How do we know what's worth understanding?</li>
    <li>...</li>
  </ul>
</li>
<li>
  <strong>By the end of this module, students will be able to:</strong>
  <ul>
    <li>Design and prompt-engineer AI assistants to generate working code (Knowledge-Based Reasoning)</li>
    <li>...</li>
  </ul>
</li>
```

(The template wraps everything in `<ol id="kl_objective_list">`, so each section above lives inside its own `<li>`.)

**If you took the topic-first path (Step 4–5):** generate 4–8 outcome statements. Each should:

- Start with an action verb (Design, Build, Evaluate, Analyze, Apply, etc.)
- Be observable / measurable (not "students will understand X" — say what they'll DO)
- Map to one of the identified competencies where applicable
- Reflect the scope of the WHOLE module, not just one lesson

```html
<li>Design and prompt-engineer AI assistants to generate working code</li>
<li>Evaluate AI-generated code for correctness, security, and clarity</li>
<li>...</li>
```

Either way, show the outcomes content in the preview (step 8) before creating the page.

### 8. Preview the full module structure

Before any tool calls touch Canvas, show the teacher the complete plan:

```
Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)
in course DSGN 9 (id: 60366)

[Design approach]
  ✓ Backwards design (UbD)   [or: Topic-first / quick scaffold]

[Module Outcomes page]
  • 3 enduring understandings, 2 essential questions, 5 student outcomes  [UbD path]
    -or-
  • 5 outcomes  [topic-first path]
  • Competency focus: Knowledge-Based Reasoning, Systems Thinking, Adaptability

[13 lesson pages]
  1. Introduction to AI Coding Assistants  (CO1)
  2. Prompt Engineering Basics for Code    (CO2, SO1)
  ... (13 total — objective tags only present on UbD path)

[1 assessment page]
  • Title: Module 4 Final Project: Build with AI
  • Type: project, 50 points, 25% of trimester, 4-week timeline
  • [UbD path: measures CO1, CO2, SO1, SO2]

[Canvas Module]
  • Name: Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)
  • Position: <next available>
  • Items in order: Outcomes → Lesson 1 → ... → Lesson 13 → Final Project
  • Status: UNPUBLISHED (teacher publishes manually)

How detailed should the lesson and assessment pages be?
  • Outline (default) — title + 1-sentence "about" stub per page
  • Stub — title only, all slots empty
  • Full — every slot filled now (slow, but complete)
```

Get explicit approval. If the teacher wants changes, iterate. Don't proceed to step 9 without "yes, go ahead."

### 9. Create everything as drafts

In this order, calling MCP tools:

#### 9a. Create the Canvas Module

```
create_module(
  course_identifier: <course>,
  name: "Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)"
)
```

The module is created with `workflow_state: "unpublished"`. Capture the returned `module.id` — you'll need it for `add_module_item` calls.

#### 9b. Create the Module Outcomes page

```
create_page(
  course_identifier: <course>,
  title: "Module 4: Vibe Coding - Outcomes",
  template: "module_outcomes",
  slots: {
    outcomes: "<li>Design and prompt-engineer AI assistants...</li><li>...</li>..."
  }
)
```

Note: the outcomes slot is the inner `<li>` items — the template wraps them in the `<ol>`. Generate clean `<li>` content; don't include the surrounding `<ol>` tags (the template adds them).

Capture the returned `url` (the page slug, e.g., `module-4-vibe-coding-outcomes`).

#### 9c. Create lesson pages

For each lesson in the sequence, call `create_page` with `template: "lesson"`.

In **Outline mode** (default), generate a 1-sentence summary for the `about` slot, leave all other slots empty:

```
create_page(
  course_identifier: <course>,
  title: "Introduction to AI Coding Assistants",
  template: "lesson",
  slots: {
    about: "<p><em>This lesson introduces students to AI coding assistants (Claude, Copilot, ChatGPT) — what they are, how they work, and the basics of interacting with them.</em></p>",
    to: "",
    concepts: "",
    resources: "",
    tasks: ""
  }
)
```

**UbD path adjustment to Outline mode:** when the teacher took the UbD path in Step 3, also include the objective alignment + any formative assessment moment from Stage 3 in the `about` slot summary, e.g.:

```
about: "<p><em>This lesson introduces AI coding assistants...</em></p>
<p><strong>Objectives:</strong> CO1 (define AI coding assistants), SO1 (evaluate when to use them).</p>
<p><strong>Formative check:</strong> Exit ticket — \"What can AI coding tools do well and badly?\"</p>"
```

This gives the teacher the UbD scaffolding inside each page so when they later run `plan-lesson` per lesson, the objective context is right there in the draft.

In **Stub mode**, leave all slots empty (the page will be created with empty accordions; the teacher fills them later via `plan-lesson` or directly in Canvas).

In **Full mode**, walk the `plan-lesson` workflow for each lesson — generate per-slot content, optional sections, competency alignment, the works. This is slow for 13 lessons but produces complete drafts. If the teacher took the UbD path, pass the relevant Stage 1 objectives + Stage 3 formative assessment to each `plan-lesson` invocation as context.

Capture each returned `url`. Don't ask the teacher to confirm each lesson individually in Outline/Stub mode — just power through and produce a summary at the end.

#### 9d. Create the assessment page

`create_page` with `template: "assessment"`. In Outline mode, write a short description and a placeholder for the grade boundaries table; leave the other slots empty. In Full mode, walk the full `plan-assessment` workflow.

**UbD path adjustment:** if the teacher took the UbD path, the summative assessment design from Stage 2 should drive the `description` slot. Reference the Stage 1 objectives the assessment measures (e.g., "This assessment measures CO1, CO2, and SO1 from the module's stated objectives — students will...").

#### 9e. Add items to the Canvas Module

In order: Module Outcomes page, then each lesson page in sequence, then the assessment page. Use `add_module_item` for each:

```
add_module_item(
  course_identifier: <course>,
  module_id: <returned-from-7a>,
  type: "Page",
  title: "Module 4: Vibe Coding - Outcomes",
  content_id: "<page-slug-from-7b>"
)
```

Note `position` defaults to "append at end" — since you're calling them in order, that's what you want.

### 10. Confirm to the teacher

Surface the result:

> Drafted **Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)** in DSGN 9 — 1 Module Outcomes page, 13 lesson pages, 1 final project page, all added to the new module in order. Everything is saved as drafts (module + items both unpublished). Open the module in Canvas at <module URL> to review and publish.
>
> Next steps:
> - Develop each lesson page (use `/plan-lesson` per lesson, or edit directly in Canvas)
> - Develop the final project page (use `/plan-assessment`, or edit directly in Canvas)
> - When the whole module is ready, publish the module + each item from the Canvas UI

## Common mistakes to avoid

- **Don't skip the design-approach choice in step 2.** Ask. Some teachers will pick UbD; some will pick quick scaffolding. Both are valid. Don't default silently.
- **Don't blend the two paths.** Pick one, walk it cleanly, then preview. Half-UbD with a topic-first preview is confusing for the teacher.
- **Don't shoehorn UbD if the teacher said quick scaffold.** They asked for fast; respect that. Save the UbD walk for next time.
- **Don't write enduring understandings as topics or facts.** "Photosynthesis" is a topic, not an understanding. "Plants convert sunlight to energy" is a fact. An enduring understanding is a transferable principle, written as a full sentence, worth uncovering.
- **Don't write essential questions with definite answers.** Essential questions are arguable. "What is photosynthesis?" is not essential. "How do we know what's worth understanding?" is.
- **Don't generate Stage 1/2/3 outputs without showing the teacher the checkpoints.** Each stage has a confirmation step. Use it.
- **Never tell the teacher anything is "published."** Module + every page are drafts. Use the word "drafted" or "created as drafts."
- **Don't improvise the module name format.** The convention is exact: `Module N: Short: Subtitle (Weeks X-Y)`. Same for the Module Outcomes page title: `Module N: Short - Outcomes`.
- **Don't draft lessons in Outline mode and silently flip to Full mode** (or vice versa) — confirm the mode in the preview and stick to it.
- **Don't ask the teacher to confirm each lesson individually** during the bulk-creation step. They confirmed the sequence; just execute and summarize at the end.
- **Don't add an extra `<ol>` wrapper in the `outcomes` slot.** The template provides the `<ol id="kl_objective_list">`; the slot is just the `<li>` items.
- **Don't make up Canvas URLs.** Use the `url` / `html_url` fields returned by `create_page` / `create_module`, not constructed paths.
- **Don't run `plan-module` on a module that already exists.** It will create a duplicate. Check via `list_modules` first if there's any chance the module already exists.
