---
name: wisemindai
description:
  WiseMindAI 本地 API 集成，支持笔记/文档/知识库/卡片的完整 CRUD、全局搜索、去重写入、批量创建等操作。
  触发场景：用户要求向 WiseMindAI 添加笔记、保存网页链接、导入文档到知识库、创建学习卡片，或任何涉及 WiseMindAI 应用的操作。
metadata:
  author: wangpingan
  version: "2.0.0"
---

# WiseMindAI Skill

WiseMindAI 本地 API 集成工具，通过 HTTP API 与本地运行的 WiseMindAI 应用交互。

## 前置确认

使用前需确认 WiseMindAI 服务端口号（默认 `38221`）。如果用户未指定端口，先询问端口号再继续。

## 基础配置

- 默认服务地址：`http://127.0.0.1:38221`
- 所有接口均为 JSON 格式，请求头需设置 `Content-Type: application/json`
- 调用方式：通过 `curl` 执行请求

## 连通性检查

每次使用前先检测服务是否可用：

```bash
curl -s http://127.0.0.1:PORT/api/health
```

返回 `{"ok": true, ...}` 表示服务正常；其他状态码或超时则提示用户确认 WiseMindAI 是否已启动。

## API 接口总览

### 通用能力

| 能力 | 方法 | 路径 |
|------|------|------|
| 健康检查 | GET | `/api/health` |
| 获取服务能力清单 | GET | `/api/capabilities` |
| 全局搜索 | GET | `/api/search` |

### 笔记

| 能力 | 方法 | 路径 |
|------|------|------|
| 获取笔记列表 | GET | `/api/v2/notes` |
| 获取笔记详情 | GET | `/api/v2/notes/:id` |
| 新建笔记 | POST | `/api/v2/notes` |
| 更新笔记 | PATCH | `/api/v2/notes/:id` |
| 删除笔记 | DELETE | `/api/v2/notes/:id` |
| 获取笔记文件夹 | GET | `/api/v2/note-folders` |
| 新建笔记文件夹 | POST | `/api/v2/note-folders` |
| 按名称获取或创建笔记文件夹 | POST | `/api/v2/note-folders/resolve` |

### 文档与链接

| 能力 | 方法 | 路径 |
|------|------|------|
| 获取文档列表 | GET | `/api/v2/files` |
| 获取文档详情 | GET | `/api/v2/files/:id` |
| 新建文档/链接记录 | POST | `/api/v2/files` |
| 更新文档记录 | PATCH | `/api/v2/files/:id` |
| 删除文档记录 | DELETE | `/api/v2/files/:id` |
| 按路径去重写入文档 | POST | `/api/v2/files/upsert` |
| 保存网页内容 | POST | `/api/v2/files/save-webpage` |
| 获取文档文件夹 | GET | `/api/v2/file-folders` |
| 新建文档文件夹 | POST | `/api/v2/file-folders` |
| 按名称获取或创建文档文件夹 | POST | `/api/v2/file-folders/resolve` |

### 知识库

| 能力 | 方法 | 路径 |
|------|------|------|
| 获取知识库列表 | GET | `/api/v2/knowledge-bases` |
| 新建知识库 | POST | `/api/v2/knowledge-bases` |
| 更新知识库 | PATCH | `/api/v2/knowledge-bases/:id` |
| 按名称获取或创建知识库 | POST | `/api/v2/knowledge-bases/resolve` |
| 获取知识文档列表 | GET | `/api/v2/knowledge-documents` |
| 获取知识文档详情 | GET | `/api/v2/knowledge-documents/:id` |
| 新建知识文档 | POST | `/api/v2/knowledge-documents` |
| 更新知识文档 | PATCH | `/api/v2/knowledge-documents/:id` |
| 删除知识文档 | DELETE | `/api/v2/knowledge-documents/:id` |

### 卡片

| 能力 | 方法 | 路径 |
|------|------|------|
| 获取卡片集列表 | GET | `/api/v2/card-decks` |
| 新建卡片集 | POST | `/api/v2/card-decks` |
| 更新卡片集 | PATCH | `/api/v2/card-decks/:id` |
| 按名称获取或创建卡片集 | POST | `/api/v2/card-decks/resolve` |
| 获取卡片目录列表 | GET | `/api/v2/card-folders` |
| 新建卡片目录 | POST | `/api/v2/card-folders` |
| 更新卡片目录 | PATCH | `/api/v2/card-folders/:id` |
| 按名称获取或创建卡片目录 | POST | `/api/v2/card-folders/resolve` |
| 获取知识卡片列表 | GET | `/api/v2/cards` |
| 获取知识卡片详情 | GET | `/api/v2/cards/:id` |
| 新建知识卡片 | POST | `/api/v2/cards` |
| 更新知识卡片 | PATCH | `/api/v2/cards/:id` |
| 删除知识卡片 | DELETE | `/api/v2/cards/:id` |
| 批量创建知识卡片 | POST | `/api/v2/cards/batch` |

## 接口详情

---

### 通用能力

#### 健康检查

```bash
curl -s http://127.0.0.1:PORT/api/health
```

**响应：** `{"ok": true, "service": "wisemindai-local-api", "version": "2.0.0"}`

#### 获取服务能力清单

返回当前支持的接口分组、文件类型等可选值，适合动态了解能力边界。

```bash
curl -s http://127.0.0.1:PORT/api/capabilities
```

**响应：** `{"ok": true, "data": {"groups": [...], "fileTypeOptions": [...], "fileCategoryOptions": [...]}}`

#### 全局搜索

按关键词在笔记、文档、知识文档和卡片中统一搜索。

```bash
curl -s "http://127.0.0.1:PORT/api/search?q=关键词&types=notes,files&limit=10"
```

| 参数 | 位置 | 必填 | 说明 |
|------|------|------|------|
| q | query | 是 | 搜索关键词 |
| types | query | 否 | 范围：`notes`, `files`, `knowledge-documents`, `cards`，逗号分隔 |
| limit | query | 否 | 每类最大返回数，默认 10 |

---

### 笔记

#### 获取笔记列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/notes?q=关键词&from_folder=1&includeFolders=false&limit=50"
```

#### 获取笔记详情

```bash
curl -s "http://127.0.0.1:PORT/api/v2/notes/1"
```

#### 新建笔记

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/notes" \
  -H "Content-Type: application/json" \
  -d '{"title": "笔记标题", "content": "正文", "tags": ["标签"]}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| title | 是 | 笔记标题 |
| content | 否 | 富文本或普通正文 |
| md | 否 | Markdown 内容 |
| text | 否 | 纯文本内容 |
| tags | 否 | 标签数组 |
| from_folder | 否 | 目标文件夹 ID |

#### 更新笔记

```bash
curl -s -X PATCH "http://127.0.0.1:PORT/api/v2/notes/1" \
  -H "Content-Type: application/json" \
  -d '{"title": "新标题", "content": "新内容"}'
```

参数同新建笔记，所有字段可选。

#### 删除笔记

```bash
curl -s -X DELETE "http://127.0.0.1:PORT/api/v2/notes/1"
```

#### 获取笔记文件夹列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/note-folders"
```

#### 新建笔记文件夹

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/note-folders" \
  -H "Content-Type: application/json" \
  -d '{"name": "文件夹名"}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| name | 是 | 文件夹名称 |
| from_folder | 否 | 父级文件夹 ID |

#### 按名称获取或创建笔记文件夹

按名称查找，不存在则自动创建，减少先查 ID 的步骤。

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/note-folders/resolve" \
  -H "Content-Type: application/json" \
  -d '{"name": "文件夹名"}'
```

**响应：** `{"ok": true, "data": {"id": 12, "name": "...", "created": false}}`

---

### 文档与链接

#### 获取文档列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/files?q=关键词&from_folder=1&includeFolders=false&limit=50"
```

#### 获取文档详情

```bash
curl -s "http://127.0.0.1:PORT/api/v2/files/1"
```

#### 新建文档或链接记录

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/files" \
  -H "Content-Type: application/json" \
  -d '{"name": "文件名", "filePath": "/path/or/url", "type": "link", "fileType": "link", "summary": "摘要"}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| name | 是 | 文件名称 |
| filePath | 否 | 本地路径、URL 或资源位置 |
| size | 否 | 文件大小（字节） |
| type | 否 | 扩展名/来源标记：mp3, wav, ogg, flac, m4a, mpga, weba, eml, xls, xlsx, jpg, jpeg, png, gif, bmp, tiff, svg, webp, link, md, markdown, pdf, pptx, txt, input, srt, ass, sub, ssa, vtt, mp4, avi, mov, wmv, mkv, flv, mpg, qt, webm, docx, rss, xmind, epub |
| fileType | 否 | 文件大类：audio, email, excel, image, link, md, pdf, ppt, text, input, tts, video, word, rss, xmind, epub |
| content | 否 | 正文内容 |
| note | 否 | 备注 |
| summary | 否 | 摘要 |
| translation | 否 | 翻译内容 |
| icon | 否 | 图标 URL |
| from_folder | 否 | 目标文件夹 ID |

#### 更新文档记录

```bash
curl -s -X PATCH "http://127.0.0.1:PORT/api/v2/files/1" \
  -H "Content-Type: application/json" \
  -d '{"summary": "新摘要", "note": "新备注"}'
```

可更新字段：name, content, summary, translation, note, from_folder。

#### 删除文档记录

```bash
curl -s -X DELETE "http://127.0.0.1:PORT/api/v2/files/1"
```

#### 按路径去重写入文档

按 filePath 查找，存在则更新，不存在则新建，幂等写入。

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/files/upsert" \
  -H "Content-Type: application/json" \
  -d '{"name": "网页标题", "filePath": "https://example.com", "type": "link", "fileType": "link", "summary": "摘要"}'
```

参数同新建文档，但 filePath 必填。**响应：** `{"ok": true, "data": {"id": 9, "created": false}}`

#### 保存网页内容

一步保存网页链接、标题、摘要和正文，无需拆成多步。

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/files/save-webpage" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/article", "title": "文章标题", "summary": "摘要", "content": "正文内容"}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| url | 是 | 网页链接 |
| title | 否 | 网页标题，不传时使用 URL |
| summary | 否 | 摘要 |
| content | 否 | 正文内容 |
| icon | 否 | 图标 URL |
| from_folder | 否 | 目标文件夹 ID |

#### 获取文档文件夹列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/file-folders"
```

#### 新建文档文件夹

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/file-folders" \
  -H "Content-Type: application/json" \
  -d '{"name": "文件夹名"}'
```

#### 按名称获取或创建文档文件夹

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/file-folders/resolve" \
  -H "Content-Type: application/json" \
  -d '{"name": "文件夹名"}'
```

**响应：** `{"ok": true, "data": {"id": 5, "name": "...", "created": true}}`

---

### 知识库

#### 获取知识库列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/knowledge-bases?q=关键词"
```

#### 新建知识库

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/knowledge-bases" \
  -H "Content-Type: application/json" \
  -d '{"name": "知识库名称", "icon": "brain", "desc": "描述"}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| name | 是 | 知识库名称 |
| icon | 否 | 图标 |
| desc | 否 | 描述 |

#### 更新知识库

```bash
curl -s -X PATCH "http://127.0.0.1:PORT/api/v2/knowledge-bases/1" \
  -H "Content-Type: application/json" \
  -d '{"name": "新名称"}'
```

可更新字段：name, icon, desc。

#### 按名称获取或创建知识库

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/knowledge-bases/resolve" \
  -H "Content-Type: application/json" \
  -d '{"name": "知识库名称"}'
```

**响应：** `{"ok": true, "data": {"id": 3, "name": "...", "created": false}}`

#### 获取知识文档列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/knowledge-documents?knowledgeBaseId=1&q=关键词&limit=50"
```

#### 获取知识文档详情

```bash
curl -s "http://127.0.0.1:PORT/api/v2/knowledge-documents/1"
```

#### 新建知识文档

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/knowledge-documents" \
  -H "Content-Type: application/json" \
  -d '{"knowledgeBaseId": 1, "title": "文档标题", "content": "正文", "summary": "摘要", "type": "input"}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| knowledgeBaseId | 是 | 知识库 ID |
| title | 是 | 文档标题 |
| content | 否 | 文档正文 |
| summary | 否 | 文档摘要 |
| fileUrl | 否 | 资源 URL 或文件位置 |
| fileType | 否 | 文件大类 |
| fileExt | 否 | 文件扩展名 |
| type | 否 | 来源类型：file, note, upload, link, input |
| sourceId | 否 | 来源记录 ID |

#### 更新知识文档

```bash
curl -s -X PATCH "http://127.0.0.1:PORT/api/v2/knowledge-documents/1" \
  -H "Content-Type: application/json" \
  -d '{"summary": "新摘要"}'
```

可更新字段：title, content, summary, fileUrl。

#### 删除知识文档

```bash
curl -s -X DELETE "http://127.0.0.1:PORT/api/v2/knowledge-documents/1"
```

---

### 卡片

#### 获取卡片集列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/card-decks"
```

**响应包含嵌套的 folders 数组。**

#### 新建卡片集

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/card-decks" \
  -H "Content-Type: application/json" \
  -d '{"title": "卡片集标题", "description": "描述"}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| title | 是 | 卡片集标题 |
| coverUrl | 否 | 封面图片地址 |
| description | 否 | 卡片集描述 |

#### 更新卡片集

```bash
curl -s -X PATCH "http://127.0.0.1:PORT/api/v2/card-decks/1" \
  -H "Content-Type: application/json" \
  -d '{"title": "新标题"}'
```

#### 按名称获取或创建卡片集

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/card-decks/resolve" \
  -H "Content-Type: application/json" \
  -d '{"title": "卡片集标题"}'
```

**响应：** `{"ok": true, "data": {"id": 1, "title": "...", "created": false}}`

#### 获取卡片目录列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/card-folders?deckId=1"
```

#### 新建卡片目录

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/card-folders" \
  -H "Content-Type: application/json" \
  -d '{"deckId": 1, "title": "目录标题", "icon": "lightbulb"}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| deckId | 是 | 所属卡片集 ID |
| title | 是 | 目录标题 |
| icon | 否 | 目录图标 |
| sortOrder | 否 | 排序值 |

#### 更新卡片目录

```bash
curl -s -X PATCH "http://127.0.0.1:PORT/api/v2/card-folders/1" \
  -H "Content-Type: application/json" \
  -d '{"title": "新标题"}'
```

#### 按名称获取或创建卡片目录

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/card-folders/resolve" \
  -H "Content-Type: application/json" \
  -d '{"deckId": 1, "title": "目录标题"}'
```

**响应：** `{"ok": true, "data": {"id": 1, "deckId": 1, "title": "...", "created": false}}`

#### 获取知识卡片列表

```bash
curl -s "http://127.0.0.1:PORT/api/v2/cards?deckId=1&folderId=1&q=关键词&limit=50"
```

#### 获取知识卡片详情

```bash
curl -s "http://127.0.0.1:PORT/api/v2/cards/1"
```

#### 新建知识卡片

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/cards" \
  -H "Content-Type: application/json" \
  -d '{"deckId": 1, "folderId": 1, "content": "卡片内容", "style": {"backgroundColor": "yellow"}}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| deckId | 是 | 所属卡片集 ID |
| folderId | 是 | 所属目录 ID |
| content | 是 | 卡片内容 |
| style | 否 | 样式配置对象 |

#### 更新知识卡片

```bash
curl -s -X PATCH "http://127.0.0.1:PORT/api/v2/cards/1" \
  -H "Content-Type: application/json" \
  -d '{"content": "新内容"}'
```

可更新字段：content, style, folderId。

#### 删除知识卡片

```bash
curl -s -X DELETE "http://127.0.0.1:PORT/api/v2/cards/1"
```

#### 批量创建知识卡片

一次提交多张卡片，适合长文拆卡后批量写入。

```bash
curl -s -X POST "http://127.0.0.1:PORT/api/v2/cards/batch" \
  -H "Content-Type: application/json" \
  -d '{"deckId": 1, "folderId": 1, "cards": [{"content": "卡片1"}, {"content": "卡片2", "style": {"backgroundColor": "yellow"}}]}'
```

| 参数 | 必填 | 说明 |
|------|------|------|
| deckId | 是 | 所属卡片集 ID |
| folderId | 是 | 所属目录 ID |
| cards | 是 | 卡片数组，每项至少包含 content，可选 style |

**响应：** `{"ok": true, "data": {"createdIds": [11, 12]}, "message": "Cards created successfully"}`

---

## 网页内容获取策略

当用户提供文章链接或网页 URL 时，需要先获取网页的标题、摘要和正文内容，再保存到 WiseMindAI。

### 使用 web-access Skill 获取网页内容

**必须加载 web-access skill 并遵循其指引**来获取网页内容。web-access 能处理需要登录态、动态渲染、反爬限制等场景，确保获取到正确的文章内容。

**获取流程：**

1. 判断 URL 类型，选择合适的获取方式（参照 web-access 的联网工具选择策略）
2. 通过 web-access 获取网页的完整内容
3. 从获取结果中提取：标题（title）、正文（content）、摘要（summary，如无则由 AI 生成）
4. 调用 WiseMindAI API 保存

**适用场景：**

| 用户输入 | 处理方式 |
|----------|----------|
| 普通文章/博客链接 | web-access 获取内容 → save-webpage 保存 |
| 微信公众号文章 | web-access CDP 模式获取 → save-webpage 保存 |
| 小红书/微博等动态内容 | web-access CDP 模式获取 → save-webpage 保存 |
| 需要登录才能查看的内容 | web-access CDP 模式（携带登录态）→ save-webpage 保存 |
| 用户已提供标题和正文 | 直接调用 save-webpage，无需再获取 |

## 操作指引

### 保存网页链接

当用户提供网址要求保存时：

1. **获取内容**：使用 web-access skill 获取网页标题和正文
2. **AI 生成摘要**：使用下方的摘要生成提示词，对获取到的内容生成结构化摘要
3. **保存到 WiseMindAI**：调用 `/api/v2/files/save-webpage`，将获取到的标题、AI 生成的摘要、正文和原始 URL 一步保存
4. **去重场景**：如需避免重复保存，改用 `/api/v2/files/upsert`，按 URL 去重写入

#### 摘要生成提示词

当需要为文章内容生成摘要时，**必须使用以下提示词**：

```
您是一位专注于内容总结的专家，请使用中文总结我提供的文本，总结内容条理清晰，能帮助读者快速、清晰地学习文章内容。请注意：
    1.精确理解文本并识别出主要信息点。将这些信息有效地组织起来，保证内容结构化。
    2.严格使用Markdown格式，按照模板格式，尽量精简内容，去除任何冗余信息。
    3.结合核心内容与关键词，最多为其生成4个简洁、相关度高的标签（tag），标签需是该知识点直接相关的主题词、概念词或领域词，使用中文语言。
    4.您的总结应该严格遵循以下模板格式，其中中括号[]中的内容是注意项，不需要保留[]：
    ### 🔍摘要
     [一句话总结所有内容]
    ### 🌈总结
     [一段话提取所有内容的摘要]
    ### ✨要点
     [这里填充主要总结点]
    ### 🏷标签
     [这里填充标签内容]

    - 请确保所有返回的内容都严格按照上述标题层级进行组织，markdown 内容中禁止包含任何代码，也不能超出这一格式要求的文本。
    - 只进行内容总结相关的任务，拒绝回答与文章总结无关的话题。
    - 所输出的内容必须使用 markdown 格式，且要点明确，条理清晰。
```

将生成的摘要作为 `summary` 字段传入保存接口。

### 保存文档内容

当用户提供文本要求保存时：

1. 调用 `/api/v2/files`，`type` 为 `input`，`fileType` 为 `input`
2. 标题填入 `name`，内容填入 `content`

### 导入知识库

当用户要求将内容导入知识库时：

1. 用 `/api/v2/knowledge-bases/resolve` 按名称获取或创建知识库（无需先查 ID）
2. 或用 `/api/v2/knowledge-bases` 列出让用户选择
3. 调用 `/api/v2/knowledge-documents` 写入文档

### 创建卡片

当用户要求创建学习卡片时：

1. 用 `/api/v2/card-decks/resolve` 按名称获取或创建卡片集
2. 用 `/api/v2/card-folders/resolve` 按名称获取或创建目录
3. 单张卡片：调用 `/api/v2/cards`
4. 多张卡片：调用 `/api/v2/cards/batch` 批量创建

### 保存到指定文件夹

当用户要求保存到特定文件夹时：

1. 用 `/api/v2/note-folders/resolve` 或 `/api/v2/file-folders/resolve` 按名称获取或创建
2. 或列出文件夹让用户选择
3. 在请求中加入 `from_folder` 参数

### 先查再写

在写入内容前，可通过以下方式避免重复：

1. **全局搜索**：`/api/search?q=关键词` 搜索全部内容
2. **去重写入**：文档用 `/api/v2/files/upsert` 按 filePath 去重
3. **按名称 resolve**：文件夹、知识库、卡片集、卡片目录均支持 resolve 接口

### 先读后改

对于更新操作，遵循"先读后改"工作流：

1. 用 GET 接口读取当前内容
2. 确认需要修改的字段
3. 用 PATCH 接口更新

## 响应处理

所有接口返回 JSON 格式：

- 成功（写入）：`{"ok": true, "id": <number>, "message": "..."}`
- 成功（resolve）：`{"ok": true, "data": {"id": <number>, "name": "...", "created": <boolean>}}`
- 成功（更新/删除）：`{"ok": true, "message": "..."}`
- 成功（批量）：`{"ok": true, "data": {"createdIds": [...]}, "message": "..."}`
- 失败：检查返回的错误信息并反馈给用户

操作完成后，向用户简洁报告结果。
