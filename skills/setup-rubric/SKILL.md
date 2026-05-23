---
name: setup-rubric
description: Set up a Canvas rubric for an assessment — by reusing an existing rubric from the course, by adapting an existing one, or by creating a new one. Reuse-first by design, since most courses have rubrics they apply to multiple assignments. Use when the teacher says "add a rubric to this assignment", "create a rubric", "use the [name] rubric for X", "I need a rubric for the lab report", or has an assignment that needs grading criteria. Attaches the rubric to the Assignment/Quiz/Discussion in Canvas; never publishes anything (rubric attachment is a metadata change, not a student-visible publish action).
---

# Setup Rubric

Get a rubric attached to an assessment, with reuse as the default mode. Three workflow branches:

1. **Reuse** — attach an existing course rubric to the assignment as-is
2. **Adapt** — start from an existing rubric, change criteria/ratings for this assignment, save as new
3. **Create new** — draft criteria + ratings from scratch when nothing existing fits

Rubric attachment is not the same as publishing a grade — it's a configuration change on the assignment. But the skill still surfaces "review in Canvas before students see it" because the rubric is visible to students once the assignment itself is published.

## When to use

A teacher needs grading criteria on an assignment. Common phrasings:

- "Add a rubric to the watershed project"
- "I need a rubric for the lab report"
- "Use the Engineering Design Project rubric for this assignment"
- "Make a rubric for the essay — clarity, evidence, voice"
- "Adapt the standard presentation rubric for the gallery walk"

## When NOT to use

- **Just describing rubric criteria in an assessment page** — that's `plan-assessment`'s `structure_and_grading` slot. `setup-rubric` creates the actual Canvas rubric object that powers SpeedGrader and rubric-based grading in `grade-submissions`.
- **Modifying an existing rubric's criteria globally** — Canvas's rubric update API is messy when other assignments use the same rubric. This skill always creates a NEW rubric when criteria change (the "Adapt" path), preserving the original for other assignments.
- **Removing a rubric from an assignment** — out of scope for v1. The teacher does this in Canvas's assignment edit UI.

## Prerequisites

- canvas-mcp **v0.3.13 or later** (needs `create_rubric` and `create_rubric_association`).
- A target Canvas Assignment, Quiz, or Discussion in the course (the rubric attaches to one of these).
- Optional: a school config with a competency framework (rubrics commonly map criteria to competencies).

## Workflow

### 1. Capture the target assessment

Get from the teacher (asking only what's missing):

- **Which Canvas course?** Code or numeric id.
- **Which assessment?** The Canvas Assignment, Quiz, or Discussion that needs the rubric. Accept a URL (e.g., `https://.../courses/914/assignments/12345`), a name ("the watershed project"), or a numeric id.

If the teacher named the assessment without an id, call `list_assignments(course_identifier)` (or `list_modules` with `include_items: true`) and confirm the match.

### 2. Show existing rubrics in the course

Call `list_all_rubrics(course_identifier, include_criteria: true)`. Present the results to the teacher in a compact summary:

```
Course DSGN 9 has 6 existing rubrics:

1. [id 101] Engineering Design Project Rubric — 4 criteria, 16 pts total
   Criteria: Problem Framing • Iteration & Testing • Communication • Process Reflection

2. [id 102] Lab Report Rubric — 5 criteria, 20 pts total
   Criteria: Hypothesis • Methods • Data Analysis • Conclusion • Scientific Voice

3. [id 103] Class Presentation Rubric — 4 criteria, 16 pts total
   Criteria: Content Mastery • Visual Design • Delivery • Q&A Engagement

4. [id 104] Essay Rubric (Persuasive) — 4 criteria, 16 pts total
   Criteria: Thesis Clarity • Evidence & Reasoning • Voice & Style • Mechanics

5. [id 105] Watershed Project Rubric — 5 criteria, 20 pts total
   Criteria: Research Depth • Map Quality • Analysis • Communication • Collaboration

6. [id 106] Generic Project Rubric — 3 criteria, 12 pts total
   Criteria: Concept Development • Execution • Reflection

Which one fits this assignment? Options:
  • "Attach #5 as-is" — use Watershed Project Rubric exactly as is
  • "Adapt #1" — start from Engineering Design rubric, modify for this assignment, save as new
  • "Make a new one" — draft criteria from scratch for this assignment
  • "None of these — show me more details on #X" — get full criteria + ratings for one rubric
```

The summary shows just enough for the teacher to recognize useful rubrics. Full criterion text + rating descriptors are available via `get_rubric_details(course_identifier, rubric_id)` on request.

If the course has **no rubrics** (the `list_all_rubrics` response has 0 entries), skip directly to the "Create new" path (step 4) — there's nothing to reuse.

### 3. Three branches

#### 3a. Reuse path: "Attach #N as-is"

The teacher picked one of the existing rubrics. No drafting needed. Confirm the choice:

> Attaching **Watershed Project Rubric** (5 criteria, 20 pts) to **Watershed Field Study Project** (assignment 12345). Use for grading: yes. Hide score total from students: no. Proceed?

On confirmation, call `create_rubric_association`:

```
create_rubric_association(
  course_identifier: <course>,
  rubric_id: 105,
  association_type: "Assignment",  // or "Quiz" / "Discussion"
  association_id: 12345,
  use_for_grading: true,
  hide_score_total: false
)
```

Confirm to the teacher:

> Attached **Watershed Project Rubric** to **Watershed Field Study Project**. It's now visible to graders in SpeedGrader and to students on the assignment page. The grade-submissions skill will use it automatically when you grade.

#### 3b. Adapt path: "Adapt #N"

The teacher picked an existing rubric as a starting point. Call `get_rubric_details(course_identifier, rubric_id)` to load the full criteria + ratings.

Present the existing rubric in an editable table and ask what they want to change:

```
Adapting **Engineering Design Project Rubric** for **Bridge Build Project**:

| Criterion             | Description (long)                                  | Ratings (Excellent/Proficient/Developing/Beginning)              | Points |
|----------------------|----------------------------------------------------|------------------------------------------------------------------|--------|
| Problem Framing       | How clearly the student defines the design problem | 4 / 3 / 2 / 1                                                    | 4      |
| Iteration & Testing   | Evidence of testing and refining the design        | 4 / 3 / 2 / 1                                                    | 4      |
| Communication         | Clarity of process documentation                   | 4 / 3 / 2 / 1                                                    | 4      |
| Process Reflection    | Depth of reflection on what was learned            | 4 / 3 / 2 / 1                                                    | 4      |

What changes for this assignment? Options:
  • Add a criterion ("add 'Structural Integrity' worth 4 pts")
  • Remove a criterion ("drop Process Reflection")
  • Modify a criterion ("change Communication's long_description to focus on technical drawings")
  • Modify a rating ("at Excellent level, what does Iteration look like for a bridge specifically?")
  • Rename the rubric ("save as 'Bridge Build Rubric'")
  • Looks good as-is — "save and attach"
```

Iterate with the teacher. Default rating scale is 4 levels (Excellent / Proficient / Developing / Beginning) at points 4 / 3 / 2 / 1; teachers can specify different scales (e.g., 5 levels, or 3 levels: Mastery / Proficient / Developing).

When the teacher approves, **always save as a NEW rubric** (don't update the existing one — Canvas's update endpoint behaves unpredictably when other assignments use the same rubric). Pick a new title based on the adaptation:

```
create_rubric(
  course_identifier: <course>,
  title: "Bridge Build Rubric (adapted from Engineering Design Project Rubric)",
  criteria: [
    { description: "Problem Framing", long_description: "...", ratings: [...] },
    // … etc.
  ],
  associate_with: {
    association_type: "Assignment",
    association_id: 12345,
    use_for_grading: true
  }
)
```

#### 3c. Create new path

When no existing rubric fits, draft from scratch. Gather context from three sources to inform the criteria:

1. **The assessment description.** Call `get_assignment_details(course_identifier, assignment_id)` and read the description. What is this assessment actually evaluating?
2. **A linked plan-assessment page** (if one exists). Call `list_pages(course_identifier)` and look for a page whose title matches the assignment. If found, call `get_page_content` and read the `structure_and_grading` slot — the teacher may have already written grade boundaries / criteria in prose form that the rubric should mirror.
3. **The competency framework** (if a school config is loaded). Call `list_competencies`. Ask the teacher which competencies this rubric should measure — 2–4 typically. Each chosen competency becomes a likely criterion.

Synthesize 3–5 criteria for the rubric. Each criterion gets:

- **description** — short name (1–4 words)
- **long_description** — a sentence explaining what the criterion measures
- **ratings** — default 4 levels with points 4 / 3 / 2 / 1, each with a short label ("Excellent") and a `long_description` explaining what work earns that rating

Draft and present in a table:

```
Proposed rubric for **Watershed Field Study Project**:

Title: Watershed Field Study Rubric

| Criterion              | Long description                                                                | Excellent (4)                                       | Proficient (3)                                       | Developing (2)                                  | Beginning (1)                              |
|------------------------|---------------------------------------------------------------------------------|-----------------------------------------------------|------------------------------------------------------|-------------------------------------------------|--------------------------------------------|
| Field Observation     | Quality of direct observations gathered in the field                            | Detailed obs at 5+ locations w/ photos & sketches   | Detailed obs at 3–4 locations                        | Obs at 1–2 locations, limited detail            | No field data or only desk research        |
| Map Quality           | Accuracy & legibility of the watershed map                                      | Scale, labels, key features all present + clear     | Most labels & features present                       | Some labels, missing scale or key               | Map missing or unlabeled                   |
| Analysis              | Depth of explanation tying observations to watershed science concepts            | Multi-factor analysis citing class concepts         | Explains 1–2 watershed concepts using observations   | Lists concepts without connecting to data       | No analysis; describes only what was seen  |
| Communication         | Clarity of written report and visuals                                            | Logical structure, polished visuals, no errors      | Clear structure, minor visual or writing issues      | Hard to follow in places                        | Disorganized, hard to understand           |

Competency alignment: Knowledge-Based Reasoning, Systems Thinking, Storytelling/Communication

Total points: 16 (4 criteria × 4 max points)
Free-form criterion comments: ON (graders can write per-criterion feedback in SpeedGrader)

Sound right? Edit the criteria, ratings, or descriptors before I create it.
```

The teacher iterates until satisfied. Then create + associate in one call:

```
create_rubric(
  course_identifier: <course>,
  title: "Watershed Field Study Rubric",
  free_form_criterion_comments: true,
  criteria: [
    {
      description: "Field Observation",
      long_description: "Quality of direct observations gathered in the field",
      ratings: [
        { description: "Excellent", long_description: "Detailed observations at 5+ locations with photos & sketches", points: 4 },
        { description: "Proficient", long_description: "Detailed observations at 3–4 locations", points: 3 },
        { description: "Developing", long_description: "Observations at 1–2 locations, limited detail", points: 2 },
        { description: "Beginning", long_description: "No field data or only desk research", points: 1 }
      ]
    },
    // … other criteria
  ],
  associate_with: {
    association_type: "Assignment",
    association_id: 12345,
    use_for_grading: true
  }
)
```

### 4. Confirm to teacher

After any of the three paths:

> Set up rubric **Watershed Field Study Rubric** on **Watershed Field Study Project** (assignment 12345). 4 criteria, 16 points total. Graders see it in SpeedGrader; students see it on the assignment page once the assignment is published.
>
> Next: when you're ready to grade, the **grade-submissions** skill will use this rubric automatically — it'll fetch the rubric, present each criterion's ratings in the draft table, and post per-criterion scores + comments back to Canvas.

## Quality requirements

- **Reuse-first thinking.** Always show existing rubrics in step 2 before drafting anything. Teachers often forget what they have; surfacing it saves duplication. If the teacher's request is clearly about creating new ("design a rubric from scratch for...") still show existing as a sanity check — it takes one tool call and one sentence.
- **Never update an existing rubric in place.** The Adapt path always creates a NEW rubric, leaving the original untouched. Canvas's rubric-update behavior with associated assignments is unreliable; copy-and-modify is the safe pattern.
- **Default rating scale: 4 levels.** Excellent / Proficient / Developing / Beginning at points 4 / 3 / 2 / 1. Teachers can override (e.g., 5-level or 3-level scales), but the default is robust and widely understood.
- **Criterion long_descriptions matter.** Every criterion should have a long_description explaining what it measures. Empty long_descriptions create grading friction in SpeedGrader.
- **Rating long_descriptions distinguish levels.** A rubric with only short labels ("Excellent / Proficient / Developing / Beginning") and no rating-level descriptors gives graders nothing to anchor judgments to. Each rating's `long_description` should describe what work earns that score for that specific criterion.
- **Competency alignment is optional but valued.** When a school config has a framework and the teacher wants alignment, name the competencies in the rubric title or in a criterion's `long_description`. Don't force-fit if the teacher didn't bring it up.

## Skill design principles (per repo conventions)

- **Never publishes content visible to students.** Rubric attachment is a configuration change; whether the rubric is *visible* to students depends on whether the assignment is published. The skill doesn't publish anything new — it sets up grading infrastructure.
- **Generic by default.** Reads competency framework via `list_competencies` only if the teacher wants competency-aligned criteria. Works fine without any school config loaded.
- **Don't over-ask.** Confirm course + target assessment. Use sensible defaults for rating scales, free_form_criterion_comments, and use_for_grading. Let the teacher adjust in the draft table.
- **Reuse-first.** Don't go to drafting when an existing rubric fits. Save the teacher time and prevent rubric proliferation.

## Linkage with other skills

- **`plan-assessment`** writes a `structure_and_grading` slot on the assessment description page that often describes grading criteria in prose form. When `setup-rubric` runs on an assignment with a matching plan-assessment page (detected via `list_pages` title match), it should read that slot and propose criteria that mirror what the teacher already wrote.
- **`grade-submissions`** is rubric-aware — when an assignment has an attached rubric, it grades per-criterion. The rubric set up by this skill becomes the grading scaffolding for that workflow. (See `skills/grade-submissions/SKILL.md` step 2 for the rubric-attached grading path.)
- **`plan-module`** sometimes specifies that a module's summative assessment should use a particular competency frame. When run as part of a UbD plan-module workflow, `setup-rubric` should respect the Stage 1 competencies the module identified and propose criteria aligned to them.

## Common mistakes to avoid

- **Don't skip step 2.** Even if the teacher said "create a rubric," show existing ones first. Some teachers ask for create because they forgot what's in the course. A 5-second list_all_rubrics call prevents rubric proliferation.
- **Don't update an existing rubric in place during the Adapt path.** Always create a NEW rubric. Canvas's update behavior with associated assignments is unreliable.
- **Don't write criteria with empty `long_description`s.** Graders need the descriptor; "Clarity" without explanation is useless in SpeedGrader.
- **Don't write rating labels without `long_description`s for each.** "Excellent / Proficient / Developing / Beginning" alone gives graders nothing to anchor judgments to.
- **Don't force-fit competencies into criteria.** When the school has a framework, ask the teacher which competencies apply. Don't list all 9 just because they exist.
- **Don't tell the teacher you've "published" anything.** Rubric attachment is a configuration change, not a publish action. Use phrasing like "set up", "attached", "configured" — never "published."
- **Don't make up Canvas URLs.** When linking back to the assignment after rubric setup, use the assignment's actual URL from `get_assignment_details` response.
- **Don't fabricate criteria.** When you don't have enough source context (no assignment description, no plan-assessment page, no clear teacher direction), ASK the teacher what they want the rubric to measure rather than guessing.

## Example

**Teacher:** "Set up a rubric for the watershed project — assignment 12345 in DSGN 9. We have a Watershed Project Rubric in the course from last year; use that."

**Skill:**

1. Confirms course 914, target Assignment 12345 (Watershed Field Study Project).
2. Calls `list_all_rubrics(914)` → finds Watershed Project Rubric at id 105.
3. Confirms the teacher's choice: "Attaching **Watershed Project Rubric** (5 criteria, 20 pts) to **Watershed Field Study Project**. Use for grading: yes. Hide score total: no. Proceed?"
4. Teacher: "yes"
5. Calls `create_rubric_association(course_identifier: 914, rubric_id: 105, association_type: "Assignment", association_id: 12345, use_for_grading: true)`.
6. Confirms: "Attached **Watershed Project Rubric** to **Watershed Field Study Project**. It's now visible in SpeedGrader. The grade-submissions skill will use it when you grade."

Total elapsed teacher time: <1 minute. No drafting needed.
