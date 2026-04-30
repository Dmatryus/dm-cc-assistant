# Release v0.3.0 — Autonomous Epic Lifecycle

> **Edit log:**
> - 2026-04-30 · v0.3.0 · release-manager · created (через v0.2.1 release-skill)

**dm-cc-assistant** делает шаг от «помощника по чеклистам» к **автономному оркестратору эпика**. Большая итерация: команды, формат backlog'а и модель исполнения переосмыслены.

«Долго думаем → автономно делаем → структурированный отчёт → диалог.»

## Что нового

### 4 команды вместо 7

```
/dm-cc-assistant:backlog          # двухуровневая модель эпик/подзадача
/dm-cc-assistant:plan             # 7-фазное планирование активного эпика
/dm-cc-assistant:execute          # автономный параллельный прогон в worktree'ах
/dm-cc-assistant:release          # подготовка релиз-материалов (без push)
```

`/research`, `/review`, `/update-docs`, `/release full` — удалены. Их функциональность поглощена `/plan` и `/execute`. Подробности в [migration guide](migration-v0.3.0.md).

### Автономия и параллельность

`/execute` — главная новинка. Один pre-execute confirm — и агент сам:
1. Создаёт `feat/E-XXX` от main.
2. Запускает подзадачи **волнами параллельно** в отдельных git worktree'ах.
3. Между волнами мёрджит готовые ветки в `feat/E-XXX`, резолвит конфликты в 3 уровня (combine → auto-pick → user dialog только для high-stakes).
4. После всех волн — code-reviewer проходит cross-subtask quality, release-manager собирает агрегированный отчёт.
5. Запускает 4-шаговый финальный диалог: narrative summary → меню по категориям → sub-dialogs → финальное действие.

Pre-execute analysis детектит остатки от прерванных прогонов — предлагает Resume / Restart / Skip по каждой подзадаче.

### Двухуровневая модель backlog'а

Эпики (`E-001`) с подзадачами (`E-001.1`). Inline-статусы подзадач отражают их жизненный цикл: Todo / In Progress / Done / Failed / Skipped / Conflict-blocked / Cancelled. Тип эпика (опциональный): Functional / Tech Debt / Research / Infrastructure / Cleanup.

Sync с docs lightweight'ный — при каждом `/backlog` агент детектит deltas в OVERVIEW / ARCHITECTURE и surface'ит их.

### `/plan` — тяжёлый интерактив

7 фаз с STOP'ами: Context (light code scan + read-list confirm) → Decomposition (5–15 atomic subtasks) → DAG + waves (Mermaid graph + file-overlap check) → Risks + alternatives → Open questions (resolve / defer / promote to research subtask) → DoD → Final review (Accept / Iterate phase X / Abort).

Soft cap: третья итерация той же фазы → агент сигналит «возможно эпик слишком большой».

### `/release` — подготовка, не релиз

Готовит локально:
- CHANGELOG entry
- Release notes (.task/release-notes-vX.Y.Z.md)
- Announcement (.task/announcement-vX.Y.Z.md)
- Tag message (текст для `git tag -a`)
- Migration guide (только при breaking)
- Bumps версии в plugin.json / marketplace.json / etc.

Коммитит на feat/E-XXX. **Не** мёрджит в main, не тэгирует, не пушит — печатает инструкции пользователю. Контроль остаётся за тобой.

### Conflict resolution в 3 уровня

Большинство merge-конфликтов между ветками подзадач — это «дополняющие изменения» (разные импорты, разные методы, разные секции). `release-manager` (merge mode) автоматически объединяет такие через **Combine**. Если combine невозможен и конфликт low-stakes — **Auto-pick** по intent analysis. **User dialog** только для high-stakes: public API breaking / security / data integrity / critical config.

Подзадачи в одной волне **не могут** иметь пересекающиеся `Файлы:` — это структурная инвариантa, проверяется в `/plan` фаза 3.

### Edit log convention

Все генерируемые файлы (`OVERVIEW.md`, `ARCHITECTURE.md`, `CLAUDE.md`, backlog, plan, report, research, releases) теперь содержат секцию `> **Edit log:**` сразу после H1. Latest first, формат: `- YYYY-MM-DD · vX.Y.Z · <agent> · <description>`.

### `[Priority]` теги в ARCHITECTURE §9/§10

Tech Debt и Code Hotspots требуют префикс `[High]` / `[Medium]` / `[Low]`. `/backlog` промоутит в draft по правилу: High → Todo, Medium → Todo (обсудить), Low → Parking lot.

## Установка / обновление

```
/plugin marketplace update dm-cc
/plugin update dm-cc-assistant@dm-cc
```

(Если кнопка Update не работает — см. [Troubleshooting в README](https://github.com/Dmatryus/dm-cc-assistant#troubleshooting).)

После обновления **обязательно** запусти `/dm-cc-assistant:backlog` — он детектит старый формат и предложит миграцию.

## Breaking changes

Удалены команды: `/research`, `/research --done`, `/review`, `/update-docs`, `/release full`. Подробности и инструкции в [migration guide](migration-v0.3.0.md).

## Спасибо

Если ловишь баги или неудобства — issue в [GitHub](https://github.com/Dmatryus/dm-cc-assistant/issues).
