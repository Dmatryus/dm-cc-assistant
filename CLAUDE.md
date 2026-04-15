# dm-cc-assistant

## WHY

Плагин для Claude Code который сопровождает весь цикл разработки: от старта проекта до ежедневной работы. v1 покрывает только старт нового проекта с нуля.

## WHAT

```
dm-cc-assistant/
├── .claude-plugin/plugin.json       # манифест плагина
├── agents/                          # агенты — каждый отвечает за один шаг
│   ├── overview-interviewer.md
│   ├── architecture-interviewer.md
│   ├── claude-md-generator.md
│   └── project-scaffolder.md
├── skills/project-init/SKILL.md    # оркестратор
└── hooks/hooks.json                 # базовые hooks
```

Агенты общаются через файлы. Каждый читает результат предыдущего из файловой системы.

## HOW

**Тестирование плагина:**
```bash
claude --plugin-dir ./dm-cc-assistant
/dm-cc-assistant:project-init
```

**Перезагрузка после изменений:**
```
/reload-plugins
```

**Проверка агентов:**
```
/agents
```

## CONSTRAINTS

- Агенты в плагине не могут использовать `hooks`, `mcpServers`, `permissionMode` в frontmatter — ограничение Claude Code
- Skills получают namespace: `/dm-cc-assistant:*` — нельзя убрать
- Subagents не могут порождать других subagents
- Все генерируемые файлы пишутся в текущую директорию проекта пользователя

## NEVER

- Не добавлять внешние зависимости — плагин должен работать из коробки
- Не писать код на Python/JS — только Markdown, YAML, Bash
- Не генерировать код проекта пользователя — только Claude окружение
