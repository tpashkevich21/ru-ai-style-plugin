# ru-ai-style — плагин с двумя навыками

Пакет для агентов (Claude Code, Codex, OpenCode и совместимых): русский текст без ИИ-стиля и подготовка к проверке на ИИ-генерацию.

Внутри два самостоятельных навыка, каждый со своим `SKILL.md` и своим триггером:

| Навык | Что делает | Когда срабатывает |
|---|---|---|
| `ru-ai-style` | Находит и убирает приметы машинной генерации, правит стиль, защищает живой голос | «убрать ИИ-стиль», «очеловечить», «почистить нейрослоп», «вычистить канцелярит» |
| `antiplagiat` | Правит текст по тепловой карте «Антиплагиата»/«Думейта», ведёт цикл до снижения доли ИИ, калибрует по отчётам | «снизить процент ИИ», «Думейт показал N %», «подготовить ВКР к антиплагиату» |

Навыки ссылаются друг на друга по относительным путям и должны лежать рядом.

## Установка как плагин

Плагин содержит `plugin.json`, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` и `package.json`. Установите пакет штатным способом для вашего хоста — оба навыка появятся из папки `skills/` и будут вызываться независимо.

## Установка как папки

Скопируйте обе папки-соседа в общий каталог навыков:

- **Codex:** `$HOME/.agents/skills/ru-ai-style/` и `$HOME/.agents/skills/antiplagiat/`
- **OpenCode:** `~/.agents/skills/ru-ai-style/` и `~/.agents/skills/antiplagiat/` (на Windows — `C:\Users\<user>\.agents\skills\...`)

Имя каждой папки совпадает с полем `name` в её `SKILL.md`. Относительные ссылки `../ru-ai-style/SKILL.md` и `../antiplagiat/SKILL.md` разрешаются, только если папки лежат рядом.

## Зависимость от aiw-ru

`ru-ai-style` работает без Python, сети и детектора, но без `aiw-ru` теряется то, что важно для результата: `validate` (проверка, что правка не съела число, оговорку, цитату или формулу), точные позиции находок, замер по фрагментам и калибровка по отчётам. `antiplagiat` без `aiw-ru` вообще не может измерить долю ИИ — остаётся слепая правка.

Исходники `aiw-ru` лежат в папке `detector/`. Разовая установка:

```bash
uv tool install ./detector
```

Дальше команда `aiw-ru` доступна из любой папки. На Windows перед запуском включи UTF-8, иначе таблица падает на кодировке консоли или печатает вопросительные знаки:

```powershell
$env:PYTHONIOENCODING = "utf-8"
[Console]::OutputEncoding = [Text.Encoding]::UTF8
```

Без установки — запуск из папки:

```bash
uv run --project detector aiw-ru scan <файл> --context vak
uv run --project detector --extra ml aiw-ru models install
```

Первый запуск поставит зависимости; веса моделей скачиваются отдельно.

## Структура

```
ru-ai-style-plugin/
├── plugin.json
├── package.json
├── .claude-plugin/plugin.json
├── .codex-plugin/plugin.json
├── LICENSE
├── NOTICE.md
├── detector/          # исходники aiw-ru (src/, pyproject.toml, uv.lock)
└── skills/
    ├── ru-ai-style/
    │   ├── SKILL.md
    │   ├── README.md
    │   ├── NOTICE
    │   ├── LICENSE
    │   └── references/
    └── antiplagiat/
        ├── SKILL.md
        └── references/
            ├── antiplagiat.md
            └── domate.md
```

## Лицензия

MIT. Подробности приведены в файле `LICENSE`, в `NOTICE.md` и `skills/ru-ai-style/NOTICE`.
