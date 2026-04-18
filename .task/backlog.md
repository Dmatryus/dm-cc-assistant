# Backlog — dm-cc-assistant

Последнее обновление: 2026-04-18

## In Progress

_(пусто)_

## Todo

- **T-004** [Low] Поддержка существующих проектов
  - Зависит от: —
  - Контекст: режим `/project-init --existing` — читает кодовую базу, предзаполняет OVERVIEW/ARCHITECTURE, интервью работает как валидация

## Done

- **T-001** [Critical] Фиксы агентов task-researcher и backlog-planner ✅ 2026-04-18
  - Контекст: task-researcher выводит промпт и результат в чат; backlog-planner строит план от фундамента

- **T-002** [High] Параллельная разработка и done-режим ✅ 2026-04-18
  - Контекст: backlog-planner строит план волн (parallel execution); task-researcher получает режим `--done`

- **T-003** [Critical] Агент и skill release-manager ✅ 2026-04-18
  - Контекст: agents/release-manager.md (mode / full), skills/release/SKILL.md, CLAUDE.md обновлён

## Open Questions

- Минимальный набор skills/rules/hooks для KMP проекта — пока не закрыт (v1).
- Нужна ли история `.task/research.md` и `.task/review.md` или достаточно последнего?
- Как docs-updater определяет границы «сессионных» открытых вопросов vs глобальных?
- **Резолв (2026-04-18)**: режим завершения задачи — расширение `research` skill. task-researcher получает режим "done". См. T-002.
