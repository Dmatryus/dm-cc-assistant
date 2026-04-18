# Changelog

Все заметные изменения проекта документируются в этом файле.

Формат — [Keep a Changelog](https://keepachangelog.com/ru/1.1.0/), версионирование — [SemVer](https://semver.org/lang/ru/).

## [0.2.1] — 2026-04-18

### Added
- **`/dm-cc-assistant:release`** — подготовка коммита: анализирует изменения, предлагает commit message, закрывает задачи In Progress в backlog.
- **`/dm-cc-assistant:release full`** — полный релиз: обновляет CHANGELOG.md и README.md, делает bump версии, предлагает git tag и генерирует release notes.
- Агент `release-manager` (model: sonnet) — два режима (mode / full), не делает git commit/tag сам — только готовит материалы.
- Режим `--done` для `/dm-cc-assistant:research T-ID --done` — закрывает задачу в backlog и предлагает commit message.

### Fixed
- `task-researcher`: промпт для реализации теперь выводится прямо в чат после записи research.md (раньше только в файл).
- `task-researcher`: результат research виден в чате — краткое резюме findings + промпт.
- `backlog-planner`: план строится от фундамента (архитектура → инфраструктура → фичи), а не копирует Must-список из OVERVIEW.
- `backlog-planner`: добавлен шаг анализа реального кода перед генерацией задач.

### Changed
- `backlog-planner`: новый шаг «Plan parallel execution» — группирует задачи в волны (Wave 1, 2, ...) с предложением имён веток.
- `research` skill: поддержка флага `--done`.
- SessionStart hook: счётчик задач исправлен (было total, стало todo), добавлена подсказка `:release`.
- CLAUDE.md: обновлён список команд и файловая карта.

## [0.2.0] — 2026-04-16

Backlog + Daily Task Cycle. Плагин теперь покрывает полный цикл разработки, а не только инициализацию.

### Added
- **`/dm-cc-assistant:backlog`** — генерирует план реализации из OVERVIEW.md + ARCHITECTURE.md. Задачи с T-ID, приоритетами (Critical/High/Medium/Low), зависимостями и итерациями. Повторные запуски показывают статус и позволяют выбрать задачу.
- **`/dm-cc-assistant:research T-ID`** — исследует кодовую базу и документацию для конкретной задачи из backlog. Пишет `.task/research.md` с relevant files, existing patterns, constraints, suggested approach. В конце — готовый промпт для копирования в новый чат.
- **`/dm-cc-assistant:review [scope]`** — интерактивный ревью кода. Обсуждает findings по одному с приоритизацией. Пользователь может принять (fix now), отложить (добавляется в backlog с новым T-ID) или отклонить каждый finding.
- **`/dm-cc-assistant:update-docs [описание]`** — обновляет три слоя: (1) project docs (OVERVIEW, ARCHITECTURE, CLAUDE.md) — targeted edits по секциям с превью, (2) backlog — задача → Done, новые задачи, (3) open questions — сессионные → docs/backlog/глобальные, глобальные → проверка решённости.
- Агент `backlog-planner` (model: opus) — генерация и управление backlog.
- Агент `task-researcher` (model: sonnet) — research задачи по T-ID.
- Агент `code-reviewer` (model: sonnet) — интерактивный ревью.
- Агент `docs-updater` (model: sonnet) — обновление docs + backlog + open questions.
- `.task/` directory — рабочая директория для backlog.md, research.md, review.md.
- Контекстный SessionStart hook — определяет состояние проекта и даёт релевантные подсказки.

### Changed
- OVERVIEW.md — обновлён скоуп: backlog/research/review/docs-update перешли из Could в Must (v2).
- ARCHITECTURE.md — Module Map, Data Flow (sequenceDiagram для task cycle), Data Model (`.task/` artifacts).
- CLAUDE.md — новые команды в WHAT/HOW.
- SessionStart hook — вместо одной строки теперь context-aware с проверкой CLAUDE.md, backlog.md, research.md.
- plugin.json / marketplace.json — description, keywords, version 0.2.0.

## [0.1.1] — 2026-04-16

Фиксы по результатам первого live-теста на реальном проекте (7 багов).

### Fixed
- **(CRITICAL)** Агенты пропускали превью и подтверждение — добавлены жёсткие STOP-gates после каждого раздела и блок NEVER в обоих интервьюерах. Оркестратор теперь проверяет не только наличие файла, но и отчёт агента о подтверждении.
- Mermaid-диаграммы теперь включаются по умолчанию (`sequenceDiagram` для линейных сценариев, `flowchart` для ветвящихся), а не предлагаются как opt-in.
- Слабое использование контекста — добавлен «Шаг 0.5 — извлечение контекста» в обоих интервьюерах; оркестратор передаёт `$ARGUMENTS` пользователя в агент; добавлено правило пропорциональности draft'ов.
- MoSCoW-раздел разбит на 4 подшага (Must → Should → Could → Won't), каждый с отдельным draft → STOP → confirm.
- Non-goals vs Won't — добавлено явное разграничение: Won't = «не сейчас, но возможно позже», Non-goals = «в принципе не наша задача, никогда». Агент обязан исключать дубликаты.
- Открытые вопросы — вместо одноразовой генерации теперь итеративный цикл ревью документа: найти пробелы → часть обогащает разделы, часть идёт в открытые вопросы → повторять до «хватит».
- Жаргон — расширен список терминов в обоих интервьюерах, добавлены негативные примеры, исправлен собственный жаргон в промпте architecture-interviewer («DI-контейнер» теперь с расшифровкой), добавлены напоминания в тяжёлых разделах.

### Added
- 5-й принцип **One Question at a Time** в шаблоне генерируемого CLAUDE.md — вопросы пользователю строго по одному.
- Блок `## NEVER` в обоих интервьюерах (5 запретов каждый).
- `## Финальная проверка согласованности` в architecture-interviewer — cross-check Data Model vs Must-фичи, Stack vs Constraints, Module Map vs Data Flow перед сборкой превью.
- Передача `$ARGUMENTS` из оркестратора в overview-interviewer — если пользователь описал проект при вызове skill'а, агент не переспрашивает.

## [0.1.0] — 2026-04-15

Первый релиз. Покрывает сценарий старта нового проекта с нуля.

### Added
- Манифест плагина `.claude-plugin/plugin.json` с метаданными (license `Apache-2.0`, `repository`, `homepage`, `keywords`).
- Self-hosted marketplace `.claude-plugin/marketplace.json` (имя `dm-cc`) — тот же репозиторий выступает и каталогом, и плагином. Установка: `/plugin marketplace add Dmatryus/dm-cc-assistant@v0.1.0` → `/plugin install dm-cc-assistant@dm-cc`.
- Skill-оркестратор `/dm-cc-assistant:project-init` — запускает четыре шага последовательно и проверяет появление файлов после каждого.
- Агент `overview-interviewer` — проводит продуктовое интервью в стиле «draft → подтверждение» и создаёт `OVERVIEW.md` (9 разделов, MoSCoW-скоуп, non-goals, mermaid user flow).
- Агент `architecture-interviewer` — обсуждает технические выборы по одному (Stack, Module Map, Data Model и др.) с альтернативами, плюсами-минусами и рекомендацией; создаёт `ARCHITECTURE.md` (10 разделов) и фиксирует тип проекта для следующего шага.
- Агент `claude-md-generator` — синтезирует `CLAUDE.md` из первых двух документов по шаблону WHY / WHAT / HOW / CONSTRAINTS / NEVER / PRINCIPLES; раздел PRINCIPLES с четырьмя принципами Карпатого неизменяем.
- Агент `project-scaffolder` — для KMP-проектов создаёт базовый `.claude/` scaffold (skill `kmp-build`, агент `kmp-reviewer`, hook `ktlintFormat`); для не-KMP завершает работу штатно.
- Hook `SessionStart` плагина — приветствие с напоминанием о `/dm-cc-assistant:project-init`.
- Документация: `README.md` (на русском, с установкой и ограничениями v0.1.0), `OVERVIEW.md`, `ARCHITECTURE.md`, `CLAUDE.md`, `docs/claude-workflow-guide.md`, `LICENSE` (Apache-2.0).

### Known limitations
- Скаффолдинг поддерживается только для KMP-проектов; для остальных типов шаг штатно пропускается.
- Все интервью и генерируемые документы — на русском языке.
- Анализ существующих кодовых баз не поддерживается — только пустые директории.
- Все четыре шага интерактивны, headless-режима нет.
- Хук `ktlintFormat` в KMP-скаффолде делает полнопроектное форматирование (у `ktlint-gradle` нет штатного per-file флага); при проблемах со скоростью хук можно убрать.
