# dm-cc-assistant

Плагин для [Claude Code](https://docs.claude.com/en/docs/claude-code), который сопровождает весь цикл разработки проекта — от старта с нуля до ежедневной работы.

## Что умеет

### Инициализация проекта (v0.1)

Команда `/dm-cc-assistant:project-init` запускает цепочку из четырёх агентов:

1. **overview-interviewer** — продуктовое интервью → `OVERVIEW.md`
2. **architecture-interviewer** — технические решения с альтернативами → `ARCHITECTURE.md`
3. **claude-md-generator** — синтез `CLAUDE.md` с принципами разработки
4. **project-scaffolder** — скаффолд `.claude/` для KMP-проектов

### Цикл разработки (v0.2)

После инициализации — четыре команды для ежедневной работы:

| Команда | Что делает |
|---|---|
| `/dm-cc-assistant:backlog` | Генерирует план реализации из OVERVIEW + ARCHITECTURE. Задачи с T-ID, приоритетами, зависимостями. Повторный запуск — выбор задачи. |
| `/dm-cc-assistant:research T-003` | Исследует кодовую базу для задачи. Пишет `.task/research.md` с relevant files, паттернами, планом. В конце — готовый промпт для нового чата. |
| `/dm-cc-assistant:review` | Интерактивный ревью: обсуждает findings по одному. Можно принять, отложить (→ backlog) или отклонить. |
| `/dm-cc-assistant:update-docs` | Обновляет docs + backlog + open questions. Показывает правки по секциям, ждёт подтверждение каждой. |

**Типичный цикл:**

```
/dm-cc-assistant:backlog              # выбрать задачу
/dm-cc-assistant:research T-003       # изучить контекст
# скопировать промпт из research.md в новый чат → реализовать
/dm-cc-assistant:review               # ревью изменений
/dm-cc-assistant:update-docs          # обновить документацию
```

## Установка

Плагин распространяется через встроенный маркетплейс Claude Code:

```
/plugin marketplace add Dmatryus/dm-cc-assistant
/plugin install dm-cc-assistant@dm-cc
```

Или через десктоп-приложение: `+` → Plugins → Add plugin → найти `dm-cc-assistant`.

После установки перезапусти Claude Code.

Обновление:

```
/plugin marketplace update dm-cc
/plugin update dm-cc-assistant@dm-cc
```

## Требования

- [Claude Code](https://docs.claude.com/en/docs/claude-code) — установленный и настроенный.
- Git — для `/review` и `/update-docs` (определение изменений через diff/log).
- Git Bash или Unix-подобный shell (хуки используют bash).

## Ограничения

- **Скаффолдинг только для KMP** — для других типов проектов шаг `project-scaffolder` пропускается.
- **Документация на русском** — интервью, backlog и генерируемые документы на русском языке.
- **Только новые проекты для init** — анализ существующих кодовых баз не поддерживается.
- **Интерактивность** — все шаги требуют диалога с пользователем, headless-режима нет.
- **Backlog в файле** — `.task/backlog.md`, не во внешней системе (Jira, GitHub Issues).

## Структура плагина

```
dm-cc-assistant/
├── .claude-plugin/
│   ├── plugin.json                  # манифест
│   └── marketplace.json             # self-hosted marketplace (dm-cc)
├── agents/
│   ├── overview-interviewer.md      # v1: интервью → OVERVIEW.md
│   ├── architecture-interviewer.md  # v1: интервью → ARCHITECTURE.md
│   ├── claude-md-generator.md       # v1: генерация CLAUDE.md
│   ├── project-scaffolder.md        # v1: скаффолд .claude/
│   ├── backlog-planner.md           # v2: генерация и управление backlog
│   ├── task-researcher.md           # v2: research задачи по T-ID
│   ├── code-reviewer.md             # v2: интерактивный ревью
│   └── docs-updater.md              # v2: обновление docs + backlog
├── skills/
│   ├── project-init/SKILL.md        # v1: оркестратор инициализации
│   ├── backlog/SKILL.md             # v2: управление backlog
│   ├── research/SKILL.md            # v2: research задачи
│   ├── review/SKILL.md              # v2: ревью кода
│   └── update-docs/SKILL.md         # v2: обновление документации
├── hooks/hooks.json                 # контекстный SessionStart
├── OVERVIEW.md
├── ARCHITECTURE.md
└── CLAUDE.md
```

Подробности архитектуры — в [`ARCHITECTURE.md`](ARCHITECTURE.md).

## Лицензия

Apache License 2.0. См. [`LICENSE`](LICENSE).
