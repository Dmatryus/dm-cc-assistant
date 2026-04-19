---
name: research
description: Research the codebase for a backlog task by T-ID (or by text description). Produces .task/research.md with findings, relevant files, suggested approach, and a ready-to-use prompt for implementation.
disable-model-invocation: true
---

# research — исследование кодовой базы для задачи

Это skill вызывается командами:
- `/dm-cc-assistant:research T-003` — стандартный research
- `/dm-cc-assistant:research T-003 --done` — закрыть задачу (Done + commit message)

## Твоя роль как основного Claude

Ты — диспетчер. Сам research не проводишь — это работа subagent'а.

## Вводное сообщение

Если аргументы содержат `--done`:

> Запускаю закрытие задачи. Ассистент подготовит commit message и обновит статус в backlog.

Иначе:

> Запускаю research. Ассистент изучит документацию и кодовую базу, найдёт релевантные файлы, паттерны и ограничения. В конце — готовый промпт для начала реализации в новом чате.

## Запуск агента

Запусти Task с `subagent_type: task-researcher`. В prompt передай `$ARGUMENTS` (включая флаг `--done` если есть):

Если `$ARGUMENTS` содержит `--done`:

> Работай в режиме done для задачи: $ARGUMENTS. Найди задачу в backlog, собери git контекст, предложи commit message, обнови статус задачи.

Иначе:

> Проведи research для задачи: $ARGUMENTS. Работай в текущей директории. Покажи пользователю сводку, получи подтверждение, запиши `.task/research.md`. Выведи промпт для реализации в чат.

## Проверка после агента

```bash
ls .task/research-*.md 2>/dev/null && echo OK || echo MISSING
```

Если `MISSING` — сообщи «Research не завершён (`.task/research-{T-ID}.md` не создан).»

## Итоговая подсказка

> После реализации — запусти `/dm-cc-assistant:review` для ревью.

## Важные правила

- **Не делай шагов за агента.** Ты не ищешь файлы и не анализируешь код — это работа task-researcher.
- **Не продолжай при отказе.** Если пользователь не подтвердил сводку — штатная остановка.
- **Относительные пути.** Всегда `.task/research.md`.
