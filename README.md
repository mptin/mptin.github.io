# mptin.github.io — портфолио лабораторных работ

Сайт-портфолио по дисциплине «Программирование на Python».
**Ашихмин Кирилл, группа P3124.**

Опубликован: <https://mptin.github.io>

## Структура репозитория

```
.
├── source/              # исходники сайта (Markdown + конфигурация MkDocs)
│   ├── mkdocs.yml
│   └── docs/
│       ├── index.md     # главная
│       ├── about.md     # об авторе
│       └── LR-1.md … LR-10.md
├── docs/                # собранный сайт — его публикует GitHub Pages
├── requirements.txt     # mkdocs, mkdocs-material
├── .github/workflows/
│   └── deploy-pages.yml # автодеплой на GitHub Pages (ЛР 3)
└── .sourcecraft/
    ├── sites.yaml       # публикация сайта в SourceCraft Sites (ЛР 3)
    └── ci.yaml          # проверка сборки и артефакты (ЛР 3)
```

Исходники и результат сборки лежат раздельно: правится только Markdown в
`source/docs/`, а каталог `docs/` целиком пересобирается одной командой.

## Локальная работа

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

cd source
mkdocs serve                       # предпросмотр на http://127.0.0.1:8000
mkdocs build --strict -d ../docs   # сборка сайта в /docs
```

Флаг `--strict` превращает предупреждения (битые ссылки, страницы вне `nav`) в
ошибки, чтобы сломанный сайт не попал в публикацию.

## Почему Material

Задание оставляет выбор темы за студентом и называет в примерах и `dracula`,
и `material`. Material выбран по функциям: русская морфология в поиске
(`language: ru`), сам полнотекстовый поиск по десяти отчётам, адаптивная
вёрстка ради широких таблиц с метриками, подсветка Python с кнопкой
копирования и блоки `admonition` для замечаний. Всё это есть в Material, плюс
светлая и тёмная схемы, переключающиеся по системной настройке. `dracula` —
по сути цветовая схема, ничего из перечисленного в ней нет.

Подробнее с таблицей критериев в [отчёте по ЛР 1](source/docs/LR-1.md).

## Именование страниц

Страницы работ названы `LR-1.md` … `LR-10.md`, а не `lab_01.md`. Причина: в
задании ЛР 2 требуется адрес отчёта вида `username.github.io/LR-2`. MkDocs
работает в режиме `use_directory_urls: true` и превращает `LR-2.md` в
`LR-2/index.html`, что даёт ровно нужный адрес `/LR-2`.

## Развёртывание

| Платформа | Механизм | Конфигурация |
|---|---|---|
| GitHub Pages | GitHub Actions | `.github/workflows/deploy-pages.yml` |
| SourceCraft Sites | SourceCraft CI | `.sourcecraft/sites.yaml`, `.sourcecraft/ci.yaml` |

Подробности настройки обеих платформ — в [отчёте по ЛР 3](https://mptin.github.io/LR-3).
