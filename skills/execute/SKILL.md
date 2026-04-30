---
name: execute
description: Autonomous parallel execution of the active epic's subtasks. Reads plan-E-XXX.md, runs waves of execution-agents in worktrees, merges results between waves via release-manager, runs in-execute review by code-reviewer, aggregates report, then engages user in a 4-step final dialog. Pre-conditions — active epic in In Progress with .task/plan-E-XXX.md ready.
disable-model-invocation: true
---

# execute — автономный прогон активного эпика (v0.3)

Это skill вызывается командой `/dm-cc-assistant:execute`. Он оркестрирует параллельное исполнение подзадач активного эпика через `execution-agent`'ов, мерджит результаты через `release-manager` (merge mode), агрегирует через `release-manager` (aggregation mode), запускает `code-reviewer` (in-execute mode), и ведёт 4-шаговый финальный диалог.

## Концепция

«Автономно от старта до финала, диалог только в начале и в конце». Pre-execute confirmation (один раз) → автономный прогон всех волн → агрегированный отчёт + диалог.

## Твоя роль как основного Claude

Ты **orchestrator** — спаунишь subagent'ов через Task tool и управляешь pipeline'ом. Сам код не пишешь, не мёрджишь, не делаешь review.

---

## Шаг 1 — pre-checks

```bash
test -f ./CLAUDE.md && test -f ./OVERVIEW.md && test -f ./ARCHITECTURE.md && echo DOCS_OK || echo DOCS_MISSING
test -f .task/backlog.md && echo BACKLOG_OK || echo BACKLOG_MISSING
test -f .task/.execute-active && echo EXECUTE_ACTIVE || echo NO_EXECUTE
```

**DOCS_MISSING** — отказ: «Нет основных docs. Запусти `/dm-cc-assistant:project-init` сначала.»

**BACKLOG_MISSING** — recovery: «Нет `.task/backlog.md`. Запусти `/dm-cc-assistant:backlog` для создания.» Завершай.

**EXECUTE_ACTIVE** — отказ: «Уже идёт `/execute` в другой сессии (есть `.task/.execute-active`). Завершите его или удалите marker вручную.» Завершай.

## Шаг 2 — найти активный эпик и план

```bash
awk '/^## In Progress$/{f=1; next} /^## /{f=0} f' .task/backlog.md | grep '^### E-'
```

| Найдено | Действие |
|---|---|
| `0` эпиков | Recovery: «Активного эпика нет. Запусти `/dm-cc-assistant:backlog` (Pick from Todo).» Завершай. |
| `1` эпик | OK, извлеки E-ID активного эпика |
| `>1` | Recovery: «Сломанное состояние — больше одного In Progress эпика. Зайди в `/dm-cc-assistant:backlog` (Unselect active epic).» Завершай. |

```bash
test -f .task/plan-E-XXX.md && echo PLAN_OK || echo PLAN_MISSING
```

**PLAN_MISSING** — recovery: «Эпик E-XXX в работе, но плана нет. Запусти `/dm-cc-assistant:plan` сначала.» Завершай.

## Шаг 3 — pre-execute analysis (детект остатков от прошлого прогона)

Проверь наличие worktree'ев и report'ов:

```bash
git worktree list
ls .task/report-E-XXX.*.md 2>/dev/null
ls .task/research-E-XXX.*.md 2>/dev/null
```

Для каждой подзадачи `E-XXX.Y` из плана сопоставь с состоянием:

| Состояние | Default action |
|---|---|
| Worktree + commits + report Status=Done + smoke OK | Skip — пометить Done в текущем плане, не перезапускать |
| Worktree + commits + report Status=Partial | Запросить: Resume / Restart |
| Worktree + commits + НЕТ отчёта | Запросить: Resume / Restart |
| Worktree + commits + report Status=Failed | Запросить: Restart / re-open `/plan` / Skip |
| Только ветка без worktree | Recreate worktree (Resume) |
| Worktree без коммитов | Clean safe (просто заново начнёт) |
| **Uncommitted changes в worktree** | **Hard stop** — ручной разбор; завершать `/execute` |

Если есть подзадачи с состоянием `Resume / Restart` — спрашивай по каждой:

> **E-XXX.Y** — обнаружено: <состояние>. Действие? (Resume / Restart / Skip / Manual)

STOP. Жди ответ по каждой.

**Если выбран Resume для подзадачи** — спауни `resume-analyzer` через Task tool (один вызов на подзадачу). Он напишет `.task/resume-E-XXX.Y.md`.

Покажи пользователю summary recommendation от resume-analyzer:

> **E-XXX.Y resume analysis:** Confidence: <high/medium/low>, Action: <Resume safe / Restart recommended / ...>
>
> Принять? (yes / переключить на Restart)

STOP.

## Шаг 4 — base branch setup

Если `feat/E-XXX` ещё не существует:

```bash
git checkout main
git pull --ff-only  # опционально, если хочешь синхронизироваться
git checkout -b feat/E-XXX
```

Если существует — `git checkout feat/E-XXX`.

## Шаг 5 — pre-execute summary + confirm

Прочитай `.task/plan-E-XXX.md`. Покажи пользователю краткий summary того, что будет запущено:

> **Активный эпик:** E-XXX [Priority] (Type) <Title>
>
> **Базовая ветка:** feat/E-XXX
>
> **Волны (M):**
> - 🌊 Волна 1 (параллельно): E-XXX.1, E-XXX.2 — типы: impl, impl
> - 🌊 Волна 2 (после волны 1): E-XXX.3 — тип: test
>
> **Контекст для каждой подзадачи:**
> - E-XXX.1: spec из plan §2, файлы [...], DoD [...], доп. контекст [...]
> - E-XXX.2: ...
> - E-XXX.3: ...
>
> **Что произойдёт:**
> 1. Создам / переиспользую worktree'ы под каждую подзадачу
> 2. Запущу подзадачи волнами параллельно через Task tool
> 3. Между волнами — release-manager (merge mode) объединит ветки в feat/E-XXX
> 4. После всех волн — code-reviewer (in-execute) + release-manager (aggregation)
> 5. Финальный диалог по результатам
>
> **Подтверди старт:** [y] go / [a] abort

STOP. Жди явный ответ.

## Шаг 6 — создать `.execute-active` marker

```bash
echo "E-XXX started $(date '+%Y-%m-%d %H:%M')" > .task/.execute-active
```

Подставь реальный E-ID активного эпика вместо `E-XXX`. `$(date ...)` подставит актуальный timestamp при выполнении — **не** записывай буквально `YYYY-MM-DD HH:MM`.

Этот marker сообщает SessionStart hook'у и другим командам, что идёт `/execute`.

## Шаг 7 — wave loop (главный цикл)

Для каждой волны `W` (от 1 до M):

### 7.1. Cascade check

Прежде чем запускать волну, проверь зависимости:

Для каждой подзадачи в волне `W`:
- Если **любая** dependency (по DAG из plan'а) не в `Done` (Failed / Skipped / Conflict-blocked / Cancelled) → пометь подзадачу `Skipped` с reason `blocked by E-XXX.Z`
- Иначе → подзадача активна для исполнения

Если все подзадачи волны `W` помечены `Skipped` — пропусти волну, иди к `W+1`.

### 7.2. Spawn execution-agents (parallel)

**В одном сообщении** запусти все активные подзадачи волны через Task tool с `subagent_type: execution-agent`:

```
[Task tool call для E-XXX.A]
[Task tool call для E-XXX.B]
[Task tool call для E-XXX.C]
```

Prompt для каждого execution-agent (см. agents/execution-agent.md «Receive prompt»):

```
Эпик: E-XXX
Подзадача: E-XXX.A
Тип: <type>
Title: <title>

Worktree: .worktrees/E-XXX.A
Ветка: feat/E-XXX.A-<slug>
Базовая ветка: feat/E-XXX

Plan: .task/plan-E-XXX.md (читай §2 для своей подзадачи)
Спецификация подзадачи:
  - Что делает: <из плана>
  - Файлы: <из плана>
  - DoD: <из плана>
  - Зависит от: <из плана>

Декларируемый контекст:
  - <files>

[Если Resume:]
Resume context: .task/resume-E-XXX.A.md — читай его перед стартом.

Согласно своим инструкциям: setup worktree, прочитай план + spec + контекст, реализуй,
коммить, прогоняй smoke check, пиши .task/report-E-XXX.A.md (5 секций), верни summary.
```

**Wait all returns.** Соберём статусы:

```
E-XXX.A → Done | Failed | Partial
E-XXX.B → ...
```

Обнови inline-статусы подзадач в `.task/backlog.md` (сразу после волны, до merge'а — `Done` / `Failed` / `Partial`).

### 7.3. Spawn release-manager (merge mode)

Один Task tool call:

```
subagent_type: release-manager
prompt:

Mode: merge
Эпик: E-XXX
Волна: W
Подзадачи в волне:
  - E-XXX.A (Status: Done) — branch feat/E-XXX.A-<slug>
  - E-XXX.B (Status: Failed) — branch feat/E-XXX.B-<slug>
  - E-XXX.C (Status: Done) — branch feat/E-XXX.C-<slug>

Согласно своим инструкциям: на feat/E-XXX замёрджи Done подзадачи, резолви конфликты по 3-уровневой схеме (combine → auto-pick → user dialog flag), cleanup worktree'ев Done, обнови inline-статусы. Verни summary.
```

Wait return.

### 7.4. Optional post-wave smoke check

Если в `plan-E-XXX.md` для этой волны есть post-wave smoke command (например, общий build) — запускай. Зафиксируй результат, **не блокируй** следующую волну.

Cascade на downstream `Failed` / `Conflict-blocked` подзадач выполняется в **следующей** волне через шаг 7.1 — отдельная запись здесь не нужна (дублирование).

### 7.5. → Wave W+1

Если ещё есть волны — продолжай. Иначе — выход из цикла.

## Шаг 8 — code-reviewer (in-execute)

После всех волн:

```
subagent_type: code-reviewer
prompt:

Mode: in-execute
Эпик: E-XXX

Согласно своим инструкциям: пройди cross-subtask quality review на feat/E-XXX vs main, пиши .task/review-E-XXX.md (severity-tagged findings).
```

Wait return.

## Шаг 9 — release-manager (aggregation)

```
subagent_type: release-manager
prompt:

Mode: aggregation
Эпик: E-XXX

Согласно своим инструкциям: собери .task/report-E-XXX.md из per-subtask отчётов + merge logs + code-review findings.
```

Wait return.

## Шаг 10 — финальный диалог (4 шага)

### Step 1 — narrative summary

Прочитай `.task/report-E-XXX.md`. Сгенерируй живой narrative summary с фиксированной структурой:

```markdown
# Epic E-XXX — отчёт

## Сделано
<connected текст что закрыто, что в feat/E-XXX>

## Не сделано
<failed / skipped / blocked, конкретно>

## Что это значит
<implication для проекта: что не работает, чего не хватает>

## Требуются действия
<главная развилка с рекомендацией и обоснованием>

## Иначе
<последствия каждого пути: accept partial / defer>

## Дополнительно
<обзор decisions / open questions / follow-ups / review findings — счётчики>

## Что обсуждаем?
<меню — см. step 2>
```

Тон conversational, скелет фиксированный. Длина — по содержанию, без искусственных лимитов.

### Step 2 — финальное меню (фиксированный порядок)

Покажи меню в порядке убывания важности:

```
1. Obstacles         (X failed + Y skipped)
2. Conflicts         (N high-stakes blocked)
3. Review findings   (Critical / High / Medium / Low)
4. Open questions    (N)
5. Decisions         (M)
6. Follow-ups        (K)
7. Show full report
8. Skip, go to final action
```

Стрелка `← рекомендую начать здесь` ставится туда, где реально требует внимания: Obstacles → Conflicts → Review Crit/High → Open → ничего.

STOP. Жди выбор.

### Step 3 — sub-dialogs по выбранным темам

Каждая категория имеет свои действия:

#### Obstacles

Для каждой failed подзадачи:
- **Re-plan** — запустить `/dm-cc-assistant:plan` (Iterate phase 2) для перепланирования
- **Replace DoD and re-run** — изменить DoD подзадачи, запустить partial /execute (см. step 4)
- **Skip permanently** — подзадача → `Cancelled`, worktree удаляется
- **Manual intervention** — выйти, пользователь сам разбирается

STOP по каждой.

#### Review (Critical / High)

Для каждого finding'а:
- **Fix now (mini-execution-agent)** — спаунь execution-agent на однократный fix, на feat/E-XXX. (Минимальная подзадача — добавить commit с fix'ом, не делать новую ветку.)
- **Acknowledge as known limitation** — добавить в ноту, попадёт в CHANGELOG секцию Known Limitations при /release
- **Abort release** — выйти без финального действия (вернуться к Defer)

STOP по каждому Critical / High.

#### Review (Medium / Low)

Для каждого finding'а:
- **Accept** — игнорим
- **Fix** — как Fix now
- **Defer** — добавить в follow-ups агрегата

STOP.

#### Open questions

Для каждого:
- **Resolve now** — обсуди, ответ записывается в backlog или эпик-отчёт
- **Defer** — оставить открытым в `## Open Questions` backlog'а
- **Promote to research subtask** — создать research-подзадачу: запустить `/dm-cc-assistant:plan` (Iterate phase 5) или вручную добавить в backlog

STOP.

#### Decisions

Для каждой:
- **Accept** — оставить как есть
- **Mark for re-visit** — добавить в follow-ups агрегата с пометкой Re-visit
- **Revert (новая subtask)** — создать новую mini-подзадачу для отката решения, запустить execution-agent

STOP.

#### Conflicts (high-stakes only)

Для каждого Conflict-blocked:
- **Take version 1** — резолв в пользу одной из веток
- **Take version 2** — резолв в пользу другой
- **Combine (опиши как)** — пользователь объясняет логику combine, спауним mini-execution-agent для применения
- **Skip merge** — подзадача → `Cancelled`, worktree удаляется
- **Show full diff** — показать полный diff для решения

STOP.

#### Follow-ups

Для каждого:
- **Accept** — попадает в backlog как новая подзадача (если эпик ещё активен) или новый эпик (Parking lot)
- **Reject** — игнорим

STOP.

### Step 4 — финальное действие

```
[r] /release         — текущее состояние feat/E-XXX готово к релизу
[i] Iterate          — re-plan failed/blocked, переход в /plan phase 2
[p] Partial /execute — перезапустить только failed/skipped без re-plan
[d] Defer            — оставить эпик в In Progress, выйти
[a] Abort epic       — откатить feat/E-XXX, эпик → Cancelled
```

**Нельзя выйти без явного финального действия** — переспрашивай.

#### `[r]` /release

Просто советуй пользователю запустить `/dm-cc-assistant:release`. Не вызывай chain'ом.

#### `[i]` Iterate

Советуй `/dm-cc-assistant:plan` (он сам определит что эпик уже планирован и предложит Iterate phase X).

#### `[p]` Partial /execute

Снова перезапустить `/execute`. Pre-execute analysis детектит, что часть подзадач Done — пропустит их, запустит только failed/skipped (с подтверждением пользователя на каждой).

Это **новый запуск skill'а** — пользователь делает руками `/dm-cc-assistant:execute`. Ты только советуешь.

#### `[d]` Defer

Эпик остаётся In Progress. Выйти. Удалить marker (см. шаг 11).

#### `[a]` Abort epic

Подтверждение:

> Удалить ВСЕ worktree'ы эпика + ветка feat/E-XXX + пометить эпик как Cancelled? (yes / no)

STOP. Если yes:

```bash
# Удалить все worktrees связанные с эпиком
for wt in $(git worktree list | grep "feat/E-XXX" | awk '{print $1}'); do
  git worktree remove --force "$wt"
done

# Удалить ветки
git checkout main
for branch in $(git branch | grep "feat/E-XXX"); do
  git branch -D "$branch"
done
```

Обновить backlog: `## In Progress` E-XXX → `## Cancelled` с датой.

Edit log в backlog.md:

```
> - YYYY-MM-DD · v0.3.0 · execute (orchestrator) · E-XXX → Cancelled (Abort epic)
```

## Шаг 11 — cleanup marker и выход

```bash
rm .task/.execute-active
```

Финальный summary:

> `/execute` завершён. Действие: <r/i/p/d/a>. Дальнейший шаг: <соответствующая подсказка>.

---

## Важные правила

- **Один pre-execute confirm + автономия до финального диалога.** Никакой интерактивности между волнами.
- **Subagents изолированы** — execution-agents не видят друг друга и conversation-history. Файловая система общая.
- **Параллельные Task calls — в одном сообщении.**
- **Ты не пишешь plan, не делаешь merge, не пишешь review** — это работа subagent'ов.
- **`.task/.execute-active` всегда удаляется** в конце (даже при Abort, defer, ошибке).
- **Прерывание сессии** (crash, network) → marker остаётся. Следующий `/execute` детектит — recovery dialog.
- **Cascade — бинарное правило:** любая dependency не в Done → подзадача Skipped с reason.
- **High-stakes conflicts** — только в финальном диалоге (Conflicts), не во время волн.
- **Нельзя пропустить финальное действие** — переспрашивай.

## Обработка ошибок subagent'ов

| Ошибка | Действие |
|---|---|
| Execution-agent вернул Status=Failed | Cascade на зависимых, продолжаем волны |
| Execution-agent вернул некорректный summary | Перечитать его report-E-XXX.Y.md, доверять файлу |
| Release-manager (merge) → hard stop (uncommitted changes) | Прерывать `/execute`, сообщить пользователю, оставить marker (для повторного запуска ручного разбора) |
| Code-reviewer не пишет файл | Считать «нет findings», продолжить aggregation |
| Release-manager (aggregation) не пишет файл | Hard stop — aggregation обязательна для финального диалога |

## NEVER

- **NEVER** запускай `/execute` если есть `.task/.execute-active`.
- **NEVER** запускай subagent'ов вне Task tool'а.
- **NEVER** делай git commit / merge / push сам — это работа subagent'ов.
- **NEVER** показывай findings / decisions / obstacles разом — строго через step 2 menu + step 3 sub-dialogs.
- **NEVER** пропускай финальное действие — переспрашивай.
- **NEVER** удаляй `.execute-active` до завершения финального действия.
- **NEVER** chain'ом запускай `/release`, `/plan`, `/backlog` — только советуй пользователю.
