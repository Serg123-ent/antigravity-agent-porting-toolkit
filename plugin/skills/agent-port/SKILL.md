---
name: agent-port
description: Автоматизированный аудит и адаптация сторонних AI-агентов (из Claude Code, OpenAI Codex, Cursor и др.) под экосистему Google Antigravity с конвертацией правил, навыков и хуков.
---

# Навык: Портирование агента в Antigravity (`/agent-port`)

Данный навык реализует детерминированный протокол портирования внешних агентов в среду Google Antigravity (CLI `agy` и Antigravity IDE).

## Порядок выполнения

1. **Аудит исходной платформы**:
   - Определите исходный харнесс (Claude Code, OpenAI Codex, Cursor, Qoder, Copilot и др.).
   - Найдите конфигурации: `plugin.json`, `hooks.json`, `AGENTS.md`, `CLAUDE.md`, `.cursorrules`.

2. **Миграция инструкций и правил**:
   - Перенесите основные поведенческие правила в `rules/AGENTS.md` (чистый Markdown без frontmatter).
   - Очистите текст от проприетарных переменных окружения (`${CLAUDE_PLUGIN_ROOT}` и др.).

3. **Адаптация навыков**:
   - Создайте для каждого навыка файл `skills/<name>/SKILL.md` с валидным YAML frontmatter (`name`, `description`).
   - Замените проприетарные вызовы на стандартные инструменты Antigravity.

4. **Эмуляция хуков жизненного цикла**:
   - Преобразуйте `SessionStart` в раздел предварительных требований (Prerequisites).
   - Преобразуйте `UserPromptSubmit` (режимы lite/full/ultra) в параметры слэш-команд или опрос через `ask_question`.
   - Удалите файлы шелл-хуков (`*.json`), вызывающие сторонние скрипты Node.js/Python.

5. **Проверка манифеста**:
   - Убедитесь, что `plugin.json` содержит только поддерживаемые поля: `name`, `displayName`, `description`, `version`, `suggestedPrompts`.
