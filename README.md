# canvas-mcp-skills

Claude Code/Desktop skills that work with the [canvas-mcp](https://github.com/jaymesdec/canvas-mcp-node) MCP server. Generic by design — adapt to whatever competency framework your school config defines (or no framework at all). Built at Franklin School (Jersey City, NJ) but reusable by any school using Canvas LMS.

## What's a skill?

A markdown file with frontmatter that lives at `~/.claude/skills/<skill-name>/SKILL.md`. Claude reads it when you trigger one of its trigger phrases ("plan a lesson", "create an assessment", etc.) and follows the documented workflow — calling the MCP's Canvas tools, generating content, publishing pages.

These skills don't run code; they're prompts that steer Claude's behavior. The MCP server (canvas-mcp) is what actually talks to Canvas.

## Prerequisites

- **Claude Desktop** (or any client that loads `~/.claude/skills/`)
- **canvas-mcp v0.3.11 or later** installed via [the .mcpb installer](https://github.com/jaymesdec/canvas-mcp-node/releases/latest)
- A school config loaded by the MCP (Franklin teachers: default; other schools: point `SCHOOL_CONFIG` at your own JSON — see canvas-mcp-node README)

## Install

```bash
# Clone the repo somewhere convenient
git clone https://github.com/jaymesdec/canvas-mcp-skills ~/code/canvas-mcp-skills

# Symlink the skills you want into Claude's skill directory
mkdir -p ~/.claude/skills
ln -s ~/code/canvas-mcp-skills/skills/plan-lesson ~/.claude/skills/plan-lesson
```

Or copy individual skill directories into `~/.claude/skills/` if you don't want a live symlink.

Restart Claude Desktop. The skill becomes active.

## Available skills

| Skill | What it does | Trigger phrases |
|---|---|---|
| [`plan-lesson`](skills/plan-lesson/) | Plan a Canvas lesson page using the school's lesson template. Generates content for all required slots; teacher reviews; published as a draft Canvas page. | "plan a lesson", "create a lesson", "draft a lesson page" |
| [`plan-assessment`](skills/plan-assessment/) | Plan a Canvas assessment page (test, quiz, project, presentation, essay, lab, portfolio) using the school's assessment template. Generates per-slot content including the grade boundaries table and AI-use policy appropriate to the assessment type. | "plan an assessment", "create a test/quiz/project page", "draft an assessment" |
| [`plan-module`](skills/plan-module/) | Plan an entire Canvas module: outcomes page, a sequence of lesson pages, a summative assessment, and the Canvas Module itself with all items added in order. Enforces the school's module naming convention. Requires canvas-mcp v0.3.12+. | "plan a module", "design a module", "draft a unit" |
| [`plan-course`](skills/plan-course/) | Plan an entire Canvas course's structure: a sequence of modules across the academic calendar, each module's Outcomes page drafted, with optional foundational pages (Start Here, More Resources). Defers lesson + assessment drafting to follow-up `plan-module` runs. Requires canvas-mcp v0.3.12+. | "plan a course", "scaffold a course", "set up the course structure", "outline a year of curriculum" |
| [`grade-submissions`](skills/grade-submissions/) | Grade every submission for a Canvas assignment. Drafts scores + feedback, presents a review table, dry-runs before writing. Handles text entries, Google Docs/Slides (via Claude Drive MCP), file uploads (notebooks, PDFs, code, images). Rubric-aware. | "grade submissions", "grade this assignment", "score student work" |
| [`create-quiz`](skills/create-quiz/) | Build a Canvas Quiz (the actual graded object with questions) from a Canvas page, PDF, Google Doc, or pasted text. Generates AI-drafted questions with mandatory source-quote citations OR extracts existing ones from a pre-written quiz doc. Teacher reviews and edits before the quiz is created. Composes with `plan-assessment` for quiz-style assessments. Classic Quizzes only. | "create a quiz", "make a quiz", "build a Canvas quiz", "convert this quiz to Canvas" |
| [`setup-rubric`](skills/setup-rubric/) | Set up a Canvas rubric for an assessment by reusing an existing one (default), adapting an existing one, or creating a new one from scratch. Reuse-first by design since most courses have rubrics applied to multiple assignments. Composes with `plan-assessment` (for criteria source) and `grade-submissions` (which grades per-criterion when a rubric is attached). Requires canvas-mcp v0.3.13+. | "add a rubric", "create a rubric", "set up a rubric", "use the [name] rubric for X" |

### Roadmap

(All originally-planned skills are now built. New skills will land here as new workflows surface.)

### Skill design principles

- **Never publish.** Every page / quiz / assignment created by these skills lands in Canvas as a **draft**. The teacher reviews and clicks Publish themselves. The MCP server enforces this at the tool layer (`published: false` is forced on create), and the skills reinforce it in their workflow language. Skills should never describe their output as "published" — only ever as drafts.
- **Generic by default.** Skills discover school-specific structure via `list_page_templates`, `list_competencies`, and the loaded school config. No hardcoded Franklin terminology.
- **Don't over-ask.** Skills should figure out what they can from the teacher's initial request and only ask about things they genuinely need (course confirmation, total points, AI policy, etc.). Module placement, publish flags, and HTML structure should never need a question.

## For other schools

These skills don't hardcode Franklin. They:

- Call `list_competencies` and adapt to whatever framework is configured (or skip competency-alignment if none)
- Call `list_page_templates` to discover what content slots the school's `lesson` / `assessment` / `default` templates expect
- Use whatever academic calendar / term names live in your school config

If your school config has different template names or competencies, the skills will pick them up automatically. No code changes needed.

## Contributing

Open issues / PRs at https://github.com/jaymesdec/canvas-mcp-skills.

When adding a new skill:

1. Create `skills/<name>/SKILL.md` with `name:` / `description:` frontmatter (the description is what triggers Claude).
2. Use `list_page_templates` and other tools to discover school-specific structure — don't hardcode slot names or competencies.
3. Add a row to the "Available skills" table in this README.
4. If the skill needs supporting files (templates, reference data), put them in `skills/<name>/` alongside the SKILL.md.

## License

MIT. See [LICENSE](LICENSE).
