---
name: release
description: Prepares release materials for the active epic locally on feat/E-XXX (CHANGELOG entry, release notes, announcement, migration guide if breaking, version bumps). Runs independent pre-release review via code-reviewer. Commits release prep + backlog update on feat/E-XXX. Prints user instructions for merge/tag/push (DOES NOT execute them). Pre-condition — active epic with .task/report-E-XXX.md from /execute.
disable-model-invocation: true
---

# release — подготовка релиза активного эпика (v0.3)

Это skill вызывается командой `/dm-cc-assistant:release`. Без аргументов и флагов. Активный эпик определяется автоматически из `## In Progress`.

Skill оркестрирует:
1. Pre-release review через `code-reviewer` (mode `release-readiness`)
2. Диалог с пользователем по Critical / High findings
3. Подготовка артефактов через `release-manager` (mode `full`)

## Концепция

**Агент готовит материалы, не релизит сам.** Skill пишет файлы локально, коммитит на `feat/E-XXX`, печатает инструкции пользователю. **Не делает** `git merge → main`, `git tag`, `git push`. Это руки пользователя.

## Твоя роль как основного Claude

Ты **orchestrator** — спаунишь subagent'ов через Task tool и ведёшь диалог по review findings между ними.

---

## Шаг 1 — pre-checks

```bash
test -f ./CLAUDE.md && test -f ./OVERVIEW.md && test -f ./ARCHITECTURE.md && echo DOCS_OK || echo DOCS_MISSING
test -f .task/backlog.md && echo BACKLOG_OK || echo BACKLOG_MISSING
test -f .task/.execute-active && echo EXECUTE_ACTIVE || echo NO_EXECUTE
```

**DOCS_MISSING** — отказ: «Нет основных docs.»

**BACKLOG_MISSING** — recovery: «Нет backlog'а. Запусти `/dm-cc-assistant:backlog` сначала.» Завершай.

**EXECUTE_ACTIVE** — отказ: «Сейчас идёт `/execute`. Дождись завершения.» Завершай.

## Шаг 2 — найти активный эпик и report

```bash
awk '/^## In Progress$/{f=1; next} /^## /{f=0} f' .task/backlog.md | grep '^### E-'
```

| Найдено | Действие |
|---|---|
| `0` эпиков | Recovery: «Активного эпика нет.» Завершай. |
| `1` эпик | OK, извлеки E-ID |
| `>1` | Recovery: «Сломанное состояние.» Завершай. |

```bash
test -f .task/report-E-XXX.md && echo REPORT_OK || echo REPORT_MISSING
```

**REPORT_MISSING** — recovery: «Нет агрегированного report'а. Запусти `/dm-cc-assistant:execute` сначала (он делает aggregation в конце).» Завершай.

## Шаг 3 — preconditions check на ветке

```bash
git rev-parse --abbrev-ref HEAD
```

Должен быть `feat/E-XXX`. Если нет:

```bash
git checkout feat/E-XXX
```

```bash
git status --short
```

Если есть uncommitted changes — отказ: «На feat/E-XXX uncommitted changes. Закоммить их или откати, потом перезапусти `/release`.»

## Шаг 4 — verify aggregated report status

Прочитай `.task/report-E-XXX.md`, найди поле `Status:`.

| Status | Действие |
|---|---|
| `Success` | OK, продолжаем |
| `Partial` | Спроси: «Эпик завершён частично. Релизим как есть? Failed/Skipped попадут в Known Limitations CHANGELOG.» STOP. Если «нет» — завершай. |
| `Failed` | Отказ: «Эпик целиком Failed. Нечего релизить. Запусти `/dm-cc-assistant:execute` ещё раз с iterate.» |
| `Unresolved conflicts` | Отказ: «Есть Conflict-blocked подзадачи. Закрой их через финальный диалог `/execute`.» |

## Шаг 5 — code-reviewer (release-readiness)

Спауни subagent через Task tool:

```
subagent_type: code-reviewer
prompt:

Mode: release-readiness
Эпик: E-XXX

Согласно своим инструкциям: проанализируй feat/E-XXX vs main с фокусом на shipping readiness (public API, breaking changes, migration, README, version consistency). Пиши .task/review-E-XXX-pre-release.md (severity-tagged findings).
```

Wait return.

## Шаг 6 — диалог по review findings

Прочитай `.task/review-E-XXX-pre-release.md`.

Покажи stats:

> **Pre-release review:**
> - Critical: <X>
> - High: <Y>
> - Medium: <Z>
> - Low: <W>

### Step 6.1 — Critical / High (блокируют release)

Если Critical + High > 0:

> **Critical / High findings блокируют release. Что делаем?**

Для каждого finding'а (по одному):

> **PR-<N> [Critical/High]: <Title>**
>
> <описание>
>
> Suggestion: <suggestion>
>
> Действия:
> - **[f] Fix now** — спауни mini-execution-agent для применения fix'а
> - **[a] Acknowledge as known limitation** — добавить в CHANGELOG `### Known Limitations`
> - **[A] Abort release** — прервать /release

STOP по каждому.

#### Если `Fix now`

Спауни Task tool call:

```
subagent_type: execution-agent
prompt:

Эпик: E-XXX
Подзадача: pre-release-fix-PR-<N>
Тип: impl
Title: Fix PR-<N>: <title>

Worktree: <текущий cwd> (работаем прямо на feat/E-XXX)
Ветка: feat/E-XXX (без новой ветки)

Plan: .task/review-E-XXX-pre-release.md (читай finding PR-<N>)
Спецификация:
  - Что делает: применить suggestion из PR-<N>
  - Файлы: <из finding'а>
  - DoD: finding закрыт, smoke check проходит
  - Зависит от: —

Согласно своим инструкциям: реализуй fix, коммить на feat/E-XXX (без worktree, прямо в текущем месте), прогоняй smoke check, пиши .task/report-pre-release-fix-PR-<N>.md.
```

После return — переход к следующему finding'у.

#### Если `Acknowledge as known limitation`

Запомни (внутренняя переменная) — finding пойдёт в CHANGELOG секцию `### Known Limitations`. release-manager (full) увидит это в prompt.

#### Если `Abort release`

`/release` завершается. Никаких изменений (но `Fix now` коммиты остаются — пользователь решает что с ними).

> Release прерван. Незакрытые findings: <list>. Запусти `/release` ещё раз когда они будут закрыты.

Завершай.

### Step 6.2 — Medium / Low (опционально)

Если Medium + Low > 0:

> **Medium / Low findings не блокируют. Обрабатывать?**
> - [y] yes, по одному
> - [n] no, accept все

Если no — все Accept (игнорируем). Если yes — для каждого:

> **PR-<N> [Medium/Low]: ...**
>
> Действия:
> - [a] Accept (игнорим)
> - [f] Fix (как Fix now выше)
> - [d] Defer (в follow-ups)

STOP по каждому.

## Шаг 7 — release-manager (full mode)

Соберём контекст для prompt'а:

- Список acknowledged limitations: <list> (из step 6.1)
- Список deferred findings: <list> (из step 6.2)

Спауни Task tool call:

```
subagent_type: release-manager
prompt:

Mode: full
Эпик: E-XXX

Acknowledged limitations:
- PR-<N>: <title>
- PR-<M>: <title>

Deferred findings (в follow-ups):
- PR-<K>: <title>

Pre-release review: .task/review-E-XXX-pre-release.md (уже прочитан)
Aggregated report: .task/report-E-XXX.md

Согласно своим инструкциям: предложи version bump, подготовь release artifacts (CHANGELOG, release-notes, announcement, tag message, migration если breaking), bump версии в файлах, commit "release: prepare vX.Y.Z" + "release: update backlog for vX.Y.Z" на feat/E-XXX. Обнови backlog (E-XXX → Done), auto-archive если >5 Done. Печать инструкции пользователю — НЕ делай push/tag/merge.
```

Wait return.

## Шаг 8 — финальный summary

Когда release-manager вернёт control:

> ✓ Release prep готов. См. инструкции выше от release-manager'а.
>
> **Коротко:**
> - Версия: vX.Y.Z
> - Backlog: E-XXX → Done
> - Коммиты на feat/E-XXX: 2 (release prep + backlog update)
>
> **Дальше — твои руки:** merge в main, tag, push, GitHub Release.

Если release-manager вернул отказ (например, был блок) — отрази в summary, не печатай ложный success.

---

## Edge case: запуск `/release` повторно после уже подготовленного коммита

release-manager сам детектит это в pre-execute analysis (см. agents/release-manager.md). Он спросит Continue / Rollback / Abort.

---

## Важные правила

- **Не делай шагов за агента.** Ты orchestrator — диалог по review между subagent'ами.
- **Не продолжай при отказе.** Abort на любом шаге — штатная остановка.
- **Относительные пути.** Всегда `.task/...`, `CHANGELOG.md`, `.claude-plugin/...`.
- **Не chain'ом запускай другие команды** — только советуй (например, при Partial релизе после Abort — пользователь сам решает что делать).
- **Не пропускай pre-release review.** Code-reviewer обязателен.
- **Не пропускай Critical / High findings** без явного выбора пользователя.
- **Push / tag / merge — только пользователь.** Skill этого не делает.

## NEVER

- **NEVER** запускай `/release` если есть `.task/.execute-active`.
- **NEVER** запускай subagent'ов вне Task tool'а.
- **NEVER** делай `git push`, `git tag`, `git merge` в main, `git checkout main`.
- **NEVER** игнорируй Critical / High findings — только через явный choice.
- **NEVER** показывай findings разом — по одному.
- **NEVER** chain'ом запускай `/execute`, `/plan`, `/backlog` — только советуй.
- **NEVER** редактируй файлы сам — только через subagent'ов.
