---
name: wisemind-ai
description: Use this skill when the user wants to create, save, search, update, organize, summarize, quiz, annotate, review cards, open sources, or delete content in WiseMindAI through the local HTTP API. Supports notes, files, links, webpages, folders, knowledge bases, knowledge documents, cards, tags, annotations, assistant chat/summarize/quiz, card review, workspace/open-source, global search, resolve workflows, and idempotent upsert writes. Trigger on requests like "保存到 WiseMindAI", "记到知识库", "帮我存网页", "创建学习卡片", "总结这段", "出几道题", "搜索我的 WiseMindAI", "打开到客户端", "save to WiseMindAI", or any task involving WiseMindAI content or AI helpers.
metadata:
  author: wangpingan
  version: '2.2.1'
---

# WiseMindAI Skill

Use this skill to operate a local WiseMindAI app over HTTP (Local API v2 / app 2.0).

## Defaults

- Base URL: `http://127.0.0.1:38221`
- First try the default port. Only ask the user for a custom port if the health check fails or the user explicitly mentions another port.
- Prefer `/api/v2/*` over legacy endpoints.
- Send JSON with `Content-Type: application/json`.
- Use `curl` unless the host agent already has a better built-in HTTP tool.
- Authoritative endpoint docs live in the desktop app API dialog and `packages/shared/config/localApi.ts`. This skill is a playbook, not a full OpenAPI dump.

## Required Workflow

1. Check connectivity before doing anything else.
2. Fetch `/api/capabilities` and read `features` + `aiReady` before advanced or AI actions.
3. Choose the correct WiseMindAI object type before writing.
4. Search or read before writing, updating, or deleting.
5. Use `resolve` endpoints for folders, knowledge bases, decks, and card folders.
6. Use `upsert` for URLs or file paths that may be written repeatedly.
7. Never delete unless the user asked clearly for deletion.
8. Report the result briefly with what was saved or changed and where it went.

## Standard Source Fields

When creating or updating knowledge documents, always preserve the source metadata:

| Source kind | `type` | `fileType` | `fileExt` | `fileUrl` |
| --- | --- | --- | --- | --- |
| Plain text typed by the user | `input` | `input` | `input` | empty string |
| Web URL, bookmark, or webpage article | `link` | `link` | `link` | original URL |
| Local/imported file | `file` or `upload` | discovered file type | actual extension | file path or Wise file URL |

For webpage or URL content saved into a knowledge base, `fileUrl` is required and must be the original URL. Do not save URL-derived knowledge documents with empty `fileUrl`, `fileType`, or `fileExt`; the app uses these fields to display the document as a link.

## Common Payload Guardrails

- Notes use `title` plus `content`. When saving Markdown/plain text notes, send the same body to `content`, `md`, and `text` unless you intentionally need different representations. `tags` must be an array.
- File records use `name` for the visible title and `filePath` for the URL or local path. For links, always send `type:"link"` and `fileType:"link"`. For typed text records, send `type:"input"` and `fileType:"input"`.
- Webpages saved to the document area should use `/api/v2/files/save-webpage` with `url`, `title`, `summary`, and `content`.
- Knowledge documents use `title` for display and must include `knowledgeBaseId`. For source-aware records, include all of `type`, `fileType`, `fileExt`, and `fileUrl`; do not rely on defaults.
- Cards require numeric `deckId`, numeric `folderId`, and non-empty `content`. For batch creation, each item in `cards` must contain `content`; `tags` should be an array when provided.
- Patch requests should include only fields that are intentionally changing. Do not send blank source fields in a patch unless the user asked to clear them.

## Health Check

Run this first:

```bash
curl -s http://127.0.0.1:38221/api/health
```

Expected shape:

```json
{"ok": true, "service": "wisemindai-local-api", "version": "2.0.0"}
```

If this fails:

- Ask the user to start WiseMindAI and enable the local API.
- If they use a custom port, retry with that port.
- Do not continue with write operations until the service is reachable.

## Capabilities Discovery

Always fetch this before assistant, annotation, quiz, todo, subscription, or tags-list workflows:

```bash
curl -s http://127.0.0.1:38221/api/capabilities
```

This endpoint exposes:

- `features` — capability gates used by plugins and this skill
- `aiReady` / `aiModel` — whether assistant endpoints can run
- supported groups and endpoint list
- `fileTypeOptions` / `fileCategoryOptions` / `knowledgeTypeOptions` / `knowledgeStatusOptions`

### Feature gates (check before calling)

| Feature | Typical endpoints |
| --- | --- |
| `tags` | `GET /api/v2/tags` |
| `annotations` | `/api/v2/annotations` |
| `screenshot` | `POST /api/v2/screenshots` |
| `quiz` | `POST /api/v2/assistant/quiz` |
| `task-plan` | `POST /api/v2/assistant/tasks/plan` |
| `todos` | `POST /api/v2/todos` |
| `subscriptions` | `POST /api/v2/subscriptions` |
| `chat-history` | `/api/v2/assistant/sessions` |
| `agent-handoff` | `POST /api/v2/open-source` with `type:"agent"` |
| `workspace` | `GET /api/v2/workspace/state` |

If a feature is missing, tell the user to update/open WiseMindAI 2.0 and enable Local API. If `aiReady` is false, do not call assistant endpoints; ask the user to connect a model first.

Prefer runtime discovery instead of hard-coding optional values when uncertain.

## Pick The Right Destination

| User intent                                                             | Preferred target      |
| ----------------------------------------------------------------------- | --------------------- |
| Quick thoughts, meeting notes, drafts, journals, plain note-taking      | `notes`               |
| Save a local file, URL, bookmark, webpage, or free-form document record | `files`               |
| Put content into a named knowledge base for long-term organization      | `knowledge-documents` |
| Create learning cards or spaced-repetition material                     | `cards`               |

Default choices:

- Plain text with no stronger structure requirement: save as a note.
- A URL or external resource: save as a file record.
- Content explicitly meant for a knowledge base: save as a knowledge document.
- Quiz, memorization, flashcard, or card request: create cards.

Chinese routing guardrails:

- Treat `保存到 wisemind-ai 文档中`, `保存到 wisemind-ai 文档中心`, `保存到文档`, `保存到文件`, and `存为链接` as the document center. Use `files`, preferably `/api/v2/files/save-webpage` or `/api/v2/files/upsert`.
- Do not interpret the Chinese word `文档` by itself as `knowledge-documents`. In this skill, `knowledge-documents` is only for an explicitly named knowledge base.
- Use `knowledge-documents` only when the user clearly says `知识库`, `知识库文档`, `放进某个知识库`, or provides a knowledge base name as the destination.
- Do not create or resolve a knowledge base such as `Skill 资料` unless the user explicitly asked for that knowledge base.
- When the destination is the document center, report it as `文档中心`, not as `知识库`.

## Core Endpoints

### Common

- `GET /api/health`
- `GET /api/capabilities`
- `GET /api/search`
- `GET /api/v2/workspace/state`
- `GET /api/v2/account/usage`
- `GET /api/v2/tags`
- `POST /api/v2/open-source`

### Notes

- `GET /api/v2/notes`
- `GET /api/v2/notes/:id`
- `POST /api/v2/notes`
- `PATCH /api/v2/notes/:id`
- `DELETE /api/v2/notes/:id`
- `GET /api/v2/note-folders`
- `POST /api/v2/note-folders`
- `POST /api/v2/note-folders/resolve`

### Files, Links, and Webpages

- `GET /api/v2/files`
- `GET /api/v2/files/:id`
- `POST /api/v2/files`
- `PATCH /api/v2/files/:id`
- `DELETE /api/v2/files/:id`
- `POST /api/v2/files/upsert`
- `POST /api/v2/files/save-webpage`
- `GET /api/v2/file-folders`
- `POST /api/v2/file-folders`
- `POST /api/v2/file-folders/resolve`

### Knowledge Bases

- `GET /api/v2/knowledge-bases`
- `POST /api/v2/knowledge-bases`
- `PATCH /api/v2/knowledge-bases/:id`
- `POST /api/v2/knowledge-bases/resolve`
- `GET /api/v2/knowledge-documents`
- `GET /api/v2/knowledge-documents/:id`
- `POST /api/v2/knowledge-documents`
- `PATCH /api/v2/knowledge-documents/:id`
- `DELETE /api/v2/knowledge-documents/:id`

### Cards

- `GET /api/v2/card-decks`
- `POST /api/v2/card-decks`
- `PATCH /api/v2/card-decks/:id`
- `POST /api/v2/card-decks/resolve`
- `GET /api/v2/card-folders`
- `POST /api/v2/card-folders`
- `PATCH /api/v2/card-folders/:id`
- `POST /api/v2/card-folders/resolve`
- `GET /api/v2/cards`
- `GET /api/v2/cards/:id`
- `POST /api/v2/cards`
- `PATCH /api/v2/cards/:id`
- `DELETE /api/v2/cards/:id`
- `POST /api/v2/cards/batch`
- `GET /api/v2/card-review/due`
- `POST /api/v2/card-review/sessions`

### Assistant (requires `aiReady`)

- `POST /api/v2/assistant/chat`
- `POST /api/v2/assistant/summarize`
- `POST /api/v2/assistant/generate-cards`
- `POST /api/v2/assistant/rewrite`
- `POST /api/v2/assistant/quiz`
- `POST /api/v2/assistant/tasks/plan`
- `GET /api/v2/assistant/sessions`
- `GET /api/v2/assistant/sessions/:id`

### Annotations / todos (feature-gated)

- `GET|POST /api/v2/annotations` · `PATCH|DELETE /api/v2/annotations/:id`
- `POST /api/v2/todos`
- `POST /api/v2/subscriptions`
- `POST /api/v2/screenshots`

Streaming (`*/stream`) and transcription WebSocket are available on Local API HTTP but are optional for this skill; prefer non-stream endpoints unless the host can consume NDJSON.

If the host is connected to WiseMindAI MCP (`http://127.0.0.1:38222`) instead of raw HTTP, prefer Resource `wisemind://capabilities` / tool `get_local_api_capabilities` first, then the same playbooks via `api_*` tools. Do not expect stream tools on MCP.

## Operating Rules

### Search Before Write

Before creating new content, prefer:

```bash
curl -s "http://127.0.0.1:38221/api/search?q=keyword&limit=10"
```

Use this to avoid duplicate notes, documents, knowledge documents, or cards.

### Read Before Update

For updates:

1. List or search candidates.
2. Read the current item if needed.
3. Patch only the fields that should change.

### Resolve Containers Instead of Guessing IDs

When the user names a destination but does not provide IDs, use resolve endpoints first:

- `/api/v2/note-folders/resolve`
- `/api/v2/file-folders/resolve`
- `/api/v2/knowledge-bases/resolve`
- `/api/v2/card-decks/resolve`
- `/api/v2/card-folders/resolve`

This is the default behavior for human-friendly names.

### Prefer Idempotent Writes For URLs And Paths

If the same URL or file path may be saved more than once, prefer:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/files/upsert" \
  -H "Content-Type: application/json" \
  -d '{"name":"Example","filePath":"https://example.com","type":"link","fileType":"link"}'
```

Use `save-webpage` when you have actual webpage content, and use `upsert` when the key requirement is deduplication by URL or path.

## Common Playbooks

### Save A Note

Create a note:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/notes" \
  -H "Content-Type: application/json" \
  -d '{"title":"Note title","content":"Body text","md":"Body text","text":"Body text","tags":["tag1","tag2"]}'
```

If the user gave a folder name, resolve the folder first and pass `from_folder`.

### Save Free-Form Text Or A Local File Record

For text that should live as a document record instead of a note:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/files" \
  -H "Content-Type: application/json" \
  -d '{"name":"Document title","type":"input","fileType":"input","filePath":"","content":"Text content","summary":"Short summary"}'
```

For URLs, use link metadata and keep the original URL in `filePath`:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/files/upsert" \
  -H "Content-Type: application/json" \
  -d '{"name":"Article title","filePath":"https://example.com/article","type":"link","fileType":"link","summary":"Short summary","content":"Optional captured content"}'
```

If you only know the URL and do not have a real page title yet, use the URL as `name`; never use a folder name, knowledge base name, or generic destination label such as `Skill 资料` as the visible link name.

For local files, choose the appropriate `type` extension and `fileType` category. If uncertain, fetch `/api/capabilities`.

### Save A Webpage

If the user provides only a URL:

1. Use the host agent's available web, browser, or fetch capability to get the page title and main content.
2. If full content is available, save it with `save-webpage`.
3. If full content is not available, either save a link record with `upsert` or ask the user to provide the content they want stored.
4. If the user explicitly said to save the URL into a knowledge base, use `/api/v2/knowledge-documents` instead and follow the link-shaped knowledge document payload.

For requests like `https://example.com 保存到 wisemind-ai 文档中`, the correct target is the document center:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/files/upsert" \
  -H "Content-Type: application/json" \
  -d '{"name":"https://example.com","filePath":"https://example.com","type":"link","fileType":"link"}'
```

After a successful document-center save, report in this shape:

```markdown
✅ **已保存！**

- 位置：文档中心
- 文档 ID：123
- filePath: https://example.com
- type、fileType 都是 "link"
```

Save a webpage with content:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/files/save-webpage" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/article","title":"Article title","summary":"Short summary","content":"Main article content"}'
```

Summary guidance:

- Match the user's language unless they ask otherwise.
- Keep the summary compact and useful.
- Prefer one short summary paragraph plus a few key points.

### Import Content Into A Knowledge Base

Resolve the knowledge base first:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/knowledge-bases/resolve" \
  -H "Content-Type: application/json" \
  -d '{"name":"AI Research"}'
```

Then create the knowledge document:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/knowledge-documents" \
  -H "Content-Type: application/json" \
  -d '{"knowledgeBaseId":1,"title":"RAG Notes","content":"Document body","summary":"Short summary","type":"input","fileType":"input","fileExt":"input","fileUrl":""}'
```

If the source is a URL or webpage, keep the original URL in `fileUrl` and mark all source type fields as `link`:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/knowledge-documents" \
  -H "Content-Type: application/json" \
  -d '{"knowledgeBaseId":1,"title":"Article title","content":"Main article content","summary":"Short summary","type":"link","fileType":"link","fileExt":"link","fileUrl":"https://example.com/article"}'
```

For requests like "put this article URL into knowledge base X", fetch the title/content when possible, then create the knowledge document with the link payload above. If content cannot be fetched, still preserve the URL fields and save a compact link record with `content` and `summary` set to the available title, description, or a short note.

If the user asked to update an existing knowledge document, search or list first, then patch the matched item.

### Create Cards

Resolve the deck, then the folder, then create cards.

Single card:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/cards" \
  -H "Content-Type: application/json" \
  -d '{"deckId":1,"folderId":1,"content":"Card content"}'
```

Batch cards:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/cards/batch" \
  -H "Content-Type: application/json" \
  -d '{"deckId":1,"folderId":1,"cards":[{"content":"Card 1","tags":["tag1"]},{"content":"Card 2","tags":["tag2"]}]}'
```

Card-writing guidance:

- Keep each card focused on one idea.
- Use batch creation when the user clearly wants multiple cards.
- Do not generate a large batch blindly if the user asked for only a few cards.

### Update Existing Content

When the user says "update", "move", "rename", "change summary", or similar:

1. Find the target by search, list, or explicit ID.
2. If multiple plausible matches exist, ask a short clarifying question.
3. Use `PATCH` on the correct endpoint.
4. Tell the user exactly what changed.

### Delete Existing Content

Deletion is destructive. Only proceed when:

- the user explicitly asked for deletion, and
- the target is unambiguous.

If the target is ambiguous, ask before deleting anything.

### Summarize Text Or A Webpage

1. Confirm `aiReady` from capabilities.
2. Prefer saving the source first (`save-webpage` / `upsert`) when the user asked to keep it.
3. Call summarize:

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/assistant/summarize" \
  -H "Content-Type: application/json" \
  -d '{"content":"正文","sourceTitle":"标题","style":"brief","language":"zh_CN"}'
```

4. Optionally save the markdown result as a note, or open the source with `/api/v2/open-source`.

### Generate Cards Or A Quiz

Cards draft (then batch-save):

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/assistant/generate-cards" \
  -H "Content-Type: application/json" \
  -d '{"content":"正文","count":5,"difficulty":"standard","structure":"qa"}'
```

Quiz (requires feature `quiz`):

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/assistant/quiz" \
  -H "Content-Type: application/json" \
  -d '{"content":"正文","questionCount":5,"questionTypes":["single","judge"],"difficulty":"medium"}'
```

Then open with `{"type":"quiz","id":<paperId>}` via `/api/v2/open-source` when useful.

### List Due Review Cards

Requires card data in the app:

```bash
curl -s "http://127.0.0.1:38221/api/v2/card-review/due?limit=20"
```

### Create An Annotation

Requires feature `annotations`. Prefer listing first by `url` or `fileId`.

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/annotations" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/page","quote":"划词原文","note":"备注","color":"yellow"}'
```

### Open In The Desktop App

```bash
curl -s -X POST "http://127.0.0.1:38221/api/v2/open-source" \
  -H "Content-Type: application/json" \
  -d '{"type":"file","id":42}'
```

For “让 AI 去做”, use `{"type":"agent","prompt":"..."}` when feature `agent-handoff` is present.

### List Tags / Workspace

```bash
curl -s "http://127.0.0.1:38221/api/v2/tags"
curl -s "http://127.0.0.1:38221/api/v2/workspace/state"
```

## Result Reporting

After a successful operation, report in one short message:

- what was created, updated, or deleted
- where it was saved
- key identifiers only when useful

Good examples:

- "Saved the article to WiseMindAI as a webpage record in `Reading Queue`."
- "Created 8 cards in deck `AI Basics` / folder `RAG`."
- "Updated the summary of the existing knowledge document in `Product Notes`."

## Legacy Endpoints

Legacy endpoints still exist for compatibility, but new work should use `/api/v2/*`.
