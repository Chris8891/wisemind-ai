# WiseMindAI Skill

[中文版](./README.md)

WiseMindAI is a local-first AI learning and knowledge workspace. It brings documents, knowledge bases, an AI workspace, notes, knowledge cards, AI exams, learning analytics, posters, and infographics into one workflow so material processing, knowledge accumulation, and final output can all happen in one place.

Official Website: [https://wisemindai.app/](https://wisemindai.app/)

An Agent Skill for WiseMindAI.  
It enables Claude Code, Codex, OpenClaw, and other compatible agents to work with the WiseMindAI local API for saving, organizing, searching, and updating knowledge content.

## Installation & Discovery

```bash
# List installable skills in this repository
npx skills add Chris8891/wisemind-ai --list

# Install WiseMindAI Skill globally
npx skills add Chris8891/wisemind-ai@wisemind-ai -g -y
```

Notes:

- `-g` installs the skill globally
- `-y` skips confirmation prompts
- these commands are intended for the published GitHub install flow

You can also tell a compatible agent in plain language:

```text
Help me install this Skill: https://github.com/Chris8891/wisemind-ai
```

If you are only testing locally for now, use the local installation guide below.

## What This Is

This is a WiseMindAI skill package for AI agents.

It helps agents:

- save notes
- save text, file records, links, and full webpages
- save content into specific folders
- write into knowledge bases and knowledge documents
- create card decks, card folders, and learning cards
- search, update, and delete existing items
- use `resolve` and `upsert` workflows to reduce duplicates

## Requirements

Before installing this skill, make sure:

- WiseMindAI is installed and running on the same machine
- the WiseMindAI local API is enabled
- the default endpoint is reachable: `http://127.0.0.1:38221`
- your agent tool can import a local skill or read a local `SKILL.md`
- for full webpage saving, the host agent should ideally also have web-fetching or browser capability

## Local Installation Guide

### Step 1: Prepare the Skill Folder

Make sure the folder looks like this:

```text
skill/
├── README.md
├── README.en.md
└── SKILL.md
```

Where:

- `SKILL.md` is the agent-facing instruction file
- `README.md` is the Chinese documentation
- `README.en.md` is the English documentation

### Step 2: Provide the Folder to Your Agent Tool

Use the whole `skill` directory as a local skill package in your agent tool.

In most tools, this usually means one of these:

1. put it into the tool's local skills directory
2. use an "import local skill" or "add local skill" action in the UI

If your tool supports GitHub-based skills, you can switch to the install commands above after publishing.

### Step 3: Start WiseMindAI and Verify the Local API

Start WiseMindAI and verify the API:

```bash
curl http://127.0.0.1:38221/api/health
```

If the response includes `ok: true`, the local service is available.

If it fails:

- check whether WiseMindAI is running
- check whether the local API is enabled
- if you use a custom port, tell the agent the correct port when using the skill

### Step 4: Run a Simple Verification Prompt

After installation, test it with a simple request:

```text
Save this content to wisemind-ai with the title "Install Test":
This is my first test entry.
```

If the agent saves it successfully, the skill is working.

## Usage Guide

### Use Case 1: Save a Normal Note

You can say:

```text
Save this meeting summary to wisemind-ai with the title "Weekly Sync".
```

Good for:

- meeting notes
- idea capture
- drafts
- quick notes

### Use Case 2: Save a Webpage

You can say:

```text
Save this webpage to the `Read Later` folder in WiseMindAI:
https://example.com/article
```

If the host agent can fetch the page content, the skill will try to save the title, summary, and main body.  
If it cannot fetch the content, it will usually save the link only or ask you to provide the content.

### Use Case 3: Save Content into a Knowledge Base

You can say:

```text
Put this content into the `AI Research` knowledge base in wisemind-ai with the title "RAG Notes".
```

Good for:

- article organization
- long-form knowledge storage
- topic-based research notes
- project reference material

### Use Case 4: Create Study Cards

You can say:

```text
Create 8 study cards from this content and put them in `AI Basics / Retrieval-Augmented Generation`.
```

Good for:

- turning text into cards
- terminology memorization
- review material generation

### Use Case 5: Search and Update

You can say:

```text
Search my wisemind-ai for content about `RAG`.
```

Or:

```text
Update the summary of `RAG Notes` in wisemind-ai.
```

## Suggested Prompts

- Save this content to wisemind-ai with the title "Weekly Sync".
- Save this webpage to the `Read Later` folder in wisemind-ai.
- Search my wisemind-ai for content about `RAG`.
- Put this article into the `AI Research` knowledge base.
- Create 8 study cards from this content in `AI Basics / Retrieval-Augmented Generation`.
- Update the summary of `RAG Notes` in wisemind-ai.
- Delete the test note `Install Test` from wisemind-ai.

## Default Behavior

- check service availability first
- read capabilities when parameters are unclear
- search before writing to reduce duplicates
- resolve folders, knowledge bases, and card containers by name when possible
- read before updating, and only delete when the target is clear

## Repository Structure

The current skill folder looks like this:

```text
skill/
├── README.md
├── README.en.md
└── SKILL.md
```

If you want the `skills` CLI to recognize it more reliably, publish it with an Agent Skills standard directory layout.

## Limitations

- this skill depends on a running local WiseMindAI service
- if the user only provides a URL and the host agent cannot fetch the page content, the agent may only be able to save the link or ask the user for the content
- delete operations should only happen when the user explicitly asks for them

## File Guide

- `SKILL.md`: agent-facing instructions
- `README.md`: Chinese documentation
- `README.en.md`: English documentation
