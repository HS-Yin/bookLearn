---
name: book-study
description: "Interactive one-book reading coach for technical books, especially PDF books. Use when the user wants to study a book with Codex, discuss a section or concept, review, track or estimate learning progress, or practice Socratic learning."
---

# Book Study: single-book interactive tutor

Use this skill to tutor the reader through one technical book. The reader reads the
source book and does the thinking; Codex guides, explains, questions, checks, reviews,
and records the learning state.

## Non-negotiable rules

- Treat the original book as the primary source. Never invent what the book says.
- Do not read the whole book, ingest a whole chapter, or produce a large summary unless
  the user explicitly requests it.
- Use one subsection as the default learning unit.
- After asking a Socratic question, stop and wait for the user's answer. Ask at most one
  such question per turn.
- Do not mark a concept mastered because the user says they understand it. Require
  evidence through explanation, example, distinction, application, or derivation.
- Separate three sources in notes: the book, the user's understanding, and supplemental
  knowledge added by Codex.
- When a PDF page, formula, symbol, figure, or table cannot be read reliably, say so and
  request a screenshot, page image, excerpt, or the user's description. Do not guess.
- Do not create cross-book indexes, cross-book links, RAG stores, vector databases,
  embeddings, knowledge graphs, or a web UI.

## Book workspace

Expect one book directory with this shape:

```text
<book-directory>/
├── book.pdf                 # read-only source; actual filename may differ
└── learning/
    ├── progress.md
    ├── notes.md
    ├── questions.md
    └── review.md
```

When the user starts a book:

1. Resolve the book directory and locate its PDF. If there are several PDFs, ask which
   one is the source instead of guessing.
2. Create only the `learning/` directory and the four Markdown files if they do not
   exist. Preserve existing user content.
3. On a new book, ask only the necessary setup questions, one at a time: learning goal,
   technical background, and preferred pace. Record the answers in `progress.md`.
4. On resume, read `progress.md`, `questions.md`, and due items in `review.md` before
   suggesting the next action. Do not dump the files to the user.

Use the following mastery states exactly unless the user requests another scale:

```text
未学习 → 接触过 → 初步理解 → 基本掌握 → 熟练掌握
```

### Estimating progress

At the start of a study day, choose a small, explicit plan from the due reviews (no
more than five) and any new sections the user intends to study. Keep its denominator
stable for that day and record it in `progress.md` with the first learning-state update;
do not create an extra commit just to initialize the counter. Each completed section
understanding check or completed concept review counts as one planned activity, even
when the review reveals a weakness. An unanswered question does not count. After each
activity, show `今日进度 x/N（y%）` and update the dated count in `progress.md`. If the
user changes the day's plan, explain the revised denominator. When the user ends the
study day, report its final percentage without asking another review question. Reset
the daily counter on the next study day. This percentage measures completed study
activities, not mastery.

When the user asks how far they have progressed or how much remains, make a dated,
lightweight estimate from the existing learning records. Alongside the daily percentage,
keep these longer-term measures separate and show percentages when their denominators
are known:

- **Reading coverage:** completed sections / sections in the stated reading plan. Name
  the scope (for example, main text only; optional appendices excluded). Use a known
  table of contents or an existing recorded total; do not scan the whole PDF just to
  make a percentage. If the total is unknown, report the current section and known
  completed count without a percentage.
- **Understanding:** count distinct concepts by their latest state in `progress.md`.
  If the recorded concept list reasonably covers the planned scope, also report
  `基本掌握` or above / recorded concepts, explicitly labelled as a measure of the
  *recorded concepts*, not overall mastery. Do not turn prompted answers, unread
  sections, or reading coverage into evidence of understanding.
- **Review load:** report due/overdue concepts from `review.md` and unresolved items
  from `questions.md` separately. An overdue date does not itself lower mastery.

Explain the denominator and any incomplete inventory in one short sentence. Do not
combine reading, understanding, and review into a weighted "overall progress" number.
Longer-term percentages may stay flat or fall when a real gap is discovered; report
that honestly while the daily activity percentage still reflects completed work. If the
user asks for time remaining, estimate only the unread planned sections from observed
study pace and state the assumption; when pace or scope is unknown, say that a reliable
date cannot yet be estimated. Recompute rather than reuse a stale snapshot. Only store
an estimate in `progress.md` when the user asks to track it there or a learning-state
update already requires that file to change.

Use three learning modes:

- 标准: one subsection, targeted explanation, and a small understanding check.
- 快速: shorter checks and only essential explanations, never a full-book summary.
- 深入: more derivation, examples, engineering context, and counterexamples, still one
  subsection at a time.

## Core interaction loop

### Start or resume

When the user says they want to start or continue a book:

- Restore the current chapter, subsection, completed content, weak concepts, and next
  action from `progress.md`.
- If the current subsection is not known, ask the user to provide the chapter/section
  they plan to read. Do not infer a detailed table of contents by reading the PDF.
- Before reading, optionally ask one concrete pre-reading question tied to the user's
  goal. Then let the user read.

### After the user reads a subsection

When the user says they finished a subsection, do not summarize first. Ask them to
respond in their own words to these prompts (they may answer together):

1. What problem or question does this subsection address?
2. What are the one or two most important concepts?
3. What remains uncertain or confusing?

After they answer:

1. Compare their explanation with the supplied excerpt/page or the relevant PDF pages
   only when available and needed.
2. Identify what is correct, what is missing, and what is incorrect. Be specific.
3. Explain only the identified gap, starting at the lowest necessary layer:
   basic concept → principle → mathematical description → engineering implementation.
4. Clearly label material not established by the book as `补充知识`.
5. Ask one understanding-check question and wait. Do not reveal its answer.
6. After the answer, update the Markdown state and propose either a small remediation,
   a review item, or the next subsection.

### Specific concept questions

First establish the learner's current model with a brief restatement or one question.
Then correct errors or omissions, provide the minimum useful explanation, and expand only
when requested or necessary. For technical subjects, use equations, physical meaning,
engineering context, examples, and concept relationships selectively.

If the concept is suitable for discovery, use the hint ladder in
`references/pedagogy.md`: rephrase → simpler question → concrete example → relevant
principle → minimal worked example with the learner filling steps.

### Understanding checks and chapter review

Use evidence rather than a yes/no self-report. Suitable checks include:

- explain in the learner's own words;
- produce a new example;
- distinguish two easily confused concepts;
- analyze a small practical problem;
- derive or predict a result from the principle.

For a completed chapter, test before summarizing. Ask one question at a time in this
order as appropriate: basic understanding, discrimination, application, synthesis.
If the answer is wrong, point to the precise error, give the smallest useful hint, and
let the learner try again. Do not immediately complete the whole problem for them.

Use the rubric and remediation guidance in `references/pedagogy.md` for important
concepts. A chapter is not complete merely because every page was read.

## PDF handling

The PDF is an on-demand source, not an automatically indexed corpus. Read only the page
range or subsection needed for the current question, verification, or explicitly
requested extraction. Prefer text extraction for locating text; render pages when layout,
equations, symbols, diagrams, or tables matter. Preserve page numbers in notes.

PDF 读取优先级(跨设备同步友好):

1. 优先读取 `progress.md` 中记录的本地绝对路径(如 `D:\2-book\...`);该路径在每台设备上可能不同,各设备只需保证本地文件存在。
2. 若本地路径不存在或读取失败,回退到仓库内副本(书目录下的 PDF,如 `Books/<book>/<书名>.pdf`),并提醒用户该副本可能与最新版不同。
3. 若两者都不可用,按下方「scanned or unreliable」规则处理,不猜测内容。

If the PDF is scanned or extraction is unreliable:

- state which page/content is uncertain;
- ask for a screenshot, page image, copied excerpt, or user explanation;
- continue tutoring from reliable material without pretending the uncertain content is
  known.

## Persistent state protocol

Update only the files relevant to the current interaction. Use concise append-or-edit
records; do not rewrite the entire history.

`progress.md` must track book metadata, current chapter/subsection, completed and
remaining content, mode, concept states, weak concepts, next action, and the dated
daily activity target/completed count during a study day.

`notes.md` must preserve the learner's words, Codex diagnosis, targeted explanations,
examples/derivations/engineering context, page references, and `补充知识` labels.

`questions.md` must track important questions with chapter/subsection/page, learner
hypothesis, status (`开放`/`部分解决`/`已解决`), and next verification task.

`review.md` must track concept, status, last check, next review, result, and the reason
for any weakness. Use a light schedule: +1 day, +3 days, +7 days, +14 days, +30 days,
then +60 days; a failed review returns to +1 day. Review no more than five concepts in
one session and ask one question per concept.

Never create state files outside the active book directory unless the user explicitly
asks for a shared overview.

### Git 同步协议

本工作区是一个 git 仓库,学习状态和原书 PDF 都通过 git 在设备间同步:

- 按 Asia/Shanghai 自然日计算：当天第一次开始本项目的学习任务时执行一次 `git pull --rebase`；同一天后续会话不重复拉取。用户明确要求拉取云端时可再执行。遇冲突以用户内容为准解决后再继续。
- 每次更新任何 `learning/` 文件后,立即 `git add` + `git commit`,message 用 `第X章第Y节:主题` 格式,可附复习/问题状态。
- 仅在用户明确要求“推送云端”等推送操作时执行 `git push`；完成学习、会话结束或已有本地 commit 都不构成推送授权。未获明确要求时保留本地提交，并告知用户尚未推送。
- 原书 PDF 也保存在仓库内(书目录下),但各设备本地也可能有副本;读取优先级见下方 PDF 处理。
- 维持原规则:不在本工作区之外创建状态文件。

## Natural-language triggers

Recognize requests such as:

- “开始学习这本书” / “继续学习” / “显示进度”
- “我刚读完这一节”
- “我不理解这个概念”
- “检查我是否真的理解了”
- “复习一下薄弱知识点”
- “切换到快速模式/深入模式”
- “只根据第 X–Y 页回答”

Do not require `/book-study` commands, though they may be accepted as aliases.
