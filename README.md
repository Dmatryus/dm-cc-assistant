# dm-cc-assistant

Плагин для [Claude Code](https://docs.claude.com/en/docs/claude-code), который сопровождает весь цикл разработки проекта — от старта с нуля до ежедневной работы. **v0.1.0 покрывает только старт нового проекта.**

## Что умеет v0.1.0

Команда `/dm-cc-assistant:project-init` запускает цепочку из четырёх агентов:

1. **overview-interviewer** — проводит продуктовое интервью и создаёт `OVERVIEW.md` (цель проекта, пользователи, скоуп, non-goals).
2. **architecture-interviewer** — обсуждает технические решения по одному с альтернативами и рекомендациями, создаёт `ARCHITECTURE.md`.
3. **claude-md-generator** — синтезирует `CLAUDE.md` из первых двух документов, добавляет четыре принципа Карпатого.
4. **project-scaffolder** — создаёт базовый скаффолд `.claude/` (skills, agents, hooks) для KMP-проектов.

Каждый агент показывает превью документа и ждёт подтверждения. Все файлы создаются в текущей директории.

## Установка

Плагин распространяется через встроенный маркетплейс Claude Code. В любой сессии выполни:

```
/plugin marketplace add Dmatryus/dm-cc-assistant@v0.1.0
/plugin install dm-cc-assistant@dm-cc
```

Первая команда добавляет каталог `dm-cc`, запиненный на тег `v0.1.0`. Вторая — устанавливает плагин из этого каталога. После установки перезапусти Claude Code, чтобы подхватить skill и агентов.

Обновление на следующую версию (после её выхода):

```
/plugin marketplace update dm-cc
/plugin update dm-cc-assistant@dm-cc
```

## Использование

В пустой директории нового проекта:

```
/dm-cc-assistant:project-init
```

Пройди четыре шага интервью. На каждом шаге увидишь превью и сможешь подтвердить или отказаться.

После завершения в директории появятся:

- `OVERVIEW.md` — продуктовое описание
- `ARCHITECTURE.md` — техническое описание
- `CLAUDE.md` — инструкции для Claude с принципами разработки
- `.claude/` — scaffold skills/agents/hooks (только для KMP)

## Требования

- [Claude Code](https://docs.claude.com/en/docs/claude-code) — установленный и настроенный.
- Git Bash или Unix-подобный shell (хуки используют bash).

## Ограничения v0.1.0

- **Скаффолдинг только для KMP** — для других типов проектов (Python, Android native, etc.) шаг `project-scaffolder` пропускается с сообщением.
- **Документация на русском** — интервью и генерируемые документы на русском языке.
- **Только новые проекты** — анализ существующих кодовых баз не поддерживается в v0.1.0.
- **Интерактивность** — все четыре шага требуют диалога с пользователем, headless-режима нет.

## Что дальше

Полный цикл задачи (task-researcher, code-reviewer, docs-updater), поддержка существующих проектов и скаффолдинг для других типов — в дорожной карте. См. [`OVERVIEW.md`](OVERVIEW.md) §4 и §6 для деталей.

## Структура плагина

```
dm-cc-assistant/
├── .claude-plugin/plugin.json       # манифест плагина
├── agents/
│   ├── overview-interviewer.md
│   ├── architecture-interviewer.md
│   ├── claude-md-generator.md
│   └── project-scaffolder.md
├── skills/
│   └── project-init/SKILL.md        # оркестратор
├── hooks/hooks.json                 # SessionStart приветствие
├── docs/
│   └── claude-workflow-guide.md     # референс по Claude Code
├── OVERVIEW.md                      # продуктовая спецификация
├── ARCHITECTURE.md                  # техническая спецификация
└── CLAUDE.md                        # инструкции для разработки плагина
```

Подробности архитектуры — в [`ARCHITECTURE.md`](ARCHITECTURE.md).

## Лицензия

Apache License 2.0. См. [`LICENSE`](LICENSE).
