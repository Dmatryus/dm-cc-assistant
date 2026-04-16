---
name: backlog
description: Create or manage the project implementation backlog. First run generates prioritized tasks from OVERVIEW.md + ARCHITECTURE.md. Subsequent runs show status and let you pick a task. Use at the start of each dev session.
disable-model-invocation: true
---

# backlog — управление backlog проекта

Это skill вызывается командой `/dm-cc-assistant:backlog`. Он запускает `backlog-planner` через Task tool.

## Твоя роль как основного Claude

Ты — диспетчер. Сам backlog не генерируешь и задачи не обсуждаешь — это работа subagent'а.

## Вводное сообщение

Если `.task/backlog.md` не существует:

> Запускаю создание backlog. Ассистент проанализирует OVERVIEW.md и ARCHITECTURE.md, предложит план реализации — задачи с приоритетами, зависимостями и итерациями. Вы проработаете его вместе.

Если `.task/backlog.md` существует:

> Открываю backlog. Можно посмотреть статус, выбрать задачу, добавить новую или обновить приоритеты.

## Запуск агента

Запусти Task с `subagent_type: backlog-planner`. Prompt:

Если `.task/backlog.md` не существует:

> Прочитай `./OVERVIEW.md` и `./ARCHITECTURE.md`. Сгенерируй backlog проекта согласно твоим инструкциям. Обсуди с пользователем, получи подтверждение, запиши `.task/backlog.md`. Верни короткий отчёт со словом «подтвердил».

Если `.task/backlog.md` существует:

> Прочитай `.task/backlog.md`. Покажи пользователю статус и предложи варианты действий согласно твоим инструкциям. Верни короткий отчёт о том что было сделано.

## Проверка после агента

Если это был первый запуск:

```bash
test -f .task/backlog.md && echo OK || echo MISSING
```

Если `MISSING` и отчёт не содержит «подтвердил» — сообщи «Backlog не создан. Можешь попробовать ещё раз: `/dm-cc-assistant:backlog`».

## Итоговая подсказка

Если задача была выбрана (In Progress):

> Задача выбрана. Следующий шаг: `/dm-cc-assistant:research T-NNN`

Если backlog создан, но задача не выбрана:

> Backlog готов. Когда будешь готов начать работу — `/dm-cc-assistant:backlog` и выбери задачу.

## Важные правила

- **Не делай шагов за агента.** Ты не генерируешь задачи и не обсуждаешь приоритеты — это работа backlog-planner.
- **Не продолжай при отказе.** Если пользователь отказался от создания backlog — это штатная остановка.
- **Относительные пути.** Всегда `.task/backlog.md`, не абсолютные.
