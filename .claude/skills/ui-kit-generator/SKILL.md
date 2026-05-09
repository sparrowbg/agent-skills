---
name: ui-kit-generator
description: Generates a complete SaaS UI kit and agent dashboard in a single self-contained HTML file using Tailwind CSS. Includes primitives, patterns, and application UI. Use when user requests UI, dashboard, SaaS interface, or design system generation.
---

# UI KIT GENERATOR SKILL

## PURPOSE
You are a UI system compiler that generates a complete SaaS design kit.

Output MUST be a SINGLE HTML file using Tailwind CSS.

---

## COMPATIBILITY MODE
This skill works in:
- Claude Code (filesystem skill execution)
- Codex CLI (skill folder execution)
- OpenAI-style agent runtimes (system prompt injection)

---

## CORE OUTPUT RULE

You MUST output ONLY ONE HTML file.

No explanations.
No markdown.
No multiple files.

---

## SYSTEM DESIGN RULE

The output must ALWAYS include ALL layers:

### 1. SYSTEM LAYER
- header / navigation
- system metrics

### 2. UI PRIMITIVES LAYER
Must include:
- button (primary, secondary, danger)
- input
- textarea
- select
- checkbox/toggle
- badge/status indicators

### 3. UI PATTERNS LAYER
Must include:
- form
- modal
- card
- dialog pattern

### 4. APPLICATION LAYER
Must include:
- agent creation panel
- dashboard overview
- logs stream
- system controls (pause/restart/kill)

---

## NON-DESTRUCTIVE MERGE RULE

Never remove existing UI layers.

If a layer exists, you MUST:
- keep it
- extend it
- integrate it

NOT replace it.

---

## DESIGN SYSTEM RULE

- Tailwind CSS only (CDN allowed)
- Dark UI (zinc palette)
- Consistent spacing scale (4–64)
- rounded-xl default radius
- minimal but dense SaaS aesthetic
- inspired by Linear + Vercel

---

## MINIMUM COMPLEXITY REQUIREMENT

Output must contain:
- ≥ 12 distinct UI components
- ≥ 6 primitives
- ≥ 3 patterns
- ≥ 3 application panels

---

## FAILURE RULES

DO NOT:
- omit inputs/forms ❌
- output only dashboard ❌
- output only components ❌
- create inconsistent styles ❌
- split into multiple files ❌

---

## SUCCESS CRITERIA

A valid output:
- looks like a real SaaS product
- is usable as a UI starter kit
- contains full interaction primitives
- is fully self-contained HTML

---

## EXECUTION BEHAVIOR

When invoked:
1. build primitives first
2. build patterns second
3. build application UI last
4. merge everything into one HTML file
5. ensure no duplication

---

## FINAL RULE

Return ONLY HTML.