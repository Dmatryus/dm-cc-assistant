# Changelog

Все заметные изменения проекта документируются в этом файле.

Формат — [Keep a Changelog](https://keepachangelog.com/ru/1.1.0/), версионирование — [SemVer](https://semver.org/lang/ru/).

## [0.1.0] — 2026-04-15

Первый релиз. Покрывает сценарий старта нового проекта с нуля.

### Added
- Манифест плагина `.claude-plugin/plugin.json` с метаданными (license `Apache-2.0`, `repository`, `homepage`, `keywords`).
- Self-hosted marketplace `.claude-plugin/marketplace.json` (имя `dm-cc`) — тот же репозиторий выступает и каталогом, и плагином. Установка: `/plugin marketplace add Dmatryus/dm-cc-assistant@v0.1.0` → `/plugin install dm-cc-assistant@dm-cc`.
- Skill-оркестратор `/dm-cc-assistant:project-init` — запускает четыре шага последовательно и проверяет появление файлов после каждого.
- Агент `overview-interviewer` — проводит продуктовое интервью в стиле «draft → подтверждение» и создаёт `OVERVIEW.md` (9 разделов, MoSCoW-скоуп, non-goals, mermaid user flow).
- Агент `architecture-interviewer` — обсуждает технические выборы по одному (Stack, Module Map, Data Model и др.) с альтернативами, плюсами-минусами и рекомендацией; создаёт `ARCHITECTURE.md` (10 разделов) и фиксирует тип проекта для следующего шага.
- Агент `claude-md-generator` — синтезирует `CLAUDE.md` из первых двух документов по шаблону WHY / WHAT / HOW / CONSTRAINTS / NEVER / PRINCIPLES; раздел PRINCIPLES с четырьмя принципами Карпатого неизменяем.
- Агент `project-scaffolder` — для KMP-проектов создаёт базовый `.claude/` scaffold (skill `kmp-build`, агент `kmp-reviewer`, hook `ktlintFormat`); для не-KMP завершает работу штатно.
- Hook `SessionStart` плагина — приветствие с напоминанием о `/dm-cc-assistant:project-init`.
- Документация: `README.md` (на русском, с установкой и ограничениями v0.1.0), `OVERVIEW.md`, `ARCHITECTURE.md`, `CLAUDE.md`, `docs/claude-workflow-guide.md`, `LICENSE` (Apache-2.0).

### Known limitations
- Скаффолдинг поддерживается только для KMP-проектов; для остальных типов шаг штатно пропускается.
- Все интервью и генерируемые документы — на русском языке.
- Анализ существующих кодовых баз не поддерживается — только пустые директории.
- Все четыре шага интерактивны, headless-режима нет.
- Хук `ktlintFormat` в KMP-скаффолде делает полнопроектное форматирование (у `ktlint-gradle` нет штатного per-file флага); при проблемах со скоростью хук можно убрать.
