---
name: research
description: Research the codebase for a backlog task by T-ID (or by text description). Produces .task/research.md with findings, relevant files, suggested approach, and a ready-to-use prompt for implementation.
disable-model-invocation: true
---

# research — исследование кодовой базы для задачи

Это skill вызывается командой `/dm-cc-assistant:research T-003` (или `/dm-cc-assistant:research описание задачи текстом`). Запускает `task-researcher` через Task tool.

## Твоя роль как основного Claude

Ты — диспетчер. Сам research не проводишь — это работа subagent'а.

## Вводное сообщение

> Запускаю research. Ассистент изучит документацию и кодовую базу, найдёт релевантные файлы, паттерны и ограничения. В конце — готовый промпт для начала реализации в новом чате.

## Запуск агента

Запусти Task с `subagent_type: task-researcher`. В prompt передай `$ARGUMENTS`:

> Проведи research для задачи: $ARGUMENTS. Работай в текущей директории. Покажи пользователю сводку, получи подтверждение, запиши `.task/research.md`. Верни короткий отчёт.

## Проверка после агента

```bash
test -f .task/research.md && echo OK || echo MISSING
```

Если `MISSING` — сообщи «Research не завершён (`.task/research.md` не создан).»

## Итоговая подсказка

> Research готов: `.task/research.md`
>
> Следующие шаги:
> 1. Скопируй промпт из конца research.md в новый чат для реализации.
> 2. После реализации — `/dm-cc-assistant:review` для ревью.

## Важные правила

- **Не делай шагов за агента.** Ты не ищешь файлы и не анализируешь код — это работа task-researcher.
- **Не продолжай при отказе.** Если пользователь не подтвердил сводку — штатная остановка.
- **Относительные пути.** Всегда `.task/research.md`.
