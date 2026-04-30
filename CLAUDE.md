# dm-cc-assistant

## WHY

Плагин для Claude Code, который сопровождает весь цикл разработки: от старта проекта до автономного исполнения эпиков. v0.3 переходит на двухуровневую модель «эпики + подзадачи» с автономным параллельным исполнением через worktree'ы.

## WHAT

```
dm-cc-assistant/
├── .claude-plugin/
│   ├── plugin.json                  # манифест плагина
│   └── marketplace.json             # self-hosted marketplace (dm-cc)
├── agents/
│   ├── overview-interviewer.md      # project-init: интервью для OVERVIEW.md
│   ├── architecture-interviewer.md  # project-init: интервью для ARCHITECTURE.md (требует [Priority] в §9/§10)
│   ├── claude-md-generator.md       # project-init: генерация CLAUDE.md
│   ├── project-scaffolder.md        # project-init: скаффолдинг .claude/
│   ├── backlog-planner.md           # /backlog: эпики, sync, suggestions, миграция v0.2→v0.3
│   ├── epic-planner.md              # /plan: 7-фазное планирование одного эпика
│   ├── execution-agent.md           # /execute: автономная работа в worktree (одна подзадача)
│   ├── resume-analyzer.md           # /execute: анализ прерванной подзадачи
│   ├── code-reviewer.md             # /execute (in-execute) и /release (release-readiness)
│   ├── docs-updater.md              # внутренний: targeted edits с [Priority] поддержкой
│   └── release-manager.md           # /execute (merge / aggregation) + /release (full)
├── skills/
│   ├── project-init/SKILL.md        # оркестратор инициализации
│   ├── backlog/SKILL.md             # оркестратор /backlog
│   ├── plan/SKILL.md                # оркестратор /plan
│   ├── execute/SKILL.md             # оркестратор /execute (Task tool, волны, финальный диалог)
│   └── release/SKILL.md             # оркестратор /release
└── hooks/hooks.json                 # SessionStart с двухуровневой моделью + .execute-active marker
```

`/backlog`, `/plan`, `/execute`, `/release` — orchestrator-команды (skill = main Claude). Они спаунят internal-агентов через Task tool. Шейринг состояния — через `.task/backlog.md`, `.task/plan-E-XXX.md`, `.task/report-E-XXX.md` и связанные файлы.

## HOW

**Команды плагина (v0.3):**

```
/dm-cc-assistant:project-init        # инициализация нового проекта (без изменений с v0.1)
/dm-cc-assistant:backlog             # двухуровневая модель: эпики + подзадачи + sync с docs
/dm-cc-assistant:plan                # тяжёлое 7-фазное планирование активного эпика
/dm-cc-assistant:execute             # автономный параллельный прогон эпика + 4-шаговый финальный диалог
/dm-cc-assistant:release             # подготовка релиз-материалов + commit на feat/E-XXX (без push/tag/merge)
```

**Удалены (Breaking от v0.2):**

```
/dm-cc-assistant:research            # → поглощено /plan (research-подзадачи как часть плана)
/dm-cc-assistant:review              # → поглощено /execute (in-execute) и /release (release-readiness)
/dm-cc-assistant:update-docs         # → docs-updater работает внутри других команд
```

**Тестирование плагина (dev):**
- Через приложение: `+` → Plugins → Add plugin → dm-cc
- Через CLI: `claude --plugin-dir ./dm-cc-assistant`
- Перезагрузка: `/reload-plugins`
- Проверка агентов: `/agents`

## CONSTRAINTS

- Агенты в плагине не могут использовать `hooks`, `mcpServers`, `permissionMode` в frontmatter — ограничение Claude Code.
- Skills получают namespace: `/dm-cc-assistant:*` — нельзя убрать.
- **Subagents не могут спаунить других subagents.** Из-за этого orchestrator (skill = main Claude) сам спаунит код-ревью, release-manager и execution-agent'ов в нужном порядке.
- Все генерируемые файлы пишутся в текущую директорию проекта пользователя.
- Один эпик в `## In Progress` за раз — линейный pipeline между эпиками, параллельность только внутри эпика (волны).
- `/release` не делает `git push`, `git tag`, `git merge` в main — только локальная работа на `feat/E-XXX`.
- `.task/.execute-active` marker блокирует параллельные `/execute` и предупреждает остальные команды.

## NEVER

- Не добавлять внешние зависимости — плагин должен работать из коробки.
- Не писать код на Python/JS — только Markdown, YAML, Bash.
- Не генерировать код проекта пользователя — только Claude окружение.
- Не пушить / тэгировать / мёрджить в main из агентов или скиллов — только инструкции пользователю.
- Не пропускать pre-execute confirmation в `/execute` — автономия начинается только после явного «go».
