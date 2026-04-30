# Backlog — dm-cc-assistant

Последнее обновление: 2026-04-30

## In Progress

_(пусто)_

## Todo

- **T-004** [Low] Поддержка существующих проектов
  - Зависит от: —
  - Контекст: режим `/project-init --existing` — читает кодовую базу, предзаполняет OVERVIEW/ARCHITECTURE, интервью работает как валидация. Вне v0.3 scope.

## Done

- **T-014** [Critical] Release v0.3.0 — plugin.json, marketplace, CHANGELOG, migration guide ✅ 2026-04-30 (prep)
  - Контекст: bump в `.claude-plugin/plugin.json` и `marketplace.json` до 0.3.0, sync description/keywords. CHANGELOG entry `[0.3.0]` с Added / Changed / Removed (Breaking) / Migration. `.task/migration-v0.3.0.md`, `.task/release-notes-v0.3.0.md`, `.task/announcement-v0.3.0.md`. Подготовка через v0.2.1 release-skill (кура-яйцо §12.6). Финальный merge / tag / push — руки пользователя.

- **T-013** [Critical] Documentation — OVERVIEW, ARCHITECTURE, CLAUDE.md, README ✅ 2026-04-30
  - Контекст: README раздел «Что умеет / Структура / Цикл / migration note», OVERVIEW §4–§9, ARCHITECTURE §1–§10 большие правки, CLAUDE.md WHAT/HOW/structure/CONSTRAINTS.

- **T-012** [Medium] Cleanup — удаление старых skills и task-researcher ✅ 2026-04-30
  - Контекст: удалены skills/research, skills/review, skills/update-docs, agents/task-researcher.md.

- **T-011** [High] Hooks SessionStart — переписать под двухуровневую модель ✅ 2026-04-30
  - Контекст: парсинг эпиков/подзадач/волн через awk/grep, edge cases (пустой backlog, нет CLAUDE.md, v0.2 формат, .execute-active marker).

- **T-010** [High] `/release` — новый flow без push ✅ 2026-04-30
  - Контекст: skills/release/SKILL.md оркестрирует code-reviewer (release-readiness) → диалог по Critical/High → release-manager (full). Без push/tag/merge.

- **T-009** [Critical] `/execute` — orchestrator + execution-agent + resume-analyzer + final dialog ✅ 2026-04-30
  - Контекст: agents/execution-agent.md (autonomy contract, smoke check, obstacle handling), agents/resume-analyzer.md (resume context), skills/execute/SKILL.md (Task tool orchestration, pre-execute analysis, inter-wave loop, 4-step final dialog), `.execute-active` marker.

- **T-008** [High] code-reviewer — два режима (in-execute / release-readiness) ✅ 2026-04-30
  - Контекст: agents/code-reviewer.md переписан — два mode'а с разным фокусом, не интерактивный (пишет findings в файл).

- **T-007** [Critical] release-manager — три режима (merge / aggregation / full) ✅ 2026-04-30
  - Контекст: agents/release-manager.md переписан под три mode — merge между волнами, aggregation после волн, full в /release без push/tag/merge. 3-уровневая резолюция конфликтов, auto-archive >5 Done.

- **T-006** [Critical] `/plan` — новый skill + epic-planner agent ✅ 2026-04-30
  - Контекст: agents/epic-planner.md (7 фаз диалога, 5-слойный light code scan, research subtask contract), skills/plan/SKILL.md.

- **T-005** [Critical] `/backlog` — двухуровневая модель + миграция v0.2→v0.3 ✅ 2026-04-30
  - Контекст: agents/backlog-planner.md переписан под epic/subtask, добавлены sync check, suggestions с rejected memory, validate-step при add-new-epic, миграция плоского T-ID формата (4 опции), orphan detection. skills/backlog/SKILL.md обновлён.

- **T-conv** [High] Convention updates — edit log + `[Priority]` в ARCHITECTURE §9/§10 ✅ 2026-04-30
  - Контекст: коммит 4e521cf — обновлены architecture-interviewer, docs-updater, backlog-planner под новые конвенции v0.3.

- **T-001** [Critical] Фиксы агентов task-researcher и backlog-planner ✅ 2026-04-18
  - Контекст: task-researcher выводит промпт и результат в чат; backlog-planner строит план от фундамента.

- **T-002** [High] Параллельная разработка и done-режим ✅ 2026-04-18
  - Контекст: backlog-planner строит план волн (parallel execution); task-researcher получает режим `--done`.

- **T-003** [Critical] Агент и skill release-manager ✅ 2026-04-18
  - Контекст: agents/release-manager.md (mode / full), skills/release/SKILL.md, CLAUDE.md обновлён.

## Open Questions

- Минимальный набор skills/rules/hooks для KMP проекта — пока не закрыт (v0.1).
- Нужна ли история `.task/plan-E-*.md`, `.task/report-E-*.md`, `.task/research-E-*.md` или достаточно текущего? (v0.3)
- Auto-archive при `## Done` > 5 захардкожен — нужен ли конфигурируемый порог?
- Resume-analyzer всегда сохраняет worktree — стоит ли иметь auto-cleanup после N дней stale?
- **Резолв (2026-04-18)**: режим завершения задачи — расширение `research` skill. task-researcher получает режим "done". См. T-002. *(теперь в v0.3 это не актуально — закрытие подзадач автоматически в /execute)*
- **Резолв (2026-04-30)**: формат backlog'а — двухуровневая модель эпик/подзадача с inline-статусами. См. T-005.
