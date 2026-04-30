# dm-cc-assistant v0.3.0 — Autonomous Epic Lifecycle

> **Edit log:**
> - 2026-04-30 · v0.3.0 · release-manager · created (через v0.2.1 release-skill)

🚀 **dm-cc-assistant v0.3.0** — большая итерация.

«Долго думаем → автономно делаем → структурированный отчёт → диалог.»

**4 команды вместо 7:**
- `/backlog` — эпики с подзадачами + sync с docs
- `/plan` — 7-фазное планирование одного эпика
- `/execute` — **автономный параллельный прогон** в git worktree'ах
- `/release` — готовит материалы, не релизит сам

**Главная новинка** — `/execute`: один confirm, агент сам гонит волны подзадач параллельно, мёрджит между волнами с 3-уровневой резолюцией конфликтов, собирает агрегированный отчёт, ведёт структурированный финальный диалог.

Удалены `/research`, `/review`, `/update-docs`, `/release full` — поглощены новыми командами.

Переход с v0.2 — авто-миграция в `/backlog` (опции: Migrate / Wipe / Keep-legacy / Abort).

GitHub Release: https://github.com/Dmatryus/dm-cc-assistant/releases/tag/v0.3.0
