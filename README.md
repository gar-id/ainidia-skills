# Ainidia Skills Repository (`ainidia-skills`)

Content catalog for [Ainidia](https://github.com/ainidia/ainidia), an Android English speaking practice application.

This repository is **content-only** (JSON and JSON Schema) — it contains no Kotlin or Android code. It defines the personas, skills (correction modes, conversation flows, feedback generators), topics, and memory summarizers that power Ainidia's AI conversational engine.

---

## 1. What This Repository Is

Ainidia decouples prompt engineering and educational content from application logic. All persona definitions, correction prompts, conversation templates, and topic catalogs are stored as structured JSON documents validated by draft-07 JSON Schemas.

- `schema/` — Draft-07 JSON Schemas for skills, personas, and the catalog manifest.
- `personas/` — AI tutor personas defining identity, tone, strictness, and boundaries.
- `skills/` — Prompt templates organized by category:
  - `skills/correction/` — Immediate drill correction (`MODE_1`) and reply-then-correct (`MODE_2`).
  - `skills/conversation/` — Natural conversation without inline corrections (`MODE_3`).
  - `skills/feedback/` — Post-session report and mistake pattern analysis.
  - `skills/topics/` — Topic discussion guides with starter questions.
- `memory/` — Post-session episodic memory summarizer prompts.
- `manifest.json` — Master registry consumed by the app to index and fetch content.

---

## 2. Relationship to the Ainidia App

The app consumes this repository via two complementary mechanisms:

1. **Build-Time Bundling (Offline Defaults)**:
   Default personas and skills are synced into the Android app's asset directory (`app/src/main/assets/default_skills/`) using `scripts/sync_default_skills.sh` before release builds. The app works fully offline out of the box.

2. **Runtime Updates (GitHub Contents API)**:
   When online, Ainidia checks the repository's `manifest.json` via the GitHub REST API. If newer versions of skills or personas are published, Ainidia downloads and caches them in Room/EncryptedSharedPreferences.

---

## 3. Local Setup

Clone this repository alongside the `ainidia` app repository as sibling directories:

```bash
mkdir -p workspace
cd workspace

# Clone both repositories
git clone https://github.com/ainidia/ainidia.git
git clone https://github.com/ainidia/ainidia-skills.git
```

To sync local skill definitions into the Android app assets:

```bash
cd ainidia
./scripts/sync_default_skills.sh
```

The sync script validates all schemas and content files before copying.

---

## 4. How to Create a Custom Skill

To add a new skill (e.g., a "Job Interview Prep" topic):

1. **Choose an identifier**: Must match `^[a-z0-9-]+$` and never be reused or renamed.
2. **Create the JSON file** under the appropriate directory (`skills/topics/job-interview.json`):

```json
{
  "id": "topic-job-interview",
  "name": "Job Interview Prep",
  "version": 1,
  "schema_version": 1,
  "category": "TOPIC",
  "description": "Practice answering common behavioral and technical interview questions.",
  "system_prompt": "You are conducting a professional mock job interview with {{user.name}}. Their English level is {{user.level}}. Ask one behavioral question at a time using the STAR method (Situation, Task, Action, Result). Maintain an encouraging yet professional demeanor.",
  "parameters": {
    "temperature": 0.7,
    "max_tokens": 400
  },
  "starter_questions": [
    "Tell me about a time you had to resolve a difficult conflict with a colleague.",
    "What is your greatest professional achievement so far?",
    "Where do you see yourself professionally in the next three years?"
  ],
  "metadata": {
    "author": "your-github-username",
    "tags": ["career", "interview", "advanced", "professional"],
    "created": "2026-11-11",
    "updated": "2026-11-11"
  }
}
```

3. **Register the skill in `manifest.json`**:

```json
{
  "id": "topic-job-interview",
  "path": "skills/topics/job-interview.json",
  "version": 1,
  "category": "TOPIC",
  "name": "Job Interview Prep",
  "description": "Practice answering common behavioral and technical interview questions."
}
```

4. **Validate** using `ajv-cli` (see section 6).

---

## 5. Forking and Pointing the App to Your Fork

You can maintain your own catalog of skills and point your Ainidia app directly to it:

1. Fork `ainidia-skills` on GitHub to your account (e.g. `https://github.com/my-user/ainidia-skills`).
2. Add or modify any skills, personas, or topics in your fork.
3. In the Ainidia Android app:
   - Go to **Settings → Developer Options → Skill Repository**.
   - Set the repository owner and name (e.g. `my-user/ainidia-skills`) and optional GitHub Personal Access Token (PAT).
   - Tap **Check for Updates**. The app will fetch `manifest.json` from your fork and load your custom content.

---

## 6. How to Validate Locally

Validation requires `ajv-cli` (v5+):

```bash
npm install -g ajv-cli
```

### Compile Schemas
Schemas must compile against the JSON Schema Draft-07 meta-schema:

```bash
ajv compile -s schema/skill.schema.json
ajv compile -s schema/persona.schema.json
ajv compile -s schema/manifest.schema.json
```

### Validate Content
```bash
# Validate all skills and memory files
find skills memory -name '*.json' -print0 | while IFS= read -r -d '' f; do
  ajv validate -s schema/skill.schema.json -d "$f"
done

# Validate all personas
for f in personas/*.json; do
  ajv validate -s schema/persona.schema.json -d "$f"
done

# Validate manifest
ajv validate -s schema/manifest.schema.json -d manifest.json
```

---

## 7. Versioning Rules

- **Immutable IDs**: Once an `id` is published, it must never change.
- **Content updates**: When editing any skill or persona, increment its integer `version` in the JSON file **and** update the corresponding entry's `version` in `manifest.json`.
- **No deletions**: Never delete a published file. Mark retired items with `"deprecated": true` in `manifest.json`.
- **Schema changes**: Increment `formatVersion` in `manifest.json` if the manifest format changes.
- **Mandatory skills**:
  - `MODE_1` requires `correction-immediate` (`requiredFor: ["MODE_1"]`).
  - `MODE_2` requires `correction-reply-then-fix` (`requiredFor: ["MODE_2"]`).
  - `MODE_3` requires `conversation-natural` (`requiredFor: ["MODE_3"]`).

---

## 8. Contributing

1. Fork the repository and create a feature branch (`git checkout -b feature/new-topic`).
2. Add your skill/persona following schema conventions.
3. Update `manifest.json` with the new entry and incremented versions.
4. Run full validation pass:
   ```bash
   bash -c "for s in skill persona manifest; do ajv compile -s schema/\$s.schema.json; done"
   ```
5. Submit a pull request. PRs must pass automated JSON Schema validation.

---

## 9. License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
