# Antigravity Agent Porting Toolkit & Integration Protocol

[English documentation](README_EN.md) | [Русская документация](README.md)

Комплексный протокол, документация и готовый плагин-шаблон для портирования и интеграции AI-агентов из различных сред выполнения (**Agent Harnesses**: Claude Code, OpenAI Codex, Cursor, Qoder, OpenCode, Windsurf, Devin) в экосистему **Google Antigravity** (CLI `agy` и Antigravity IDE).

---

## Содержание

1. [Обзор проекта](#обзор-проекта)
2. [Проблема импорта агентов из Codex и Claude в Antigravity](#проблема-импорта-агентов-из-codex-и-claude-в-antigravity)
3. [Какие проблемы решает Antigravity Agent Porting & Integration Protocol](#какие-проблемы-решает-antigravity-agent-porting--integration-protocol)
4. [Быстрый старт в Antigravity](#быстрый-старт-в-antigravity)
5. [Структура репозитория](#структура-репозитория)
6. [Лицензия](#лицензия)

---

## Обзор проекта

Современные AI-агенты разрабатываются под разные среды выполнения (харнессы), использующие несовместимые механизмы доставки:
* **Claude Code**: маркетплейс плагинов, шелл-хуки (`SessionStart`, `UserPromptSubmit`, `SubagentStart`) на базе скриптов Node.js, переменные `${CLAUDE_PLUGIN_ROOT}`.
* **OpenAI Codex**: расширения с событиями жизненного цикла, доверительная модель исполнения скриптов (`/hooks`).
* **Google Antigravity**: безопасная инструментально-центричная среда (Tool-Centric Architecture), управляемая строгой структурой каталогов (`skills/`, `rules/`, `plugin.json`), встроенными примитивами оркестрации (`invoke_subagent`, `ask_question`) и протоколом MCP.

Данный проект предоставляет стандартизированный протокол адаптации, устраняющий несовместимости, и готовый шаблон плагина.

---

## Проблема импорта Агентов из Codex и Claude в Antigravity

При попытке прямого копирования плагинов из Claude Code или Codex в Antigravity разработчики сталкиваются с рядом фундаментальных архитектурных барьеров:

### 1. Несовместимость хуков жизненного цикла (Lifecycle Hooks)
* **В Claude Code / Codex**: плагины часто полагаются на скрытое исполнение shell-команд при каждом событии (например, запуск скрипта Node.js через `SessionStart` или `UserPromptSubmit` для динамической инъекции промпта или проверки статуса).
* **В Antigravity**: фоновые шелл-перехватчики отсутствуют по соображениям безопасности и детерминизма. Прямое копирование файлов хуков (`claude-codex-hooks.json`, `cursor-hooks.json`) приводит либо к засорению логов ошибками `node: command not found`, либо просто игнорируется средой.

### 2. Привязка к проприетарным путям и переменным окружения
* Плагины Claude/Codex жестко рассчитывают на наличие путей `process.env.CLAUDE_PLUGIN_ROOT` или `$CODEBUDDY_CONFIG_DIR`, а также сохраняют состояние в непереносимые директории (`~/.claude/.ponytail-active`).
* В Antigravity эти переменные не определены, что приводит к аварийному завершению скриптов.

### 3. Различия в схеме манифеста `plugin.json`
* Манифесты в Codex и Claude могут содержать произвольные поля (`skills`, `rules`, `hooks`, `author`, `homepage`).
* В Antigravity манифест `plugin.json` валидируется строго. Неизвестные поля отбрасываются, а синтаксические расхождения могут помешать регистрации плагина.

### 4. Несовместимость модели субагентов (Subagents Orchestration)
* В Claude Code перехват субагентов (`SubagentStart`) часто выполняет инъекцию контекста через JSON-вывод хука.
* В Antigravity контекст передается строго через нативные вызовы (`invoke_subagent` с явным системным промптом и описанием роли) либо декларативные описания субагентов.

### 5. Обработка слэш-команд и аргументов
* Пространства имен команд (например, `/ponytail:review` или `/plugin:command`) не транслируются напрямую. В Antigravity слэш-команда формируется из имени директории навыка (`skills/<name>/SKILL.md`).

---

## Какие проблемы решает промпт Antigravity Agent Porting & Integration Protocol

Специализированный протокол [ANTIGRAVITY_AGENT_PORTING_PROTOCOL.md](ANTIGRAVITY_AGENT_PORTING_PROTOCOL.md) решает задачу систематической миграции за счет детерминированного 5-шагового процесса:

1. **Двойной аудит (Source Audit)**: Выявление скрытых зависимостей от Node.js/Python, проприетарных переменных окружения и жестко закодированных путей.
2. **Паттерны замещения хуков (Hook Emulation Patterns)**:
   * `SessionStart` (инициализация) $\rightarrow$ переносится в раздел `Prerequisites` файла `AGENTS.md` или в отдельный навык онбординга (`skills/<agent>-init`).
   * `UserPromptSubmit` (переключение режимов `lite / full / ultra`) $\rightarrow$ преобразуется в интерактивный выбор через `ask_question`, параметры аргументов слэш-команд или локальный конфиг (`.antigravity/settings.json`).
   * `SubagentStart` (проброс контекста в субагентов) $\rightarrow$ трансформируется в явную декларацию системного промпта через `invoke_subagent` или шаблоны агентов.
   * `PreToolUse` / `PostToolUse` $\rightarrow$ переносятся в строгие инварианты `Critical Rules` в `AGENTS.md`.
3. **Очистка и нормализация правил (Context Migration)**: Удаление специфичного XML/JSON-синтаксиса и очистка `rules/AGENTS.md` от YAML-заголовков.
4. **Конвертация инструментов в нативный Toolset**: Трансляция кастомных команд чтения/записи в стандартные вызовы Antigravity (`view_file`, `write_to_file`, `replace_file_content`, `run_command`).
5. **Валидация упаковки**: Контроль корректности манифеста `plugin.json` и изоляция пространств имен MCP-серверов.

---

## Быстрый старт в Antigravity

### Вариант 1. Использование плагина из репозитория

1. **Скопируйте директорию плагина** в системный каталог пользовательских плагинов:
   * **Windows**:
     ```powershell
     Copy-Item -Recurse -Force plugin "$HOME\.gemini\config\plugins\antigravity-porting-toolkit"
     ```
   * **Linux / macOS**:
     ```bash
     cp -r plugin ~/.gemini/config/plugins/antigravity-porting-toolkit
     ```

2. **Перезапустите среду Antigravity**:
   * Для Antigravity IDE: выполните перезагрузку окна / перезапуск приложения, чтобы среда проиндексировала новую директорию.
   * Для Antigravity CLI (`agy`): перезапустите сессию терминала.

3. **Используйте команду**:
   В чате появится доступ к команде:
   ```text
   /agent-port
   ```

---

### Вариант 2. Быстрое подключение на уровне проекта (Workspace-level)

Если вы хотите подключить правила и навыки без глобальной установки:

1. Скопируйте файл правил `plugin/rules/AGENTS.md` в корень вашего рабочего проекта под именем `AGENTS.md`.
2. Скопируйте папку `plugin/skills/` в корень проекта в `.agents/skills/` или `.gemini/skills/`.
3. Навыки станут доступны в текущей рабочей области автоматически.

---

## Структура репозитория

```text
├── .gitignore                               # Исключения системных и временных файлов
├── LICENSE                                  # Лицензия MIT
├── README.md                                # Основная документация (на русском языке)
├── README_EN.md                             # Английская версия документации
├── ANTIGRAVITY_AGENT_PORTING_PROTOCOL.md     # Полный протокол архитектурной миграции
└── plugin/                                  # Готовый шаблон плагина для Antigravity
    ├── plugin.json                          # Строгий манифест плагина
    ├── rules/
    │   └── AGENTS.md                        # Постоянные правила (без YAML-frontmatter)
    └── skills/
        └── agent-port/
            └── SKILL.md                     # Навык /agent-port с метаданными
```

---

## Лицензия

Проект распространяется под открытой лицензией [MIT](LICENSE).
