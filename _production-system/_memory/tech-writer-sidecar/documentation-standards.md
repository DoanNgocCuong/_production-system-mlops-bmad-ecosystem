# Technical Documentation Standards for PSM

CommonMark standards, technical writing best practices, and style guide compliance for the Production Systems & MLOps module.

## User Specified CRITICAL Rules - Supersedes General CRITICAL Rules

None

## General CRITICAL RULES

### Rule 1: CommonMark Strict Compliance

ALL documentation MUST follow CommonMark specification exactly. No exceptions.

### Rule 2: NO TIME ESTIMATES

NEVER document time estimates, durations, level of effort or completion times for any workflow, task, or activity unless EXPLICITLY asked by the user. Focus on workflow steps, dependencies, and outputs.

## CommonMark Essentials

**Headers:**
- Use ATX-style ONLY: `#` `##` `###` (NOT Setext underlines)
- Single space after `#`: `# Title` (NOT `#Title`)
- No trailing `#`: `# Title` (NOT `# Title #`)
- Hierarchical order: Don't skip levels (h1→h2→h3, not h1→h3)

**Code Blocks:**
- Use fenced blocks with language identifier:
  ````markdown
  ```javascript
  const example = 'code';
  ```
  ````
- NOT indented code blocks

**Lists:**
- Consistent markers within list: all `-` or all `*` (don't mix)
- Proper indentation for nested items

**Links:**
- Inline: `[text](url)` — NO bare URLs without `<>` brackets

**Line Breaks:**
- Two spaces at end of line + newline, OR blank line between paragraphs
- NO single line breaks (they're ignored in HTML rendering)

## Mermaid Diagrams

**Critical Rules:**
1. Always specify diagram type on first line
2. Use valid Mermaid v10+ syntax
3. Test syntax before outputting
4. Keep focused: 5-10 nodes ideal, max 15

**Diagram Types:**
- **flowchart** — Process flows, decision trees, workflows
- **sequenceDiagram** — API interactions, message flows
- **stateDiagram-v2** — State machines, lifecycle stages

## Style Guide Principles

### Core Writing Rules

- **Task-Oriented**: Write for user GOALS, not feature lists
- **Start with WHY**: Then HOW — every doc answers "What can I accomplish?"
- **Active voice**: "Click the button" NOT "The button should be clicked"
- **Present tense**: "The function returns" NOT "The function will return"
- **Second person**: "You configure" NOT "Users configure"

### Structure

- One idea per sentence
- One topic per paragraph
- Headings describe content accurately
- Examples follow explanations

## Production Documentation Types

### README

- What (overview), Why (purpose), How (quick start)
- Installation, Usage, Contributing
- Under 500 lines

### Runbook

- Step-by-step operational procedures
- Decision trees for incident scenarios
- Escalation contacts and paths
- Troubleshooting section

### Architecture Decision Record (ADR)

- Context: Why this decision was made
- Decision: What was chosen
- Consequences: Pros and cons
- Status: Proposed / Accepted / Deprecated

### Postmortem

- Timeline of events
- Root cause analysis (5-Why)
- Impact assessment
- Action items with owners

## Quality Checklist

Before finalizing ANY documentation:

- [ ] CommonMark compliant
- [ ] NO time estimates
- [ ] Headers in proper hierarchy
- [ ] All code blocks have language tags
- [ ] Links work and have descriptive text
- [ ] Mermaid diagrams render correctly
- [ ] Active voice, present tense
- [ ] Task-oriented (answers "how do I...")
- [ ] Examples are concrete and working
