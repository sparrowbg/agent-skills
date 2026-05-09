# OUTPUT SPEC — UI KIT GENERATOR

## OUTPUT FORMAT

The output MUST be a single HTML file.

No exceptions.

---

## REQUIRED STRUCTURE

The HTML file must follow this structure:

### 1. HEAD SECTION
Must include:
- Tailwind CDN script
- optional Tailwind config extension (if needed)

---

### 2. SYSTEM UI (TOP LEVEL)
Must include:
- navigation bar
- system title / identity
- system metrics (agents, tasks, latency, memory)

---

### 3. UI PRIMITIVES SECTION
Must include visual examples of:

- Button (primary / secondary / danger)
- Input field
- Textarea
- Select dropdown
- Checkbox / toggle
- Status badge

Each primitive must be visually grouped.

---

### 4. UI PATTERNS SECTION
Must include:

- Form pattern (input + select + submit)
- Modal pattern (static representation allowed)
- Card component
- Dialog-style layout block

Patterns must show reuse of primitives.

---

### 5. APPLICATION SECTION (CORE SaaS SYSTEM)

Must include at minimum:

- Agent creation panel (form-based)
- Live logs stream (scrollable)
- Task / execution pipeline
- System control panel (pause / restart / kill)

---

### 6. DESIGN CONSISTENCY RULE

All UI must use:

- Tailwind CSS only
- dark zinc-based palette
- consistent spacing scale
- rounded-xl default radius

---

## VISUAL DENSITY RULE

The UI must NOT feel like a mock.

It must resemble:
- real SaaS product
- internal developer tooling
- production-grade agent system

---

## OUTPUT QUALITY CHECK

Before final output, ensure:

- No missing primitives
- No missing forms
- No missing system metrics
- No isolated UI fragments
- All layers integrated

---

## FINAL OUTPUT RULE

Return ONLY HTML.

No markdown.
No explanation.
No additional text.