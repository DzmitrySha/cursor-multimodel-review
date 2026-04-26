---
name: Review modes skill
overview: Ввести три явных режима ревью (light / standard / deep) в skill и команду, обновить описание плагина и README; синхронизировать критиков с полем `Review mode` в задаче субагента.
todos:
  - id: skill-modes
    content: "Переписать SKILL.md: режимы, триггеры, workflow, приоритет"
    status: completed
  - id: command-readme
    content: Обновить mm-review.md, plugin.json, README.md
    status: completed
  - id: agents-mode-line
    content: Добавить блок Review mode во всех agents/*.md
    status: completed
isProject: false
---

# План: режимы light / standard / deep

## Цели

- **standard** по умолчанию: три параллельных критика + жёсткие лимиты evidence packet (экономия токенов на родителе без отказа от мультимодели).
- **light**: один `inherit-critic`, минимальный пакет (быстро и дёшево).
- **deep**: прежнее поведение «полный контекст» для высоких ставок.
- Явный выбор по словам пользователя; при отсутствии — **standard**.

## Файлы

1. **[skills/adversarial-multimodel-review/SKILL.md](skills/adversarial-multimodel-review/SKILL.md)**  
   - Обновить YAML `description` под режимы.  
   - Заменить вводные абзацы про «expensive / no light mode».  
   - Добавить раздел **Review modes** (правила выбора, лимиты standard/light, политика deep).  
   - Добавить **приоритет** над старыми формулировками.  
   - Расширить **Trigger phrases** (EN/RU: лёгкий, стандартный, глубокий, light/standard/deep).  
   - Переписать **Default review behavior** на три подраздела или оставить один блок «Deep-only» и сослаться на режимы.  
   - В **Workflow**: шаг про evidence packet — зависимость от режима; шаг 3 — какие критики для light vs standard/deep; шаг 4 — первая строка задачи критика: `Review mode: …`.

2. **[commands/mm-review.md](commands/mm-review.md)**  
   - Маршрутизация: `light`/лёгкий → light; `deep`/глубокий/max → deep; иначе standard; затем вызов skill с этим режимом.

3. **[.cursor-plugin/plugin.json](.cursor-plugin/plugin.json)**  
   - Обновить `description` (без «Always-on deep»); при желании bump `version` до `0.2.0`.

4. **[README.md](README.md)**  
   - Краткая таблица/список режимов, примеры промптов, уточнение Max Mode для deep, правка блока «Run a review».

5. **Критики** — [agents/gemini-critic.md](agents/gemini-critic.md), [agents/gpt-critic.md](agents/gpt-critic.md), [agents/opus-critic.md](agents/opus-critic.md), [agents/inherit-critic.md](agents/inherit-critic.md)  
   - Одна согласованная вставка: читать первую строку `Review mode:`; для light/standard не расширять контекст за пределы пакета; для deep или отсутствия строки — сохранить текущий абзац про full-context (обратная совместимость со старыми вызовами).

## Зависимости

Режим работает только если родительский агент следует skill и передаёт критикам `Review mode:` — это зафиксировано в workflow шаге 4.
