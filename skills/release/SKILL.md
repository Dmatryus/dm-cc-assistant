---
name: release
description: Prepares a release. Default mode: proposes commit message and updates task status in backlog. Pass "full" for a complete release — updates README/CHANGELOG, bumps version, proposes git tag, generates release notes.
disable-model-invocation: true
---

# release — подготовка релиза

Это skill вызывается командой `/dm-cc-assistant:release` (mode) или `/dm-cc-assistant:release full`.

## Твоя роль как основного Claude

Ты — диспетчер. Сам не анализируешь git и не редактируешь файлы — это работа subagent'а.

## Вводное сообщение

Если аргумент `full`:

> Запускаю полный релиз. Ассистент подготовит commit message, обновит CHANGELOG.md и README.md, сделает bump версии, предложит git tag и release notes.

Иначе:

> Запускаю подготовку коммита. Ассистент проанализирует изменения, предложит commit message и обновит статус задач в backlog.

## Запуск агента

Запусти Task с `subagent_type: release-manager`. В prompt передай режим:

Если аргумент `full`:

> Подготовь полный релиз (режим full). Работай в текущей директории. Собери git контекст, предложи commit message, обнови CHANGELOG.md и README.md, сделай bump версии, выведи итоговый чеклист с командами для git tag.

Иначе:

> Подготовь коммит (режим mode, дефолт). Работай в текущей директории. Собери git контекст, предложи commit message, обнови статус задач в backlog если есть In Progress.

## Важные правила

- **Не делай шагов за агента.** Ты не анализируешь diff и не редактируешь CHANGELOG — это работа release-manager.
- **Не продолжай при отказе.** Если пользователь прервал процесс — штатная остановка.
- **Относительные пути.** Всегда `.task/backlog.md`, `CHANGELOG.md`, не абсолютные.
