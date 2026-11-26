# ru-ai-style — плагин с двумя навыками

Пакет для агентов (Claude Code, Codex, OpenCode и совместимых): русский текст без ИИ-стиля и подготовка к проверке на ИИ-генерацию.

Внутри два самостоятельных навыка, каждый со своим `SKILL.md` и своим триггером:

| Навык | Что делает | Когда срабатывает |
|---|---|---|
| `ru-ai-style` | Находит и убирает приметы машинной генерации, правит стиль, защищает живой голос | «убрать ИИ-стиль», «очеловечить», «почистить нейрослоп», «вычистить канцелярит» |
| `antiplagiat` | Правит текст по тепловой карте «Антиплагиата»/«Думейта», ведёт цикл до снижения доли ИИ, калибрует по отчётам | «снизить процент ИИ», «Думейт показал N %», «подготовить ВКР к антиплагиату» |

Навыки ссылаются друг на друга по относительным путям и должны лежать рядом.

## Быстрый старт

Нужны Python 3.12+ и доступ в сеть: установщик скачивает зависимости детектора. `uv` желателен, но не обязателен.

Windows, из папки плагина:

```powershell
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

macOS и Linux:

```bash
bash install.sh
```

Установщик:

- копирует `skills/ru-ai-style` и `skills/antiplagiat` в `~/.agents/skills/`, заменяя прежние копии;
- ставит детектор: `uv tool install ./detector`, а если `uv` нет — `python -m pip install --user`;
- прописывает `PYTHONIOENCODING=utf-8` в пользовательские переменные;
- проверяет, что `aiw-ru --version` отвечает.

Затем перезапустите терминал и сессию агента: PATH и `PYTHONIOENCODING` применяются к новым процессам. После установки папку плагина можно удалить: детектор поставлен в окружение `uv` (`~/.local/share/uv/tools/aiw-ru`), команда `aiw-ru` лежит в `~/.local/bin`, а навыки — в `~/.agents/skills/`.

Проверка:

```bash
aiw-ru --version
```

Если команды нет, откройте новый терминал. До этого детектор доступен как `python -m aiw_ru`.

## Установка как плагин

Плагин содержит `plugin.json`, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` и `package.json`. Установите пакет штатным способом для вашего хоста — оба навыка появятся из папки `skills/` и будут вызываться независимо.

## Установка как папки, вручную

Если не хотите запускать установщик, скопируйте обе папки-соседа в общий каталог навыков:

- **Codex:** `$HOME/.agents/skills/ru-ai-style/` и `$HOME/.agents/skills/antiplagiat/`
- **OpenCode:** `~/.agents/skills/ru-ai-style/` и `~/.agents/skills/antiplagiat/` (на Windows — `C:\Users\<user>\.agents\skills\...`)

Имя каждой папки совпадает с полем `name` в её `SKILL.md`. Относительные ссылки `../ru-ai-style/SKILL.md` и `../antiplagiat/SKILL.md` разрешаются, только если папки лежат рядом.

После копирования поставьте детектор (раздел «Зависимость от aiw-ru») и перезапустите сессию агента.

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
├── install.ps1        # установщик для Windows
├── install.sh         # установщик для macOS и Linux
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

## Лицензия и происхождение

MIT. Пакет собран из `slopotron` (Rodion Scryabin), `avoid-ai-writing-russian` (Conor Bronsdon, Ilya Lubenets) и под-навыка `antiplagiat` (Ilya Lubenets). Подробности — в `NOTICE.md` и `skills/ru-ai-style/NOTICE`.
