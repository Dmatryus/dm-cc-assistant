---
name: backlog
description: Create or manage the project backlog with a two-level epic/subtask model. First run generates epics from OVERVIEW.md + ARCHITECTURE.md. Subsequent runs sync with docs, let you pick / add / unselect epics, and detect orphans. Use at the start of each dev session.
disable-model-invocation: true
---

# backlog — управление backlog проекта (v0.3)

Это skill вызывается командой `/dm-cc-assistant:backlog`. Он запускает `backlog-planner` через Task tool.

## Твоя роль как основного Claude

Ты — диспетчер. Сам backlog не генерируешь и эпики не обсуждаешь — это работа subagent'а.

## Pre-check

Прежде чем запускать subagent — проверь состояние:

```bash
test -f ./OVERVIEW.md && test -f ./ARCHITECTURE.md && echo DOCS_OK || echo DOCS_MISSING
test -f .task/backlog.md && echo BACKLOG_EXISTS || echo BACKLOG_NEW
test -f .task/.execute-active && echo EXECUTE_ACTIVE || echo NO_EXECUTE
```

**Если `DOCS_MISSING`** — сообщи пользователю:

> Нет `OVERVIEW.md` или `ARCHITECTURE.md`. Запусти `/dm-cc-assistant:project-init` сначала.

И завершай. Subagent в этом случае не вызывай.

**Если `EXECUTE_ACTIVE`** — предупреди:

> Сейчас идёт `/execute` (есть `.task/.execute-active` marker). Любые правки backlog'а могут конфликтовать с release-manager. Запустить `/backlog` всё равно?

STOP. Жди подтверждение. Если «нет» — завершай.

## Вводное сообщение

**Если `BACKLOG_NEW`**:

> Запускаю создание backlog. Ассистент проанализирует OVERVIEW.md и ARCHITECTURE.md, предложит план реализации в виде **эпиков** — логических групп задач с самостоятельной ценностью. Подзадачи будут добавлены позже, через `/plan`.

**Если `BACKLOG_EXISTS`**:

> Открываю backlog. Ассистент проверит синхронность с docs, найдёт orphan-файлы, и предложит варианты действий — выбрать эпик в работу, добавить новый, и т.п.

Если детект v0.2 формата происходит внутри subagent'а — он сам начнёт миграционный диалог.

## Запуск агента

Запусти Task с `subagent_type: backlog-planner`. Prompt:

**Если `BACKLOG_NEW`**:

> Прочитай `./OVERVIEW.md` и `./ARCHITECTURE.md`. Сгенерируй backlog проекта в двухуровневой модели согласно твоим инструкциям (5–12 эпиков). Обсуди с пользователем, получи подтверждение, запиши `.task/backlog.md`. Верни короткий отчёт со словом «подтвердил».

**Если `BACKLOG_EXISTS`**:

> Прочитай `.task/backlog.md`. Определи режим (v0.3 repeat-run или v0.2 migration). Выполни sync check, surface orphans, предложи стандартное меню согласно твоим инструкциям. Верни короткий отчёт о том что было сделано.

## Проверка после агента

Если это был первый запуск:

```bash
test -f .task/backlog.md && echo OK || echo MISSING
```

Если `MISSING` и отчёт не содержит «подтвердил» — сообщи:

> Backlog не создан. Можешь попробовать ещё раз: `/dm-cc-assistant:backlog`

## Итоговая подсказка

**Если эпик выбран (Todo → In Progress в этом запуске)**:

> Эпик в работе. Следующий шаг — `/dm-cc-assistant:plan` для декомпозиции на подзадачи.

**Если backlog создан (first-run), но эпик не выбран**:

> Backlog готов. Когда будешь готов начать работу — `/dm-cc-assistant:backlog` ещё раз и выбери эпик через `Pick from Todo`.

**Если миграция v0.2 прошла**:

> Backlog мигрирован в v0.3 формат. Запусти `/dm-cc-assistant:backlog` ещё раз для стандартного меню.

**Если миграция отменена (Abort)**:

> Миграция отменена. `/backlog` v0.3 не сработает на v0.2 формате. Запусти миграцию позже или вернись к старому процессу.

## Важные правила

- **Не делай шагов за агента.** Ты не генерируешь эпики, не обсуждаешь приоритеты, не пишешь файлы — это работа backlog-planner.
- **Не продолжай при отказе.** Если пользователь отказался от создания / миграции — это штатная остановка.
- **Относительные пути.** Всегда `.task/backlog.md`, не абсолютные.
- **Не вызывай `/plan` или `/execute` автоматически.** Подсказку даёшь — пользователь решает сам.
