---
name: plan-lesson
description: Plan a Canvas lesson page using the school's lesson template. Use when the teacher says "plan a lesson", "create a lesson page", "draft a lesson", or describes a single class session that needs a Canvas page. Generates content for every slot the school's lesson template defines (objectives, key concepts, resources, tasks, etc.), reviews with the teacher, then publishes as a draft Canvas page. Generic — works with whatever lesson template the school config provides.
---

# Plan Lesson

Plan a single class lesson and create it as a **draft** Canvas page using the school's lesson template. Generic across schools: the slot list, section defaults, and competency framework all come from the canvas-mcp's school config — this skill doesn't hardcode any of that.

> **This skill never publishes pages.** Every page lands in Canvas as a draft (`published: false`). The teacher reviews the draft in Canvas and clicks **Publish** themselves when ready. This is a hard policy — don't try to flip the published flag, and don't describe what the skill does as "publishing."

## When to use

A teacher describes a single class session they want to teach. Common phrasings:

- "Plan a lesson on the water cycle for DSGN 9"
- "Create a Canvas page for tomorrow's class about prototyping"
- "Draft a lesson — Week 3 of FSV 117, intro to user testing"
- "I need a lesson page on parallel circuits"

## When NOT to use

- **Multi-class units / modules.** That's `plan-module` (when we build it).
- **Assessments / quizzes / projects.** That's `plan-assessment`.
- **Just editing existing content.** Use `edit_page_content` directly via the canvas-mcp tools.

## Prerequisites

- canvas-mcp v0.3.11+ installed and configured.
- A school config with a `lesson` page template (Franklin: bundled by default; other schools: set `SCHOOL_CONFIG` to point at your own).

## Workflow

### 1. Confirm the basics

Before generating any content, confirm with the teacher:

- **Which Canvas course?** If they named a course by code or fragment (e.g., "DSGN 9"), call `list_courses` and confirm the match. If they passed a numeric `course_id`, you can skip the lookup but mention which course you're targeting so they can catch a typo.
- **What's the lesson title?** If they gave a topic ("the water cycle"), propose a working title ("The Water Cycle") and let them adjust.
- **Anything else they specifically want included?** A guest speaker, a particular reading, a discussion prompt — note these so they land in the right slot.

Don't ask about things you can figure out yourself or that don't matter at planning time. Specifically: don't ask about module placement yet (we'll handle it after the draft is created), don't ask about publishing — the MCP creates the page as a draft and the teacher publishes manually — don't ask about HTML structure (the template handles it).

### 2. Discover the lesson template

Call `list_page_templates` and find the entry named `lesson`. Read its:

- **`slots`** — these are the named content holes you'll need to fill. Every slot has a `description` telling you what content goes there.
- **`sections`** — optional sections with default-include/omit states. You can override via `include_sections` / `omit_sections` on the publish call.

If no `lesson` template exists in the school config, fall back: tell the teacher "no lesson template configured — I'll create a generic page instead" and use `template: "default"` or `template: "none"`. Continue the workflow as best you can.

### 3. Consider competency + module alignment

Call `list_competencies`. If a framework is configured (e.g., Franklin's TD Competencies, or another school's framework), **always** spend a moment thinking about which 1–3 competencies this lesson naturally targets. Don't skip this just because the teacher didn't bring it up — most lessons have a clear competency frame, and surfacing it makes the page genuinely more useful.

This is a *thinking* step, not a content step (yet). You're using the competency frame to inform what goes into each slot — especially `to` (the skills slot) and `tasks` (the activity slot). A watershed lesson naturally targets **Knowledge-Based Reasoning** (applying geography concepts) and **Systems Thinking** (understanding the interconnected water cycle). Knowing this, you'd phrase the `to` slot's skills with that lens, and design `tasks` that actually exercise those competencies rather than something tangential.

Pick 1–3 competencies, not all 9. The alignment is meaningful only if the lesson genuinely targets them. Don't shoehorn.

**If the teacher is working from a module-level unit plan** (e.g., they ran `plan-module` with the backwards-design path, or shared a UbD-style unit plan), look for the lesson's connection to that bigger picture:

- **Which essential question does this lesson advance?** A lesson should help students wrestle with at least one of the module's open-ended inquiry questions.
- **Which enduring understanding does it build toward?** Lessons accumulate toward the module's big-idea takeaways — name which one(s).
- **Which content + skill objectives is it teaching toward?** Often 1–3 per lesson, not all of them.

Ask the teacher: *"Is this lesson part of a module you've already designed? If so, which essential question / enduring understanding does it advance?"* If they have that context, fold it into the `to` slot framing (e.g., `<p>Through this lesson, students explore the question: "How do natural systems organize across landscapes?"</p>` followed by the can-do statements). If they don't have a unit plan, that's fine — skip this and use just the competency framing.

In step 6 (preview), surface the suggested competency alignment AND any unit-plan connections you noted. Ask whether to call them out explicitly in the lesson or keep them woven into the content.

If `list_competencies` returns `configured: false` (no framework set), still ask about unit-plan context — competencies and unit plans are separate concerns.

### 4. Generate content for each slot

For each slot in the template, generate appropriate HTML content. Match the slot's `description` from `list_page_templates`. Keep each slot focused — a slot is one section of the page, not a whole lesson.

Quality bar:

- **Concrete, classroom-ready.** "Students will diagram a local watershed" beats "Students will learn about watersheds."
- **Use real student work patterns.** Reference accessible materials, give clear directions. If you don't know what tools/platforms the class uses, ask.
- **HTML, not markdown.** The template body is HTML; produce `<p>`, `<ul>`, `<li>`, etc. directly.
- **Don't pad.** Empty slots are fine if they'd genuinely be empty in a real lesson. Don't fill `discussion` with a generic prompt unless the teacher asked for a discussion.
- **No fake links.** If you're referencing a resource, either use a real URL the teacher provided or leave it as plain text for them to fill in.

### 5. Handle optional sections smartly

Sections marked `default: "omit"` in the template (typically `discussion`) should stay omitted unless the teacher mentioned a discussion, debate, or forum prompt. When they did, pass `include_sections: ["discussion"]` and fill the corresponding slot.

Sections marked `default: "include"` (typically `assessment`) stay on unless the teacher explicitly says "no assessment for this lesson." Then pass `omit_sections: ["assessment"]`.

Don't ask the teacher about optional sections by name — let their words drive the choice. If they mentioned "tomorrow we'll debate the impact of dams" → discussion section on. If they said "this is purely an intro lesson, no graded work" → assessment section off.

### 6. Preview before creating the draft

Show the teacher a brief outline of what each slot will contain, plus the suggested competency alignment if you identified one in step 3:

```
Lesson page draft — "The Water Cycle"

• About: Watershed geography + evapotranspiration cycle
• To: Diagram a local watershed, explain evapotranspiration
• Concepts: Watershed, evapotranspiration, runoff, precipitation
• Resources: USGS watershed tool, [reading TBD]
• Tasks: Map your home watershed (worksheet), share with class
• Discussion: (omitted — not requested)
• Assessment: Friday quiz link (placeholder)

Suggested competency focus: Knowledge-Based Reasoning + Systems Thinking.
Want me to call these out explicitly in the lesson, or keep them woven into the content?

Ready to create this as a draft in Canvas?
```

Wait for approval or revision requests. If the teacher asks for changes, iterate on the affected slots without regenerating everything.

If the teacher wants competencies called out explicitly, add a short paragraph at the end of the `to` slot referencing them by name (e.g., `<p>This lesson exercises <strong>Knowledge-Based Reasoning</strong> and <strong>Systems Thinking</strong> through the watershed mapping task.</p>`). If they want them kept implicit, the slot content stays as you generated it — the competency lens already shaped it.

If no competency framework was configured (step 3 was skipped), omit the "Suggested competency focus" line from the preview.

### 7. Create the draft in Canvas

Call `create_page` with:

```
create_page(
  course_identifier: "<course code or id>",
  title: "<lesson title>",
  template: "lesson",
  slots: { about: "...", to: "...", concepts: "...", ... },
  include_sections: [...],  // only if the teacher wanted a default-omit section
  omit_sections: [...]      // only if the teacher wanted to drop a default-include section
)
```

The page is created as a draft (`published: false` — enforced by the MCP). Confirm to the teacher:

> Created draft lesson page **The Water Cycle** in DSGN 9. It's saved as a draft — open it in Canvas at <Canvas link>, review, and click **Publish** when you're ready.

If `create_page` returns `template_applied: null`, surface that — it means the school config didn't load and no template was applied.

**Never tell the teacher you "published" the page.** It's a draft until they publish it manually in Canvas.

### 8. Module placement (only if asked)

If the teacher asked to put this lesson in a specific module ("add it to Week 3" or "right after the prototyping lesson"), call `list_modules(course_identifier, include_items: true)` to find the right module + position, then `add_module_item` with `type: "Page"` and the new page's URL slug in `content_id`.

If they didn't mention module placement, don't add it — they'll drag it where they want in the Canvas UI.

## Updating an existing lesson

If the teacher says "add a discussion to that lesson page" or "rewrite the tasks section" — use `edit_page_content` with the template machinery, NOT `create_page`:

```
edit_page_content(
  course_identifier: ...,
  page_url: "<existing-slug>",
  template: "lesson",
  slots: { ...all slots, with the changes... },
  include_sections: ["discussion"],   // if adding the discussion accordion
)
```

You must provide **all slots** in the rebuild, not just the changed ones — the MCP doesn't preserve previous slot content from the on-Canvas HTML. If you generated the lesson earlier in this conversation, you already have all the slot content. If you didn't, call `get_page_content` first and reconstruct the slot values from what's visible, OR ask the teacher what content should stay vs. change.

## Common mistakes to avoid

- **Don't use `create_page` to update an existing lesson** — it creates a duplicate with a `-2` slug.
- **Don't fill slots with template-like prose** ("Here, students will learn about X") — the template provides the chrome; slots get the actual content.
- **Don't hardcode Franklin competencies / structure** in your output. Use `list_competencies` / `list_page_templates` to discover what the school provides. Other schools using this skill should get equally relevant output.
- **Never publish.** The MCP forces `published: false`; don't try to flip it, don't suggest you have, and don't tell the teacher you "published" anything. Teachers publish manually in Canvas after reviewing the draft.
- **Don't make up Canvas URLs.** If you reference a "syllabus" or "module 3," use the actual Canvas URLs from `list_modules` / `get_course_details` — not constructed paths.
