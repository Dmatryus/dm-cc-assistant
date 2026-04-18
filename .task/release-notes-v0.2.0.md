## Backlog + Daily Task Cycle

Плагин теперь покрывает полный цикл разработки — от инициализации проекта до ежедневной работы над задачами.

### Установка / обновление

```
/plugin marketplace update dm-cc
/plugin update dm-cc-assistant@dm-cc
```

Или через `+` → Plugins → Manage plugins.

### Новые команды

| Команда | Что делает |
|---|---|
| `/dm-cc-assistant:backlog` | Генерирует план реализации из OVERVIEW + ARCHITECTURE. Задачи с T-ID, приоритетами (Critical/High/Medium/Low), зависимостями. Повторный запуск — статус и выбор задачи. |
| `/dm-cc-assistant:research T-003` | Исследует кодовую базу для задачи из backlog. Пишет `.task/research.md` с relevant files, паттернами, constraints, планом. В конце — готовый промпт для нового чата. |
| `/dm-cc-assistant:review` | Интерактивный ревью: обсуждает findings по одному с приоритизацией. Можно принять, отложить (→ backlog с новым T-ID) или отклонить. |
| `/dm-cc-assistant:update-docs` | Обновляет 3 слоя: project docs (targeted edits по секциям), backlog (задача → Done, новые задачи), open questions (сессионные → docs/backlog/глобальные). |

### Типичный цикл

```
/dm-cc-assistant:backlog              # выбрать задачу
/dm-cc-assistant:research T-003       # изучить контекст
# скопировать промпт из research.md → реализовать в новом чате
/dm-cc-assistant:review               # ревью изменений
/dm-cc-assistant:update-docs          # обновить документацию
```

### Новые агенты

- **backlog-planner** (opus) — разбивает Must-фичи на задачи ≈1ч, назначает приоритеты, зависимости, итерации
- **task-researcher** (sonnet) — research по T-ID + генерация промпта для реализации
- **code-reviewer** (sonnet) — интерактивный ревью, findings по одному, интеграция с backlog
- **docs-updater** (sonnet) — targeted section edits, 3-layer updates, open questions flow

### Task system

- `.task/backlog.md` — задачи с T-ID (T-001, T-002, ...), приоритетами, зависимостями
- `.task/research.md` — research текущей задачи (перезаписывается)
- `.task/review.md` — результаты ревью (перезаписывается)
- Open questions связаны между сессиями: сессионные → docs / backlog / глобальные (OVERVIEW §9)

**Full Changelog**: https://github.com/Dmatryus/dm-cc-assistant/compare/v0.1.1...v0.2.0
