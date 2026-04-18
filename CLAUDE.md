# dm-cc-assistant

## WHY

Плагин для Claude Code который сопровождает весь цикл разработки: от старта проекта до ежедневной работы. v1 покрывает только старт нового проекта с нуля.

## WHAT

```
dm-cc-assistant/
├── .claude-plugin/
│   ├── plugin.json                  # манифест плагина
│   └── marketplace.json             # self-hosted marketplace (dm-cc)
├── agents/
│   ├── overview-interviewer.md      # v1: интервью для OVERVIEW.md
│   ├── architecture-interviewer.md  # v1: интервью для ARCHITECTURE.md
│   ├── claude-md-generator.md       # v1: генерация CLAUDE.md
│   ├── project-scaffolder.md        # v1: скаффолдинг .claude/
│   ├── backlog-planner.md           # v2: генерация и управление backlog
│   ├── task-researcher.md           # v2: research задачи по T-ID
│   ├── code-reviewer.md             # v2: интерактивный ревью кода
│   ├── docs-updater.md              # v2: обновление docs + backlog + open questions
│   └── release-manager.md           # v2: подготовка релиза (mode / full)
├── skills/
│   ├── project-init/SKILL.md        # v1: оркестратор инициализации
│   ├── backlog/SKILL.md             # v2: создание / управление backlog
│   ├── research/SKILL.md            # v2: research задачи
│   ├── review/SKILL.md              # v2: ревью изменений
│   ├── update-docs/SKILL.md         # v2: обновление документации
│   └── release/SKILL.md             # v2: подготовка релиза
└── hooks/hooks.json                 # SessionStart + контекстные подсказки
```

v1 агенты общаются через pipeline (файл → следующий агент). v2 агенты независимы — каждый читает из ground truth (git, docs, codebase). `.task/backlog.md` — единственный shared state.

## HOW

**Команды плагина:**
```
/dm-cc-assistant:project-init        # v1: инициализация нового проекта
/dm-cc-assistant:backlog             # v2: создать / показать / управлять backlog
/dm-cc-assistant:research T-003      # v2: research задачи по ID
/dm-cc-assistant:review              # v2: ревью текущих изменений
/dm-cc-assistant:update-docs         # v2: обновить docs + backlog + open questions
/dm-cc-assistant:release             # v2: подготовить коммит + закрыть задачи
/dm-cc-assistant:release full        # v2: полный релиз (CHANGELOG, версия, tag, release notes)
```

**Тестирование плагина (dev):**
- Через приложение: `+` → Plugins → Add plugin → dm-cc
- Через CLI: `claude --plugin-dir ./dm-cc-assistant`
- Перезагрузка: `/reload-plugins`
- Проверка агентов: `/agents`

## CONSTRAINTS

- Агенты в плагине не могут использовать `hooks`, `mcpServers`, `permissionMode` в frontmatter — ограничение Claude Code
- Skills получают namespace: `/dm-cc-assistant:*` — нельзя убрать
- Subagents не могут порождать других subagents
- Все генерируемые файлы пишутся в текущую директорию проекта пользователя

## NEVER

- Не добавлять внешние зависимости — плагин должен работать из коробки
- Не писать код на Python/JS — только Markdown, YAML, Bash
- Не генерировать код проекта пользователя — только Claude окружение
