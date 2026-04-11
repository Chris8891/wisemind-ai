# WiseMindAI Skill

[English Version](./README.en.md)

WiseMindAI 是一款本地优先的 AI 学习与知识工作台。将文档中心、知识库、AI 工作台、笔记中心、知识卡片、AI 考试、学习统计、海报与信息图等模块放进同一套工作流里，让资料处理、知识沉淀和成果输出都在一个地方完成。

官网：[https://wisemindai.app/](https://wisemindai.app/)

一个面向 AI Agent 的 WiseMindAI 技能。  
它让 Claude Code、Codex、OpenClaw 等支持 Skill 的工具，可以直接调用 WiseMindAI 本地 API，完成保存、整理、搜索、更新知识内容等操作。

## 安装与发现

```bash
# 查看这个仓库里可安装的技能
npx skills add Chris8891/wisemind-ai --list

# 全局安装 WiseMindAI Skill
npx skills add Chris8891/wisemind-ai@wisemind-ai -g -y
```

说明：

- `-g` 表示全局安装
- `-y` 表示跳过确认
- 上面这组命令适合发布后的 GitHub 安装方式

也可以直接对支持安装 Skill 的 Agent 说一句自然语言：

```text
帮我安装 https://github.com/Chris8891/wisemind-ai 这个 Skill
```

如果你现在只是本地试用，直接看下面的“本地安装教程”。

## 这是什么

这是一个给 AI Agent 使用的 WiseMindAI 技能包。

它可以帮助 Agent：

- 保存笔记
- 保存文本、文件记录、网页链接和网页正文
- 保存到指定文件夹
- 写入知识库和知识文档
- 创建卡片集、目录和学习卡片
- 搜索、更新、删除已有内容
- 用 `resolve` 和 `upsert` 减少重复创建

## 使用前要求

在安装这个技能前，请先确认：

- 你的电脑已经安装并运行 WiseMindAI
- WiseMindAI 已开启本地 API
- 默认接口地址可访问：`http://127.0.0.1:38221`
- 你的 Agent 工具支持导入本地 Skill，或者支持读取本地 `SKILL.md`
- 如果你希望保存网页正文，宿主 Agent 最好还具备网页抓取或浏览能力

## 本地安装教程

### 第一步：准备技能目录

确保这个目录结构存在：

```text
skill/
├── README.md
├── README.en.md
└── SKILL.md
```

其中：

- `SKILL.md` 是给 AI Agent 读取的执行说明
- `README.md` 是中文说明
- `README.en.md` 是英文说明

### 第二步：把技能目录提供给 Agent 工具

把整个 `skill` 目录作为一个本地技能包导入你的 Agent 工具。

常见方式通常有两种：

1. 放进工具约定的本地 skills 目录
2. 在工具里选择“导入本地 skill”或“添加本地技能”

如果你的工具支持读取 GitHub Skill，也可以在发布后直接用上面的安装命令。

### 第三步：启动 WiseMindAI 并确认接口可用

启动 WiseMindAI 后，确认本地接口可访问：

```bash
curl http://127.0.0.1:38221/api/health
```

如果返回里有 `ok: true`，说明本地服务正常。

如果访问失败：

- 先检查 WiseMindAI 是否已经启动
- 再检查本地 API 是否已经打开
- 如果你用的不是默认端口，需要在使用时告诉 Agent 正确端口

### 第四步：做一次安装验证

安装完成后，可以直接给 Agent 一条简单指令：

```text
把这段内容保存到 WiseMindAI，标题叫“安装测试”：
这是我的第一条测试内容。
```

如果 Agent 能成功保存，说明技能已经生效。

## 使用教程

### 用法 1：保存普通笔记

你可以直接说：

```text
把这段会议纪要保存到 WiseMindAI，标题叫“周会记录”。
```

适合：

- 会议纪要
- 灵感记录
- 草稿内容
- 随手笔记

### 用法 2：保存网页

你可以直接说：

```text
把这个网页保存到 WiseMindAI 的“待读”文件夹：
https://example.com/article
```

如果宿主 Agent 能抓到网页正文，技能会尽量把标题、摘要、正文一起保存。  
如果抓不到正文，通常会退化成只保存链接，或者要求你补充正文内容。

### 用法 3：保存到知识库

你可以直接说：

```text
把这篇内容放进 WiseMindAI 的“AI 研究”知识库，标题叫“RAG Notes”。
```

适合：

- 文章整理
- 长文归档
- 主题知识沉淀
- 项目资料归类

### 用法 4：创建学习卡片

你可以直接说：

```text
根据这段内容创建 8 张学习卡片，放到“AI 基础 / 检索增强”。
```

适合：

- 知识拆卡
- 术语记忆
- 复习材料整理

### 用法 5：搜索和更新

你可以直接说：

```text
在 WiseMindAI 里搜索一下 “RAG” 相关内容。
```

或者：

```text
帮我更新 WiseMindAI 里那篇 “RAG Notes” 的摘要。
```

## 推荐提示词

- 把这段内容保存到 WiseMindAI，标题叫“周会记录”。
- 把这个网页保存到 WiseMindAI 的“待读”文件夹。
- 在 WiseMindAI 里搜索一下 “RAG” 相关内容。
- 把这篇文章放进“AI 研究”知识库。
- 根据这段内容创建 8 张学习卡片，放到“AI 基础 / 检索增强”。
- 帮我更新 WiseMindAI 里那篇 “RAG Notes” 的摘要。
- 删除 WiseMindAI 里那条测试笔记“安装测试”。

## 这个技能默认会怎么工作

- 先检查 WiseMindAI 服务是否在线
- 不确定参数时先读取能力清单
- 写入前优先搜索，减少重复内容
- 需要文件夹、知识库、卡片集时优先自动按名称定位或创建
- 修改前先读取，删除前只在目标明确时执行

## 仓库结构

当前技能目录如下：

```text
skill/
├── README.md
├── README.en.md
└── SKILL.md
```

如果你准备让 `skills` CLI 更稳定识别，建议发布时使用 Agent Skills 标准目录结构。

## 限制说明

- 这个技能依赖本地运行中的 WiseMindAI，服务没开就不能用
- 如果用户只给了一个网页链接，而宿主 Agent 又拿不到网页正文，那么通常只能保存链接，或者需要用户补充内容
- 删除操作应该只在用户明确要求时执行

## 文件说明

- `SKILL.md`：给 AI Agent 看的执行说明
- `README.md`：中文说明
- `README.en.md`：英文说明
