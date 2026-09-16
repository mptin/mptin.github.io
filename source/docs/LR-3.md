# ЛР 3. CI/CD для статического сайта в SourceCraft

## Цель работы

Настроить автоматическое развёртывание сайта на MkDocs двумя способами, через
SourceCraft и через GitHub Actions, из одного локального репозитория с двумя
удалёнными.

## Задание

1. Реализовать автоматическое развёртывание сайта на SourceCraft.
2. Реализовать автоматическое развёртывание того же сайта через GitHub Actions.
3. В одном локальном репозитории настроить два удалённых (`origin` и
   `sourcecraft`).
4. Указать 4 ссылки: на оба сайта и на оба репозитория.
5. Описать, какие настройки нужно сделать в самих репозиториях.

## Четыре ссылки

| Что | Ссылка |
|---|---|
| Сайт на GitHub Pages | <https://mptin.github.io> |
| Репозиторий на GitHub | <https://github.com/mptin/mptin.github.io> |
| Сайт на SourceCraft | <https://anthony-004.sourcecraft.site/portfolio> |
| Репозиторий на SourceCraft | <https://sourcecraft.dev/anthony-004/portfolio> |

## Реализация

### Как организован репозиторий

Один локальный репозиторий отправляется в два удалённых:

```bash
git remote add origin https://github.com/mptin/mptin.github.io.git
git remote add sourcecraft ssh://git@ssh.sourcecraft.dev/anthony-004/portfolio.git

git remote -v      # проверка: должны быть оба
git push origin main
git push sourcecraft main
```

Для GitHub используется HTTPS с personal access token, для SourceCraft — SSH.
Токену GitHub нужны две области: `repo` и `workflow`. Без второй пуш
отклоняется целиком, потому что в репозитории лежит
`.github/workflows/deploy-pages.yml`.

SSH выбран сознательно: при HTTPS токен пришлось бы вписывать прямо в адрес
удалённого репозитория, и он осел бы в `.git/config` открытым текстом. При SSH
на диске лежит только приватный ключ, а серверу отдаётся публичный.

Структура, общая для обеих платформ:

```
mptin.github.io/
├── source/                      # исходники MkDocs
├── docs/                        # собранный сайт
├── requirements.txt             # mkdocs + mkdocs-material
├── .github/workflows/
│   └── deploy-pages.yml         # деплой на GitHub Pages
└── .sourcecraft/
    ├── sites.yaml               # что публиковать в SourceCraft Sites
    └── ci.yaml                  # workflow проверки сборки
```

### Деплой через GitHub Actions

```yaml
name: Deploy MkDocs to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements.txt
      - run: mkdocs build --strict -f source/mkdocs.yml -d ../docs
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: docs

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### Деплой через SourceCraft

`.sourcecraft/sites.yaml` описывает, что публиковать:

```yaml
site:
  root: "docs"
  ref: "main"
```

`.sourcecraft/ci.yaml` проверяет сборку и сохраняет артефакт:

```yaml
on:
  push:
    - workflows: [build-site]
      filter:
        branches: [main]

workflows:
  build-site:
    tasks:
      - name: build
        cubes:
          - name: mkdocs-build
            image: python:3.12-slim
            script:
              - pip install --no-cache-dir -r requirements.txt
              - mkdocs build --strict -f source/mkdocs.yml -d ../docs
            artifacts:
              paths:
                - docs
```

## Настройки в самих репозиториях

Про них в задании отдельный пункт, и не зря: конфигурации в репозитории мало,
часть переключателей живёт в вебе.

### GitHub

1. Settings → Pages → Source переключить с `Deploy from a branch` на
   `GitHub Actions`.

    !!! warning "Про этот шаг легко забыть"
        В ЛР 1 сайт публиковался напрямую из каталога `/docs` ветки `main`.
        Если оставить эту настройку, GitHub Pages продолжит отдавать
        закоммиченный `docs/`, а результат работы Actions проигнорирует.
        Workflow при этом отработает зелёным, и решить, что всё хорошо, очень
        просто. Проверять надо так: поменять что-нибудь заметное в
        `source/docs/index.md`, закоммитить и запушить **без** локальной
        пересборки `docs/`. Если изменение появилось на сайте, значит собрал
        именно Actions.

2. Settings → Actions → General → Workflow permissions: убедиться, что
   Actions разрешены для репозитория.
3. Репозиторий должен быть публичным, иначе Pages на бесплатном тарифе не
   работает.

Права в самом workflow (`permissions: pages: write, id-token: write`) нужны
штатному экшену `actions/deploy-pages`, без них деплой падает с ошибкой
доступа.

### SourceCraft

1. Создать публичную организацию и публичный репозиторий. Приватные
   репозитории Sites не публикует.
2. Создать персональный токен (PAT) с правами Maintainer для работы по HTTPS.
   Токен показывается один раз, его надо сразу сохранить.
3. В настройках репозитория включить Sites.
4. Проверить, что CI/CD включён: workflow запускается из `.sourcecraft/ci.yaml`
   автоматически при пуше в `main`.

## Выводы

Один локальный репозиторий спокойно обслуживает несколько платформ. Git
поддерживает произвольное число удалённых, а `origin` это просто имя по
умолчанию без какого-либо особого статуса.

Токен прямо в URL удалённого репозитория удобен, но строка
`https://<аккаунт>:<токен>@git...` остаётся в `.git/config` открытым текстом.
Для учебной задачи сойдёт, в рабочем проекте лучше SSH-ключ или менеджер
учётных данных.

Флаг `--strict` у `mkdocs build` защищает от тихих поломок: без него битая
ссылка или файл, не включённый в `nav`, дают только предупреждение, и
сломанный сайт спокойно уезжает в прод.

Отдельный вывод про настройку платформы. Правильный workflow при неверной
настройке Pages отработает успешно и ничего не задеплоит, причём молча.
После первого деплоя обязательно надо открыть сайт и убедиться, что изменения
видны.
