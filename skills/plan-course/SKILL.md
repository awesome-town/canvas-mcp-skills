---
name: plan-course
description: Plan an entire Canvas course's structure — a sequence of modules across the academic calendar, with each module's Outcomes page drafted, plus optional foundational pages (Start Here, More Resources). Use when the teacher says "plan a course", "design a course", "set up the course structure", "scaffold a course", "outline a year of curriculum", or is starting work on a fresh Canvas course shell that needs its skeleton built out. Creates Canvas Modules + Module Outcomes pages; defers lesson + assessment drafting to follow-up plan-module / plan-lesson / plan-assessment runs.
---

# Plan Course

Plan the top-level structure of a Canvas course: the module sequence across the year (or semester / trimester / term), each module's outcomes, and optionally the foundational pages that orient students to the course. Drafts only — no publishing.

> **This skill never publishes anything.** Modules are created in Canvas's default unpublished state. Pages land as drafts. The teacher reviews and publishes manually.

## When to use

A teacher is setting up a fresh Canvas course or restructuring an existing one at the course level. Common phrasings:

- "Plan the structure for DSGN 9 for next year"
- "Set up the modules for FSV 117 — 4 modules across the year"
- "I need to scaffold this course — Modules 1-5 covering [topic]"
- "Outline a course on AI for high schoolers"

## When NOT to use

- **Single module** — that's `plan-module`. plan-course creates a SEQUENCE of modules; if you only need one, use plan-module directly.
- **Adding a module to an existing course's existing structure** — also `plan-module`.
- **Lessons or assessments within an already-scaffolded course** — `plan-lesson` / `plan-assessment`.
- **Creating the actual Canvas course itself** — courses are usually provisioned by your school's Canvas account admin / SIS sync. This skill assumes the course shell already exists.

## Prerequisites

- canvas-mcp v0.3.12 or later (needs `create_module`).
- An existing Canvas course shell (this skill doesn't create courses, only their contents).
- A school config (Franklin: bundled by default). The skill reads `academicCalendar.weeksPerYear` if present, to inform module sizing.

## Workflow

### 1. Capture the course context

Get these pieces from the teacher's initial request; ask only for what's missing:

- **Canvas course** — code or numeric id. Confirm by calling `get_course_details(course_identifier)`. Show the teacher the course name + code you found so they can catch a typo.
- **Subject / discipline** — what the course is about (e.g., "Design 9", "AI for Designers", "Entrepreneurship"). Often inferable from the course name.
- **Grade level** — if not already obvious from the course code/name.
- **Term type** — year-long / semester / trimester / quarter. For Franklin, year-long is most common (35 weeks).
- **Number of modules to plan** — see "Default module count" below. The teacher tells you, or you propose.

If the school config has `academicCalendar.weeksPerYear`, that's the default duration. Otherwise ask.

### Default module count

Based on duration:
- 4–8 weeks (one term/quarter): 1–2 modules
- 12–16 weeks (semester): 3–4 modules
- 24–35 weeks (year): 4–6 modules

Modules don't have to be equal length — most courses have a longer arc with a few shorter focused modules. Confirm with the teacher.

### 2. Read the academic calendar from school config

If `list_competencies` (or a future config-introspection tool) surfaces `academicCalendar.weeksPerYear` data, use it to scope week numbering. For Franklin's 35-week year, modules typically span weeks 1–7, 8–14, 15–21, 22–28, 29–35 (or some similar split — depends on the course).

If no calendar info is available, ask the teacher: "What's the week numbering convention for this course? (e.g., Weeks 1-35, or numbered per-term)"

### 3. Identify course-level competencies + transfer goals

#### 3a. Competency mapping

Call `list_competencies`. For a year-long course, the typical expectation is that students will encounter and develop **all** of the configured competencies over the year — not just a subset like in a single module or lesson. Different modules emphasize different competencies; together they cover the full framework.

In the preview (step 7), surface this: *"Across the year this course should touch all [N] [framework name] competencies. Different modules will emphasize different ones — I'll suggest the primary 2–4 competencies for each module below."*

If no framework is configured, skip the competency mapping.

#### 3b. Transfer goals (UbD course-level)

Beyond competencies, ask the teacher about **transfer goals** — what students should be able to do by the end of the *entire course*, in contexts beyond the classroom. Transfer goals are broader than module-level objectives and longer-lasting than any single unit.

For each course, identify **2–5 transfer goals**. Examples:

- "Apply user research methods to understand a real audience's needs and design accordingly." (a design course)
- "Use scientific reasoning to evaluate claims about energy systems in everyday life." (a physics course)
- "Build a sustained creative practice and reflect on one's own process." (an arts course)

Each transfer goal should:

- Start with an action verb
- Describe something students can do in life, not just on a test
- Be ambitious but achievable across the year
- Inform multiple modules' work (not just one)

Show in the preview. Most teachers haven't articulated transfer goals explicitly even if they have them implicitly — this step helps surface them. Don't push if the teacher resists; some courses are happy without an explicit course-level transfer frame.

### 4. Propose the module sequence + throughlines

Generate a list of modules. For each module, include:

- **Number** (sequential — Module 1, Module 2, …)
- **Short title** (the marketing name; e.g., "User Research")
- **Subtitle** (elaboration; e.g., "Discovering and Understanding Your Audience")
- **Week range** (e.g., Weeks 1–7)
- **Primary competencies** (2–4 from the framework)
- **1-sentence topic / scope summary**
- **Transfer goal alignment** (which of the course transfer goals from step 3b this module advances — usually 1–2 per module)

#### Identify throughlines (UbD course-level)

After proposing the module sequence, identify the **throughlines** — concepts, skills, or essential questions that recur across multiple modules and build progressively. These are what makes a course feel cohesive rather than like 5 disconnected units.

Examples:

- **A recurring essential question** ("How do designers turn empathy into product?" — visits in Module 1's research lens, Module 3's prototyping, Module 5's showcase)
- **A skill that builds progressively** (sketching → wireframing → high-fidelity prototyping → user-testing across modules)
- **A concept that deepens over time** (the design process gets richer with each module's tools)

Surface 2–4 throughlines and show which modules they thread through. Like transfer goals, don't push if the teacher doesn't see them — but most courses have them implicit and surfacing them makes the course design stronger.

Use the Franklin module naming convention for each:

```
Module {N}: {Short Title}: {Subtitle} (Weeks {start}-{end})
```

Example proposal (with transfer goals + throughlines):

```
Course structure proposal — DSGN 9 (Design 9)
Year-long course, Weeks 1-35

## Transfer Goals
By the end of this course, students will be able to:
1. Apply user research methods to understand a real audience's needs and design accordingly
2. Build a sustained creative practice and reflect on their own process
3. Communicate design decisions clearly to varied audiences

## Module Sequence

Module 1: User Research: Discovering and Understanding Your Audience (Weeks 1-7)
  • Primary competencies: Empathy/Perspective Taking, Collaboration, Reflexivity
  • Transfer goals: 1, 2
  • Scope: Interview techniques, observation methods, synthesis, persona-building

Module 2: Ideation & Sketching: Generating Solutions (Weeks 8-14)
  • Primary competencies: Adaptability, Futures Thinking, Storytelling/Communication
  • Transfer goals: 2, 3
  • Scope: Divergent thinking, structured ideation, rapid sketching, concept selection

Module 3: Prototyping: Making Ideas Tangible (Weeks 15-21)
  • Primary competencies: Knowledge-Based Reasoning, Adaptability, Agency
  • Transfer goals: 1, 2
  • Scope: Paper prototypes, digital wireframes, physical mockups, testing protocols

Module 4: User Testing & Iteration: Refining the Solution (Weeks 22-28)
  • Primary competencies: Empathy/Perspective Taking, Reflexivity, Systems Thinking
  • Transfer goals: 1, 2
  • Scope: Test design, observation, feedback synthesis, iteration cycles

Module 5: Communication & Showcase: Presenting Your Work (Weeks 29-35)
  • Primary competencies: Storytelling/Communication, Agency, Reflexivity
  • Transfer goals: 3, 2
  • Scope: Process documentation, presentation design, final showcase

## Throughlines (concepts/skills that recur)
1. The design process — visits in every module, gets richer each time
2. Reflective practice — students journal/reflect at the end of every module
3. Essential question: "How do designers turn empathy into product?" — recurs across Modules 1, 3, 4, 5

Across the year all 9 Transdisciplinary Competencies are encountered (mapped above).

Sound right? Adjust titles, swap modules, change week ranges, or change competency / transfer-goal emphasis before I draft outcomes.
```

Iterate with the teacher until the sequence is locked. Don't move on until they say "looks good" or equivalent.

### 5. Draft outcomes for each module

For each module in the sequence, draft 4–8 outcome statements following the same standard as `plan-module`:

- Start with an action verb (Design, Build, Evaluate, Analyze, Apply, …)
- Be observable / measurable
- Map to the module's primary competencies
- Reflect the scope of the WHOLE module, not one lesson
- **When transfer goals are defined (step 3b)**, ensure each module's outcomes contribute to advancing at least one transfer goal — surface that alignment in the preview

This step takes a moment — 5 modules × 6 outcomes = ~30 outcome statements. Power through; the teacher will review them all in the preview.

When the teacher later runs `plan-module` on each module, they can choose the UbD path within that skill to develop the module further (enduring understandings, essential questions, formative + summative assessment plan). The transfer goals and throughlines you identified here give them the bigger-picture frame to work within.

### 6. Identify foundational pages (optional)

Ask the teacher whether they want the skill to also draft these two foundational pages:

- **Start Here** — welcome message, course overview, how-to-navigate, contact info, expectations
- **More Resources** — external links, reading lists, office hours, references

These are referenced by the nav strip in the school's lesson/assessment templates, so they should exist. If they already exist in the course, skip them. To check: call `list_pages` and look for `start-here` and `more-resources` slugs.

If the teacher wants them created:
- Ask for any specific content they want included (e.g., for Start Here: "what should students do first?", "what are your office hours?")
- Default Start Here content includes: a welcome paragraph, a "how to navigate this course" section pointing at modules + syllabus, and an "expectations" section
- Default More Resources content is mostly empty — the teacher fills it in over time

Use `template: "default"` for these (or `template: "none"` if no default template is configured) — they don't fit the `lesson` or `assessment` shape.

### 7. Preview the full course plan

Before any tool calls touch Canvas, show the teacher the complete plan:

```
Course Structure Plan — DSGN 9 (course id 60366)
Year-long, Weeks 1-35
All 9 TD Competencies covered across modules

[5 Canvas Modules]
  1. Module 1: User Research: Discovering and Understanding Your Audience (Weeks 1-7)
     Outcomes: 6 items, competency-aligned (Empathy, Collaboration, Reflexivity)
  2. Module 2: Ideation & Sketching: Generating Solutions (Weeks 8-14)
     Outcomes: 5 items (Adaptability, Futures Thinking, Storytelling)
  ... (5 total)

[5 Module Outcomes pages]
  • One per module, with the outcomes listed above
  • All use template: "module_outcomes"

[2 Foundational pages] (if requested)
  • Start Here — welcome, navigation, expectations
  • More Resources — empty placeholder for the teacher to fill in

Status of everything: UNPUBLISHED (teacher publishes manually)

NOT included in this plan:
  • Lesson pages within each module — run /plan-module per module to draft those
  • Assessment pages — same, drafted within each module's plan-module run
  • Canvas syllabus_body — edit directly in Canvas (or use update_course tool if available)

Ready to create?
```

Get explicit approval. If the teacher wants changes, iterate.

### 8. Create everything as drafts

In this order, calling MCP tools:

#### 8a. Foundational pages (if requested)

Create them BEFORE the modules so the nav-strip links in the module pages have somewhere to point.

```
create_page(
  course_identifier: <course>,
  title: "Start Here",
  template: "default",
  body: "<h2>Welcome to DSGN 9</h2><p>...</p>..."
)
```

Same for More Resources. Capture the URL slugs.

#### 8b. For each module, in order: create the Canvas Module, create the Outcomes page, add Outcomes to module

For Module 1:

```
// Create the module container
const m = await create_module(
  course_identifier: <course>,
  name: "Module 1: User Research: Discovering and Understanding Your Audience (Weeks 1-7)"
);
// m.module.id is the new module id

// Draft the Outcomes page
const outcomesPage = await create_page(
  course_identifier: <course>,
  title: "Module 1: User Research - Outcomes",
  template: "module_outcomes",
  slots: {
    outcomes: "<li>Design and conduct user interviews using semi-structured protocols</li><li>...</li>..."
  }
);

// Add the Outcomes page as the first item in the module
await add_module_item(
  course_identifier: <course>,
  module_id: m.module.id,
  type: "Page",
  title: "Module 1: User Research - Outcomes",
  content_id: outcomesPage.url   // for Page items, content_id is the slug
);
```

Repeat for each module. **5 modules = 15 tool calls in this step** (3 per module). Just power through; you'll summarize at the end.

### 9. Confirm to the teacher

Surface the result:

> Drafted the structure for **DSGN 9** — 5 Canvas Modules with their Outcomes pages, all saved as drafts. Plus 2 foundational pages (Start Here, More Resources) if you asked for those.
>
> What's NOT done yet (suggested next steps):
> - Develop lessons + assessments inside each module — run `/plan-module` per module
> - Edit the Canvas syllabus content (the page accessed via the nav-strip Syllabus link) — do this in Canvas directly, or ask me to add it
> - Publish the modules + pages when you're ready (manually in Canvas)
>
> Module URLs:
> - Module 1: <link>
> - Module 2: <link>
> - ...

## Common mistakes to avoid

- **Never tell the teacher anything is "published."** Modules + every page are drafts. Use "drafted" or "created as drafts."
- **Don't improvise the module naming convention.** Each module: `Module N: Short: Subtitle (Weeks X-Y)`. Each Outcomes page: `Module N: Short - Outcomes`. Numbers must be sequential.
- **Don't make up the Canvas course.** If `get_course_details` says the course doesn't exist or the teacher doesn't have access, stop and surface that — don't try to "plan around" a missing course.
- **Don't try to map every module to ALL competencies.** Each module emphasizes 2–4. Together the modules cover the framework.
- **Don't draft lessons or assessments in this skill.** That's `plan-module`'s job, per module. Suggest it in the wrap-up.
- **Don't create foundational pages without asking.** They might already exist (`list_pages` to check first), or the teacher might have their own version.
- **Don't add an `<ol>` wrapper in the `outcomes` slot.** The `module_outcomes` template provides the `<ol id="kl_objective_list">`; the slot is just `<li>` items.
- **Don't make up Canvas URLs.** Use the `url` / `html_url` fields returned by the tools.
- **Don't create a course that already has these modules.** Run `list_modules` first if there's any chance the structure already exists — duplicate Module 1s will be confusing.
