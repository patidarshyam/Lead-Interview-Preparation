# Copilot Instructions — Technical Interview Prep Workspace

## Answer Format
- All Q&A answers follow: **Definition → Mechanism → Benefits/Trade-offs** as a single flowing paragraph.
- No bullet-heavy answers. Use bullets only for lists of 3+ named items (e.g., ACID properties, DDL/DML categories).
- Theory paragraphs: keep as one-liners or max 2–3 lines. Remove entirely where the answer body already covers the concept.

## File & Folder Naming
- Company prep files: `<company-name>-interview-preparation.md`
- Folder: `05-company-wise-interview-preparation` (not short names like `05-company-interviews`)
- New technical topic files: prefix with two-digit index (e.g., `13-microservices-concepts.md`)

## README.md Structure
- Uses a **phase-based table structure** (Phase 1–7). Any new content file must be linked in the relevant phase table with columns: Step | Topic | File | Q&A count.
- Repo structure section in README must stay in sync with actual folder/file names.

## Image & SVG Standards
- Image embedding format: `[![alt](path)](path)` — renders inline on GitHub, click opens full size for zoom.
- **Prefer PNG/JPG/GIF over SVG** for diagrams where text readability matters.
- All SVG files must have on the root `<svg>` element: `width="100%" height="auto" preserveAspectRatio="xMidYMid meet"` for browser zoom support.
- SVG style standard: dark background `#0f1117`, AWS orange `#FF9900` for titles, color-coded sections, `font-family="Segoe UI, Arial, sans-serif"`.
- When multiple use-case diagrams exist, display them as a markdown table (one row per use case) rather than a single combined image.

## assets/images/README.md
- Must be updated whenever a new image is added.
- Add a row with: filename (as link) | description | linked source file.

## Editing Rules
- **Always read a file before editing** — user may have undone previous edits; never assume current state.
- Do NOT create markdown files to summarize or document changes unless explicitly asked.
- Do not add features, refactor, or improve beyond what is asked.
