# Migration guide — v0.2.x → v0.3.0

> **Edit log:**
> - 2026-04-30 · v0.3.0 · release-manager · created (вручную, через v0.2.1 release-skill, кура-яйцо §12.6)

v0.3.0 — большая итерация. Команды переименованы / удалены, формат backlog'а изменён, ввели worktree-based параллельное исполнение.

## TL;DR

- Запусти `/dm-cc-assistant:backlog` после обновления — агент детектит старый формат и предлагает варианты миграции.
- Старые команды `/research`, `/review`, `/update-docs` удалены — их функциональность поглощена `/plan` и `/execute`.
- `/release` больше не пушит и не тэгирует — готовит материалы локально, печатает инструкции.

## Новые команды

| v0.2 | v0.3 | Что изменилось |
|---|---|---|
| `/backlog` | `/dm-cc-assistant:backlog` | Двухуровневая модель эпик/подзадача, sync с docs, suggestions с rejected memory |
| — | `/dm-cc-assistant:plan` | **Новая.** 7-фазное планирование активного эпика |
| — | `/dm-cc-assistant:execute` | **Новая.** Автономный параллельный прогон через worktree'ы |
| `/release` (mode) | — | Удалена. Закрытие подзадач — автоматически в `/execute` |
| `/release full` | `/dm-cc-assistant:release` | Подкоманд нет. `/release` делает всё сразу. **НЕ пушит/тэгирует** — печатает инструкции. |

## Удалённые команды (Breaking)

| Команда | Замена |
|---|---|
| `/dm-cc-assistant:research T-ID` | Поглощено `/plan` — research-подзадачи как часть плана |
| `/dm-cc-assistant:research T-ID --done` | Автоматически в `/execute` (через финальный диалог + `/release`) |
| `/dm-cc-assistant:review` | Поглощено `/execute` (in-execute review после волн) и `/release` (release-readiness) |
| `/dm-cc-assistant:update-docs` | `docs-updater` работает внутри других команд (например, sync в `/backlog`) |

## Миграция backlog'а

При первом запуске `/dm-cc-assistant:backlog` после upgrade'а — агент детектит плоский T-ID формат и предлагает 4 опции:

1. **Migrate (default)** — Done T-IDs группируются в архивный эпик `E-000 «v0.2 release»`. Активные T-IDs становятся подзадачами выбранного эпика или новых. Backlog переписывается в v0.3 формате. Бэкап в `.task/backlog.v02.md`.
2. **Wipe** — бэкап старого в `.task/backlog.v02.md`, генерация эпиков с нуля из текущих OVERVIEW + ARCHITECTURE.
3. **Keep-legacy** — T-IDs остаются в секции `## Legacy`, новые эпики в стандартных секциях. Двойная модель для постепенного перехода.
4. **Abort** — оставить как есть. `/backlog` v0.3 не работает на v0.2 формате — пока не запустишь миграцию, остальные команды не сработают.

## Структурные изменения

### `.task/` директория

Новые файлы (создаются на лету):

- `plan-E-XXX.md` — план эпика (`epic-planner`)
- `report-E-XXX.Y.md` — отчёт подзадачи (`execution-agent`)
- `report-E-XXX.md` — агрегированный отчёт эпика (`release-manager` aggregation)
- `research-E-XXX.Y.md` — research-deliverable (`execution-agent` для research-подзадачи)
- `resume-E-XXX.Y.md` — resume context (`resume-analyzer`)
- `review-E-XXX.md` — in-execute review (`code-reviewer` in-execute mode)
- `review-E-XXX-pre-release.md` — release-readiness review (`code-reviewer` release-readiness mode)
- `merge-E-XXX-wave-N.md` — merge log per wave (опционально, `release-manager` merge mode)
- `release-notes-vX.Y.Z.md`, `announcement-vX.Y.Z.md`, `migration-vX.Y.Z.md` — release artifacts (`release-manager` full mode)
- `backlog-archive.md` — auto-archive при `## Done` > 5 (`release-manager` full)
- `.execute-active` — marker активного `/execute` (machine-only, не в edit log)

Старые `.task/` файлы (`research.md`, `review.md`, `release-notes-v0.2.x.md`, `tg-post.md`) **не трогаются** — остаются как исторические артефакты.

### Worktree-based исполнение

`/execute` создаёт `feat/E-XXX` от main, потом для каждой подзадачи `feat/E-XXX.Y-<slug>` worktree в `.worktrees/E-XXX.Y/`. Между волнами `release-manager` мёрджит подзадачи в `feat/E-XXX` и удаляет worktree'ы Done подзадач.

**Важно:** добавь `.worktrees/` в `.gitignore` своего проекта (если ещё не сделано).

### `[Priority]` теги в ARCHITECTURE.md

В §9 Tech Debt и §10 Code Hotspots каждая запись теперь требует `[Priority]` префикс:

```markdown
## 9. Tech Debt

- [High] <описание> — <почему блокер>
- [Medium] <описание> — <обоснование>
- [Low] <описание> — <почему не срочно>
```

Уровни: High / Medium / Low. `/backlog` использует тег для определения секции в draft (High → Todo, Medium → Todo с пометкой обсудить, Low → Parking lot). Для legacy записей без тега — fallback на Medium.

### Edit log convention

Все генерируемые файлы (OVERVIEW, ARCHITECTURE, CLAUDE, backlog, plan, report, research, drafts) теперь содержат секцию edit log сразу после H1:

```markdown
# OVERVIEW.md — <Project>

> **Edit log:**
> - 2026-04-30 · v0.3.0 · docs-updater · обновил §2 Module Map
> - 2026-04-26 · v0.2.0 · docs-updater · v2-фичи в Must
> - 2026-04-15 · v0.1.0 · overview-interviewer · создан
```

Latest first. Для legacy файлов без edit log — агенты создают секцию с одной записью (текущая правка) при первой правке.

## Что делать после обновления

1. **Запусти `/dm-cc-assistant:backlog`** — мигрируй (или wipe / keep-legacy) старый backlog.
2. **Проверь `.gitignore`** — добавь `.worktrees/` если ещё нет.
3. **Прочитай новый README** — обновлённый типичный цикл.
4. **Если есть скрипты / CI**, упоминающие `/research`, `/review`, `/update-docs` — обнови их (теперь нет таких команд).
5. **Удалённый формат** ARCHITECTURE.md §9/§10 — добавь `[Priority]` теги к существующим записям (или дай агенту fallback на Medium при следующем `/backlog` sync).

## Helpline

Если что-то сломано или непонятно — issue в [GitHub](https://github.com/Dmatryus/dm-cc-assistant/issues).
