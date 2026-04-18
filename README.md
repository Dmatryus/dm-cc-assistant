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

После инициализации — пять команд для ежедневной работы (плюс флаги):

| Команда | Что делает |
|---|---|
| `/dm-cc-assistant:backlog` | Генерирует план реализации из OVERVIEW + ARCHITECTURE. Задачи с T-ID, приоритетами, зависимостями, волнами параллельного выполнения. Повторный запуск — выбор задачи. |
| `/dm-cc-assistant:research T-003` | Исследует кодовую базу для задачи. Пишет `.task/research.md` с relevant files, паттернами, планом. Промпт для реализации выводится прямо в чат. |
| `/dm-cc-assistant:research T-003 --done` | Закрывает задачу: меняет статус на Done в backlog, предлагает commit message. |
| `/dm-cc-assistant:review` | Интерактивный ревью: обсуждает findings по одному. Можно принять, отложить (→ backlog) или отклонить. |
| `/dm-cc-assistant:update-docs` | Обновляет docs + backlog + open questions. Показывает правки по секциям, ждёт подтверждение каждой. |
| `/dm-cc-assistant:release` | Подготовка коммита: анализирует изменения, предлагает commit message, закрывает задачи In Progress. |
| `/dm-cc-assistant:release full` | Полный релиз: обновляет CHANGELOG.md и README.md, bump версии, предлагает git tag и release notes. |

**Типичный цикл:**

```
/dm-cc-assistant:backlog                    # выбрать задачу
/dm-cc-assistant:research T-003             # изучить контекст → промпт в чат
# открыть новый чат → вставить промпт → реализовать
/dm-cc-assistant:review                     # ревью изменений
/dm-cc-assistant:research T-003 --done      # закрыть задачу + commit message
/dm-cc-assistant:update-docs                # обновить документацию
/dm-cc-assistant:release full               # оформить релиз
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

> **Внимание**: в текущей версии Claude Code Desktop кнопка Update и команда `/plugin update` могут не работать — приложение не делает `git pull` на кешированный marketplace clone и остаётся на старой версии. Workaround — см. раздел [Troubleshooting](#troubleshooting).

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
│   ├── task-researcher.md           # v2: research задачи по T-ID + done mode
│   ├── code-reviewer.md             # v2: интерактивный ревью
│   ├── docs-updater.md              # v2: обновление docs + backlog
│   └── release-manager.md           # v2: подготовка релиза (mode / full)
├── skills/
│   ├── project-init/SKILL.md        # v1: оркестратор инициализации
│   ├── backlog/SKILL.md             # v2: управление backlog
│   ├── research/SKILL.md            # v2: research задачи + --done
│   ├── review/SKILL.md              # v2: ревью кода
│   ├── update-docs/SKILL.md         # v2: обновление документации
│   └── release/SKILL.md             # v2: подготовка релиза
├── hooks/hooks.json                 # контекстный SessionStart
├── OVERVIEW.md
├── ARCHITECTURE.md
└── CLAUDE.md
```

Подробности архитектуры — в [`ARCHITECTURE.md`](ARCHITECTURE.md).

## Troubleshooting

### Плагин не обновляется на новую версию

**Симптом**: выпустил новую версию плагина (git tag, release), но приложение по-прежнему показывает старую — кнопка Update серая или `/plugin update` не тянет изменения.

**Причина**: Claude Code Desktop не делает `git pull` на кешированный marketplace clone — он остаётся на snapshot'е от первоначального install.

**Решение** — force reinstall:

1. Закрой Claude Code Desktop.

2. Удали **оба** каталога (Windows, Git Bash):
   ```bash
   rm -rf "$HOME/.claude/plugins/marketplaces/<marketplace-name>"
   rm -rf "$HOME/.claude/plugins/cache/<marketplace-name>"
   ```
   Для `dm-cc-assistant`: `<marketplace-name>` = `dm-cc`.

3. Обнули `installed_plugins.json` — замени содержимое на:
   ```json
   {
     "version": 2,
     "plugins": {}
   }
   ```
   Файл: `~/.claude/plugins/installed_plugins.json`.

4. Запусти Claude Code Desktop. При старте сессии приложение пересклонирует marketplace из GitHub и переустановит плагин с актуальной версией.

**Важно**: удалять нужно **оба** каталога. Если удалить только `cache/`, marketplace-клон не обновляется и плагин переустановится из старого snapshot'а.

**Верификация**: после рестарта в `installed_plugins.json` должен появиться новый `gitCommitSha` и `version`. В UI плагина (`+` → Plugins → dm-cc-assistant) должно быть актуальное число skills и agents.

## Лицензия

Apache License 2.0. См. [`LICENSE`](LICENSE).
