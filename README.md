# dm-cc-assistant

Плагин для [Claude Code](https://docs.claude.com/en/docs/claude-code), который сопровождает весь цикл разработки проекта — от старта с нуля до автономного исполнения эпиков с параллельными worktree'ами и подготовки релиза.

## Что умеет

### Инициализация проекта (v0.1)

Команда `/dm-cc-assistant:project-init` запускает цепочку из четырёх агентов:

1. **overview-interviewer** — продуктовое интервью → `OVERVIEW.md`
2. **architecture-interviewer** — технические решения с альтернативами → `ARCHITECTURE.md` (требует `[Priority]` теги в §9 Tech Debt и §10 Code Hotspots)
3. **claude-md-generator** — синтез `CLAUDE.md` с принципами разработки
4. **project-scaffolder** — скаффолд `.claude/` для KMP-проектов

### Эпический цикл разработки (v0.3)

Четыре команды без аргументов и флагов. Активный эпик определяется автоматически из `## In Progress` секции `.task/backlog.md`. По правилу «один эпик за раз» — он всегда один.

| Команда | Что делает | Интерактивность |
|---|---|---|
| `/dm-cc-assistant:backlog` | Двухуровневая модель: эпики (E-001) с подзадачами (E-001.1). Sync с docs, suggestions с rejected memory, миграция v0.2→v0.3, validate-step при add-new-epic, orphan detection. | Лёгкий диалог. |
| `/dm-cc-assistant:plan` | 7-фазное планирование активного эпика: Context / Decomposition / DAG + waves / Risks / Open questions / DoD / Final review. Декомпозиция на 5–15 подзадач, Mermaid-граф, file-overlap detection в волнах. | Тяжёлый интерактив. |
| `/dm-cc-assistant:execute` | Автономный параллельный прогон волн через `execution-agent`'ов в worktree'ах. Inter-wave merge через `release-manager` (merge mode) с 3-уровневой резолюцией конфликтов. После всех волн — `code-reviewer` (in-execute) + `release-manager` (aggregation). 4-шаговый финальный диалог: narrative → меню → sub-dialogs → финальное действие. | Pre-execute confirm + автономия + финальный диалог. |
| `/dm-cc-assistant:release` | `code-reviewer` (release-readiness mode) → диалог по Critical/High → `release-manager` (full) генерирует CHANGELOG entry, release notes, announcement, migration. Bumps версии. **Локальный коммит на feat/E-XXX**. Печатает инструкции для merge / tag / push (НЕ выполняет). | Подтверждения по артефактам. |

**Типичный цикл:**

```
/dm-cc-assistant:backlog          # синк с docs, выбрать эпик (Pick from Todo)
/dm-cc-assistant:plan             # 7 фаз → .task/plan-E-XXX.md
/dm-cc-assistant:execute          # автономный прогон + финальный диалог
/dm-cc-assistant:release          # подготовка релиз-материалов на feat/E-XXX
# дальше — твои руки: merge → tag → push → GitHub Release
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

## Миграция с v0.2

Старые команды **удалены (Breaking)**:

| v0.2 | v0.3 |
|---|---|
| `/dm-cc-assistant:research T-ID` | Поглощено `/plan` (research-подзадачи как часть плана) |
| `/dm-cc-assistant:review` | Поглощено `/execute` (in-execute review после волн) и `/release` (release-readiness) |
| `/dm-cc-assistant:update-docs` | `docs-updater` теперь работает внутри других команд (backlog sync), не как отдельный command |
| `/dm-cc-assistant:release full` | Нет подкоманд — `/release` делает всё сразу |

Backlog v0.2 (плоский T-ID формат) → v0.3 двухуровневая модель: при первом запуске `/backlog` агент детектит формат и предлагает 4 опции:
1. **Migrate (default)** — Done T-IDs группируются в архивный эпик `E-000`, активные становятся подзадачами выбранного эпика.
2. **Wipe** — бэкап в `.task/backlog.v02.md`, генерация эпиков с нуля из docs.
3. **Keep-legacy** — T-IDs в секции `## Legacy`, новые эпики в стандартных секциях.
4. **Abort** — оставить как есть, не запускать `/backlog`.

Подробности — в `.task/migration-v0.3.0.md` (генерируется в `/release` при upgrade'е твоего проекта).

## Требования

- [Claude Code](https://docs.claude.com/en/docs/claude-code) с поддержкой Task tool.
- Git — обязателен (worktree'ы для `/execute`, diff для `/release` review).
- Git Bash или Unix-подобный shell (хуки используют bash).

## Ограничения

- **Скаффолдинг только для KMP** — для других типов проектов шаг `project-scaffolder` пропускается.
- **Документация на русском** — интервью, backlog, план, отчёты на русском.
- **Только новые проекты для init** — анализ существующих кодовых баз не поддерживается (есть в backlog как Should-фича).
- **Один эпик In Progress за раз** — линейный pipeline между эпиками. Параллельность только внутри эпика.
- **`/release` не пушит, не тэгирует, не мёрджит в main** — готовит локально, печатает инструкции.
- **Backlog в файле** — `.task/backlog.md`, не во внешней системе.

## Структура плагина

```
dm-cc-assistant/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json             # self-hosted marketplace (dm-cc)
├── agents/
│   ├── overview-interviewer.md      # project-init
│   ├── architecture-interviewer.md  # project-init (требует [Priority] в §9/§10)
│   ├── claude-md-generator.md       # project-init
│   ├── project-scaffolder.md        # project-init
│   ├── backlog-planner.md           # /backlog: эпики + миграция v0.2→v0.3 + sync
│   ├── epic-planner.md              # /plan: 7-фазное планирование
│   ├── execution-agent.md           # /execute: автономная работа в worktree
│   ├── resume-analyzer.md           # /execute: анализ прерванной подзадачи
│   ├── code-reviewer.md             # /execute (in-execute) + /release (release-readiness)
│   ├── docs-updater.md              # внутренний: targeted edits docs с [Priority]
│   └── release-manager.md           # /execute (merge / aggregation) + /release (full)
├── skills/
│   ├── project-init/SKILL.md
│   ├── backlog/SKILL.md
│   ├── plan/SKILL.md
│   ├── execute/SKILL.md
│   └── release/SKILL.md
├── hooks/hooks.json                 # SessionStart с двухуровневой моделью + .execute-active marker
├── OVERVIEW.md
├── ARCHITECTURE.md
├── CLAUDE.md
├── CHANGELOG.md
└── README.md
```

Подробности архитектуры — в [`ARCHITECTURE.md`](ARCHITECTURE.md). Дизайн v0.3 — в [`.task/v0.3-design.md`](.task/v0.3-design.md).

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
