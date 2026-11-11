# ainidia-skills — Repository Specification

**Version**: 1.0  
**Date**: 2026-11-11  
**Status**: APPROVED — ready for implementation  
**Companion spec**: `../../../ainidia/docs/specs/2026-11-11-ainidia-app.md`  
**Repository path**: `../ainidia-skills/` (sibling of the app repo)

---

## 1. Purpose

`ainidia-skills` is the companion content repository for the Ainidia Android app. It contains:

- **AI Personas** — personality definitions for the AI tutor
- **Skills** — prompt templates that define how the AI teaches, corrects, and converses
- **Topics** — conversation scenario prompts
- **Memory** — prompts for summarizing sessions into episodic memory
- **Schemas** — JSON schemas for validating all content files

The app reads from this repository at two points:
1. **Build time** — a copy script bundles the content into the app's `assets/default_skills/`
2. **Runtime** — the GitHub Contents API fetches updates without a new app release

---

## 2. Repository Structure

```
ainidia-skills/
├── README.md
├── manifest.json                      # Master content index
├── schema/
│   ├── manifest.schema.json
│   ├── skill.schema.json
│   └── persona.schema.json
├── personas/
│   ├── andi-friendly.json
│   └── prof-precise.json
├── skills/
│   ├── correction/
│   │   ├── immediate.json             # Mode 1
│   │   └── reply-then-fix.json        # Mode 2
│   ├── conversation/
│   │   └── natural-flow.json          # Mode 3
│   ├── feedback/
│   │   ├── session-report.json        # End-of-session feedback (Mode 3)
│   │   └── mistake-analysis.json      # Pattern detection
│   └── topics/
│       ├── daily-life.json
│       ├── tech-discussion.json
│       ├── travel.json
│       └── business.json
├── memory/
│   └── summarizer.json
└── LICENSE
```

---

## 3. File Formats

### 3.1 manifest.json

Illustrative example — values like `"string"` and `"CORRECTION|CONVERSATION|..."` are placeholders showing shape and allowed values, not literal data. The normative definition is `schema/manifest.schema.json`.

```json
{
  "formatVersion": 1,
  "minAppVersion": "1.0.0",
  "repository": {
    "name": "string",
    "description": "string",
    "author": "string",
    "url": "string"
  },
  "personas": [
    {
      "id": "string",
      "path": "string",
      "version": 1,
      "name": "string",
      "description": "string"
    }
  ],
  "skills": [
    {
      "id": "string",
      "path": "string",
      "version": 1,
      "category": "CORRECTION|CONVERSATION|FEEDBACK|ANALYSIS|TOPIC|MEMORY",
      "name": "string",
      "description": "string",
      "requiredFor": ["MODE_1|MODE_2|MODE_3"]
    }
  ],
  "memory": [
    {
      "id": "string",
      "path": "string",
      "version": 1,
      "name": "string",
      "description": "string"
    }
  ]
}
```

- `formatVersion` increments when the manifest schema itself changes (breaking change)
- `minAppVersion` — app versions below this value should ignore the entire manifest
- Each list entry's `version` increments when the content of that file changes
- `requiredFor` is optional: omit it for skills the user selects manually (topics, feedback). When present it is non-empty and only `MODE_1`, `MODE_2`, `MODE_3` are allowed

### 3.2 skill.schema.json (canonical definition)

Required fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (pattern: `^[a-z0-9-]+$`) | Unique, URL-safe identifier |
| `name` | string | Human-readable name |
| `version` | integer (≥1) | Increments on any content change |
| `schema_version` | integer | Always `1` for v1 format |
| `category` | enum | One of: `CORRECTION`, `CONVERSATION`, `FEEDBACK`, `ANALYSIS`, `TOPIC`, `MEMORY` |
| `system_prompt` | string | Prompt template with `{{variable}}` placeholders |

Optional fields:

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Short description shown in app skill browser |
| `parameters.temperature` | number (0–2) | Overrides app default if present |
| `parameters.max_tokens` | integer (≥100) | Overrides app default if present |
| `parameters.response_delimiter` | string | Token used to split reply vs. correction sections |
| `starter_questions` | string[] (min 1) | Conversation openers; `TOPIC` files use 3 |
| `metadata.author` | string | Author name |
| `metadata.tags` | string[] | Searchable tags |
| `metadata.created` | string (ISO date) | Creation date |
| `metadata.updated` | string (ISO date) | Last modified date |

No additional properties allowed (`additionalProperties: false`).

### 3.3 persona.schema.json (canonical definition)

Required fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier |
| `name` | string | Display name (used in UI and as `{{persona.name}}`) |
| `version` | integer | Increments on any content change |
| `schema_version` | integer | Always `1` |
| `system_identity` | string | Full text injected as persona description in system prompt |
| `greeting` | string | Opening message shown when session starts |

Optional fields:

| Field | Type | Description |
|-------|------|-------------|
| `personality.tone` | string | e.g. "warm, encouraging" |
| `personality.style` | string | Teaching style description |
| `personality.quirks` | string[] | Distinctive behaviors |
| `personality.strictness` | number (0–1) | 0 = very lenient, 1 = very strict |
| `boundaries` | string[] | Behavioral constraints injected into prompt |
| `metadata` | object | author, tags |

---

## 4. Template Variables

All `system_prompt` fields support Mustache-style template variables resolved by the app's `SkillEngine`:

| Variable | Resolves to |
|----------|------------|
| `{{persona.name}}` | Active persona's `name` field |
| `{{persona.personality}}` | Active persona's `system_identity` text |
| `{{user.name}}` | User's name from `UserPersonaEntity` (or "there" if null) |
| `{{user.nativeLanguage}}` | Native language string (or "unknown" if null) |
| `{{user.level}}` | `beginner`, `intermediate`, or `advanced` |
| `{{user.interests}}` | Comma-separated interests list |
| `{{user.commonMistakes}}` | Top 5 mistake patterns, comma-separated descriptions |
| `{{memory.recent}}` | Last 3 episodic memory session notes concatenated |
| `{{#if <var>}}...{{/if}}` | Block included only if variable is non-null and non-empty |

Unknown variables resolve to empty string. No code execution in templates. No nested variables.

---

## 5. Default Content Specifications

### 5.1 Personas

#### `personas/andi-friendly.json`
- **ID**: `andi-friendly`
- **Name**: `Andi`
- **Personality**: Warm, encouraging, slightly playful. Celebrates small wins. Uses relatable metaphors for grammar explanations. Occasionally shares interesting word origins.
- **Strictness**: 0.3 (lenient, focuses on fluency over perfection)
- **Boundaries**: Age-appropriate content; redirects off-topic to English practice; acknowledges being an AI if asked
- **Greeting**: Friendly, casual opening that invites the user to pick a topic

#### `personas/prof-precise.json`
- **ID**: `prof-precise`
- **Name**: `Prof. Lin`
- **Personality**: Precise, formal, academic. Focuses on correctness. Explains grammar rules with linguistic terms when appropriate. Less casual than Andi.
- **Strictness**: 0.8 (catches minor errors, explains rules rigorously)
- **Boundaries**: Same as Andi
- **Greeting**: Professional, welcoming, sets expectations for a structured session

---

### 5.2 Skills

#### `skills/correction/immediate.json`
- **ID**: `correction-immediate`
- **Category**: `CORRECTION`
- **Mode**: Mode 1
- **Behavior**: AI responds ONLY with a correction. No conversational reply. Shows: original phrase → corrected phrase → brief explanation (1 sentence). If no errors found, acknowledges good English briefly.
- **No delimiter** needed (entire response is correction content)
- **Parameters**: `temperature: 0.3` (low — corrections should be accurate, not creative), `max_tokens: 300`

#### `skills/correction/reply-then-fix.json`
- **ID**: `correction-reply-then-fix`
- **Category**: `CORRECTION`
- **Mode**: Mode 2
- **Behavior**: AI first gives a natural conversational reply (1-3 sentences). Then the delimiter `---CORRECTION---` on its own line. Then corrections in bullet format: `• "original" → "corrected" — explanation`. If no errors: `✨ No corrections needed.`
- **Delimiter**: `---CORRECTION---`
- **Parameters**: `temperature: 0.7`, `max_tokens: 500`

#### `skills/conversation/natural-flow.json`
- **ID**: `conversation-natural`
- **Category**: `CONVERSATION`
- **Mode**: Mode 3
- **Behavior**: AI converses naturally. Never corrects English. Asks follow-up questions to keep conversation flowing. Adapts vocabulary complexity to user level. Does NOT include correction delimiter.
- **Parameters**: `temperature: 0.8`, `max_tokens: 400`

#### `skills/feedback/session-report.json`
- **ID**: `feedback-session-report`
- **Category**: `FEEDBACK`
- **Behavior**: Given the full conversation history, generates a structured JSON report. See schema below.
- **Output schema**:
  ```json
  {
    "score": "integer 0-100",
    "summary": "string (1 sentence)",
    "mood": "string",
    "grammar": [{ "original": "string", "corrected": "string", "explanation": "string" }],
    "vocabulary": [{ "type": "improve|learn", "original": "string", "suggestion": "string" }],
    "fluency": ["string observations"],
    "strengths": ["string"],
    "suggested_focus_next": "string",
    "topics_covered": ["string"]
  }
  ```
- **Parameters**: `temperature: 0.3`, `max_tokens: 800`

#### `skills/feedback/mistake-analysis.json`
- **ID**: `feedback-mistake-analysis`
- **Category**: `ANALYSIS`
- **Behavior**: Analyzes conversation for recurring mistake patterns. Returns structured JSON for updating `MistakePatternEntity`.
- **Output schema**:
  ```json
  {
    "patterns": [
      {
        "category": "GRAMMAR|VOCABULARY|TENSE|ARTICLE|PREPOSITION|FLUENCY|OTHER",
        "description": "string",
        "examples": ["string"],
        "severity": "minor|moderate|major"
      }
    ]
  }
  ```
- **Parameters**: `temperature: 0.2`, `max_tokens: 400`

---

### 5.3 Topics

Each topic file has this structure:
```json
{
  "id": "string",
  "name": "string",
  "category": "TOPIC",
  "version": 1,
  "schema_version": 1,
  "description": "string (shown in topic picker)",
  "system_prompt": "string (injected as additional context in conversation)",
  "starter_questions": ["string", "string", "string"],
  "parameters": { "temperature": 0.8 },
  "metadata": {
    "author": "string",
    "tags": ["beginner", "intermediate", "advanced"],
    "created": "YYYY-MM-DD",
    "updated": "YYYY-MM-DD"
  }
}
```

Tags are free-form strings and live in `metadata.tags`; include a level tag (`beginner` / `intermediate` / `advanced`) plus descriptors such as `travel` or `business`. A top-level `tags` key is **not** allowed — the skill schema sets `additionalProperties: false` and validation will reject it.

#### `skills/topics/daily-life.json`
- **ID**: `topic-daily-life`
- **Tags**: `["beginner", "intermediate"]`
- **Scenario**: Everyday topics (morning routines, food, weather, weekend plans, hobbies)
- **3 starter questions** that Andi can use to open the conversation

#### `skills/topics/tech-discussion.json`
- **ID**: `topic-tech`
- **Tags**: `["intermediate", "advanced"]`
- **Scenario**: Technology, programming, gadgets, apps, AI — suited for technically-minded users
- **3 starter questions** on tech themes

#### `skills/topics/travel.json`
- **ID**: `topic-travel`
- **Tags**: `["beginner", "intermediate"]`
- **Scenario**: Airports, hotels, asking for directions, booking reservations, cultural experiences

#### `skills/topics/business.json`
- **ID**: `topic-business`
- **Tags**: `["intermediate", "advanced"]`
- **Scenario**: Meetings, presentations, emails, negotiation, professional introductions

---

### 5.4 Memory

#### `memory/summarizer.json`
- **ID**: `memory-summarizer`
- **Category**: `MEMORY`
- **Behavior**: Given a full conversation, produces a structured JSON summary for episodic memory storage.
- **Output schema**:
  ```json
  {
    "topics_discussed": ["string"],
    "mood": "string",
    "mistakes": [
      { "category": "string", "pattern": "string", "examples": ["string"], "severity": "minor|moderate|major" }
    ],
    "strengths": ["string"],
    "vocabulary_used": ["string"],
    "suggested_focus_next": "string",
    "session_note": "string (max 2 sentences for context continuity)"
  }
  ```
- **Parameters**: `temperature: 0.3`, `max_tokens: 600`
- **Note**: `session_note` must capture actionable continuity context (e.g., "User preparing for a job interview next week" or "User struggled significantly with past perfect tense")

---

## 6. Validation Rules

1. Every file listed in `manifest.json` must exist at the specified `path`
2. Every file must validate against its corresponding JSON schema
3. No two entries in the manifest may share the same `id`
4. Each correction mode has exactly one mandatory skill, and it must exist, validate, be non-deprecated, and declare that mode in `requiredFor` (the app cannot function in that mode without it):
   - `MODE_1` → `correction-immediate`
   - `MODE_2` → `correction-reply-then-fix`
   - `MODE_3` → `conversation-natural`

   Additional skills may also list a mode in `requiredFor` (e.g. a feedback skill required in every mode); only the three above are mandatory.
5. `version` must be a positive integer and must be incremented on any content change
6. The prompt field must not be empty: `system_prompt` for skills and memory files, `system_identity` for personas
7. Topic files must have at least one `starter_questions` entry

---

## 7. Versioning Policy

- **File version**: Increment `version` in the file AND in the manifest entry whenever the file content changes
- **Manifest formatVersion**: Increment only when the manifest schema structure changes (breaking)
- **Schema version**: `schema_version: 1` for all v1 files; increment when the file format itself changes
- **No deletions in manifest**: Mark deprecated entries with `"deprecated": true` rather than removing them; the app skips deprecated entries
- **No renames**: Once an `id` is published, it is permanent. A new version of a skill = new content in the same file, not a new file.

---

## 8. Integration with App

### 8.1 Build-Time Bundling

The app repository contains `scripts/sync_default_skills.sh`:
```bash
#!/bin/bash
set -euo pipefail
# Run from ainidia/ root
# Requires ainidia-skills/ to be cloned as sibling directory

SKILLS_DIR="../ainidia-skills"
ASSETS_DIR="app/src/main/assets/default_skills"

# Validate skills repo exists
if [ ! -d "$SKILLS_DIR" ]; then
  echo "ERROR: ../ainidia-skills not found. Clone it first."
  exit 1
fi

command -v ajv >/dev/null || { echo "ERROR: ajv-cli not installed (npm i -g ajv-cli)"; exit 1; }

# Validate content BEFORE copying — never bundle invalid JSON
cd "$SKILLS_DIR"
ajv compile -s schema/skill.schema.json
ajv compile -s schema/persona.schema.json
ajv compile -s schema/manifest.schema.json
find skills memory -name '*.json' -print0 | while IFS= read -r -d '' f; do
  ajv validate -s schema/skill.schema.json -d "$f" || { echo "FAIL: $f"; exit 1; }
done
for f in personas/*.json; do
  ajv validate -s schema/persona.schema.json -d "$f" || { echo "FAIL: $f"; exit 1; }
done
ajv validate -s schema/manifest.schema.json -d manifest.json || { echo "FAIL: manifest.json"; exit 1; }
cd - > /dev/null

# Clean and copy runtime content only (no .git, no docs, no schema)
rm -rf "$ASSETS_DIR"
mkdir -p "$ASSETS_DIR"
cp -r "$SKILLS_DIR/manifest.json" "$SKILLS_DIR/personas" \
      "$SKILLS_DIR/skills" "$SKILLS_DIR/memory" "$ASSETS_DIR/"

echo "Synced skills at $(date)"
echo "Files copied: $(find "$ASSETS_DIR" -name '*.json' | wc -l) JSON files"
```

This script is run manually before a release build. It is not part of the automated build. Validation failures abort the sync, so a broken file can never reach the app bundle.

### 8.2 Runtime Access

The app's `GitHubSource` reads from this repository via the GitHub Contents API:
- `GET https://api.github.com/repos/{owner}/{repo}/contents/manifest.json?ref={branch}`
- `GET https://api.github.com/repos/{owner}/{repo}/contents/{path}?ref={branch}`

For a public repository: no authentication required.  
For a private repository: `Authorization: token {pat}` header with a PAT that has `contents:read` scope.

The default skill repo URL configured in the app at first launch (if the user has not changed it):
`github.com/tegar/ainidia-skills` *(or the actual published URL — to be confirmed at first release)*

---

## 9. Developer Guide (README outline)

The repository README must cover:
1. What this repository is and how it relates to the Ainidia app
2. How to clone and use with the app (link to app repo setup guide)
3. How to create a custom skill (with a worked example)
4. How to fork this repo and point the app at your fork
5. How to validate your changes locally (recommend `ajv-cli` or similar)
6. Versioning rules
7. How to contribute (PR + review process)
8. License
