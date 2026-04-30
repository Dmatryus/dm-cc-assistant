---
name: plan
description: Heavy interactive planning of the active epic. Decomposes it into atomic subtasks, builds DAG and waves, identifies risks, captures DoD. Pre-condition — one epic in In Progress (set via /backlog Pick from Todo).
disable-model-invocation: true
---

# plan — планирование активного эпика (v0.3)

Это skill вызывается командой `/dm-cc-assistant:plan`. Он запускает `epic-planner` через Task tool.

## Твоя роль как основного Claude

Ты — диспетчер. Сам планирование не ведёшь — это работа subagent'а.

## Pre-check

```bash
test -f ./OVERVIEW.md && test -f ./ARCHITECTURE.md && test -f ./CLAUDE.md && echo DOCS_OK || echo DOCS_MISSING
test -f .task/backlog.md && echo BACKLOG_OK || echo BACKLOG_MISSING
test -f .task/.execute-active && echo EXECUTE_ACTIVE || echo NO_EXECUTE
```

**DOCS_MISSING** — сообщи:

> Нет основных docs (OVERVIEW.md / ARCHITECTURE.md / CLAUDE.md). Запусти `/dm-cc-assistant:project-init` сначала.

Завершай.

**BACKLOG_MISSING** — recovery dialog (см. §13.7 дизайн-документа):

> Нет `.task/backlog.md`. Запустить `/dm-cc-assistant:backlog`?

STOP. Если «да» — советуй пользователю запустить `/dm-cc-assistant:backlog` и **не** chain'ом запусти что-то сам — пусть пользователь решает порядок. Завершай.

**EXECUTE_ACTIVE** — предупреди:

> Сейчас идёт `/execute`. Планирование может конфликтовать. Запустить `/plan` всё равно?

STOP. Если «нет» — завершай.

## Активный эпик — pre-detect

Прежде чем запускать subagent — посмотри сам, есть ли активный эпик:

```bash
awk '/^## In Progress$/{f=1; next} /^## /{f=0} f' .task/backlog.md | grep -c '^### E-' || true
```

| Ответ | Действие |
|---|---|
| `0` | Recovery: «Активного эпика нет. Запустить `/dm-cc-assistant:backlog` (Pick from Todo)?» STOP, завершай если «нет» |
| `1` | OK, запускай subagent |
| `>1` | Recovery: «В In Progress больше одного эпика — сломанное состояние. Зайди в `/dm-cc-assistant:backlog` и используй Unselect active epic для всех кроме одного.» Завершай. |

## Вводное сообщение

> Запускаю планирование активного эпика. Это **тяжёлый интерактив** — 7 фаз с подтверждениями: контекст / декомпозиция / DAG + волны / риски / открытые вопросы / DoD / финальный review. Прерваться можно на phase 7 (Abort).

## Запуск агента

Запусти Task с `subagent_type: epic-planner`. Prompt:

> Прочитай `.task/backlog.md`, найди эпик в `## In Progress`. Проведи 7-фазное планирование согласно своим инструкциям. На фазе 7 (Final review) после Accept — запиши `.task/plan-E-XXX.md` и обнови backlog.md (добавь подзадачи под активным эпиком). Верни короткий отчёт со словом «подтвердил».

## Проверка после агента

```bash
test -f .task/plan-E-*.md 2>/dev/null && echo OK || echo MISSING
```

Если `MISSING` и отчёт не содержит «подтвердил» — сообщи:

> Plan не записан (Abort или ошибка). Запусти `/dm-cc-assistant:plan` ещё раз когда будешь готов.

Если `OK` — выведи итоговую подсказку:

> Plan готов. Подзадачи добавлены в backlog. Следующий шаг — `/dm-cc-assistant:execute` для автономного прогона.

## Важные правила

- **Не делай шагов за агента.** Ты не декомпозируешь, не строишь DAG, не пишешь plan.md — это работа epic-planner.
- **Не продолжай при отказе.** Abort на фазе 7 — штатная остановка. План не пишется, эпик остаётся In Progress без подзадач (валидное состояние).
- **Относительные пути.** Всегда `.task/plan-E-XXX.md`, не абсолютные.
- **Не вызывай `/execute` автоматически.** Подсказку даёшь — пользователь решает сам.
- **Не chain'ь recovery dialogs.** При recovery ты только советуешь команду — пользователь сам её запускает.
