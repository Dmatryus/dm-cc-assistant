---
name: review
description: Interactively review code changes against project conventions. Discusses findings one by one, can add new tasks to backlog. Use after implementing a feature or fix.
disable-model-invocation: true
---

# review — интерактивный ревью кода

Это skill вызывается командой `/dm-cc-assistant:review` (или `/dm-cc-assistant:review HEAD~3`, `/dm-cc-assistant:review --staged`). Запускает `code-reviewer` через Task tool.

## Твоя роль как основного Claude

Ты — диспетчер. Сам ревью не проводишь — это работа subagent'а.

## Вводное сообщение

> Запускаю ревью. Ассистент проанализирует изменения, найдёт проблемы и обсудит каждую с тобой. Замечания можно принять (исправить сейчас), отложить (добавить в backlog) или отклонить.

## Запуск агента

Определи scope из `$ARGUMENTS`:
- Если пуст — default scope: `git diff` (unstaged changes)
- Если есть — передай как есть (пользователь знает что хочет: `--staged`, `HEAD~3`, `main..HEAD`)

Запусти Task с `subagent_type: code-reviewer`. Prompt:

> Проведи интерактивный ревью кода. Scope: {$ARGUMENTS или 'unstaged changes'}. Работай в текущей директории. Обсуди каждый finding с пользователем по одному. Запиши итог в `.task/review.md`. Верни отчёт с количеством accepted/backlog/dismissed.

## Проверка после агента

```bash
test -f .task/review.md && echo OK || echo MISSING
```

Если `MISSING` — возможно diff был пуст или пользователь прервал процесс. Не считай ошибкой.

## Итоговая подсказка

> Ревью завершён: `.task/review.md`
>
> Следующий шаг: `/dm-cc-assistant:update-docs` для обновления документации и backlog.

## Важные правила

- **Не делай шагов за агента.** Ты не анализируешь diff и не обсуждаешь findings.
- **Не продолжай при отказе.** Пустой diff = штатная остановка.
- **Относительные пути.**
