# Форк archify для Project_BL

Форк `tt-a1i/archify` (MIT, v2.16.0), клон от 2026-08-31. Живёт локально,
апстриму ничего не отправляется.

## Зачем форк

Апстрим умеет открыть из Semantic Passport только *исходники на публичном
GitHub* — блок Repository Evidence требует `meta.repository` с `https://github.com/…`
и полным SHA коммита. Нам нужен другой переход: из узла на **соседний артефакт**
с детальной схемой этого узла, локально и без GitHub. Штатного поля для этого
нет, поэтому оно добавлено здесь.

## Что добавлено

Две вещи: **Detail Link** — переход из паспорта на соседнюю схему, и
**Description** — раскрывающееся текстовое описание узла.

## Detail Link

Узел получает необязательное поле `link`, и в паспорте под кнопкой `Copy link`
появляется ссылка на другой артефакт.

```json
{ "id": "parse", "type": "backend", "label": "БЛ_Р_РазборКандидатов",
  "link": { "href": "bki-sides.architecture.html", "label": "Сбор ЮЛ и ФЛ" } }
```

- `href` — обязателен. Разрешены относительный путь до `.html` (с необязательным
  якорем вида `#focus=set`) и абсолютный `https://`. Схема `javascript:` и прочие
  протоколы отсекаются паттерном на уровне JSON-схемы.
- `label` — необязателен, до 48 символов. Без него берётся `Detail` / `详情`
  из i18n.
- Поле поддержано во всех пяти типах: `architecture.components`,
  `dataflow.nodes`, `lifecycle.states`, `sequence.participants`,
  `workflow.nodes`.

Ссылка сознательно оформлена как действие паспорта, а **не** как Repository
Evidence: та секция означает «источник проверен по ревизии», и притворяться ею
для соседнего html было бы враньём. Detail Link ничего не проверяет — это
переход, заявленный автором схемы.

## Description

Узел получает необязательное поле `description` — массив от одного до восьми
абзацев, до 480 символов каждый. В паспорте появляется кнопка `Description`,
панель свёрнута по умолчанию.

```json
{ "id": "rule", "type": "security", "label": "Правило ЮФЛ",
  "description": [
    "Сторона считается прямо в тексте запросов, в двух местах.",
    "Правило унаследовано от прежнего контура и дублирование сохранено осознанно."
  ] }
```

Абзацы едут в SVG-атрибуте `data-node-description` как JSON-массив: так
разрывы абзацев переживают round-trip через атрибут, и не нужен разделитель,
который может встретиться внутри живого предложения. Вьюер разбирает массив,
непарсящееся значение молча даёт пустое описание и скрытую кнопку. Текст
вставляется через `textContent` — разметка внутри описания не исполняется.

Кнопка появляется только у узла с описанием; при закрытии паспорта панель
очищается, состояние раскрытия сбрасывается, `aria-expanded` возвращается в
`false`.

## Изменённые файлы

| Файл | Что |
|---|---|
| `archify/schemas/common.schema.json` | `$defs.nodeLink` (форма и паттерн `href`), `$defs.nodeDescription` |
| `archify/schemas/{architecture,dataflow,lifecycle,sequence,workflow}.schema.json` | поля `link` и `description` у узла |
| `archify/renderers/shared/cli.mjs` | атрибуты `data-node-link`, `data-node-link-label`, `data-node-description` в SVG-узле |
| `archify/renderers/*/render-*.mjs`, `workflow-compiler.mjs` | проброс `link` и `description` в паспорт узла |
| `archify/renderers/shared/i18n.mjs` | ключи `viewer.passport.detailLink*` и `viewer.passport.description*` (en + zh) |
| `archify/assets/template.html` | разметка `#focus-detail-link`, `#btn-focus-description`, `#focus-description-panel`, стиль, заполнение, переключение и сброс |
| `archify/renderers/shared/generated-validators.mjs` | перегенерирован (`npm run generate:validators`) |
| `examples/*.html`, `archify/examples/*.html` | перерендерены под новый шаблон, как требует golden |

## Проверки

- `node scripts/generate-validators.mjs --check` — чисто.
- `node test/golden.mjs` — **all checks passed** (после перерендера примеров;
  `examples/web-app.html` пересобирается командой
  `node bin/archify.mjs render architecture examples/web-app.architecture.json ../examples/web-app.html --quality showcase`).
- Обе схемы Project_BL собраны этим форком: 9/9 проверок, composition showcase
  pass, у `bki-sides` visual-check тоже pass.
- Переход проверен в браузере: ссылка появляется только у узла с `link`,
  у остальных скрыта и атрибуты чисты; клик уводит на соседний артефакт.
- Описание проверено в браузере: кнопка видна только у узла с `description`,
  панель свёрнута до клика, раскрывается по нему, при переходе на узел без
  описания кнопка скрывается и состояние сбрасывается.

Известный дефект апстрима, не связанный с правкой: `npm test` на Windows падает
на `test/workflow-migration.test.mjs` — дочерний процесс получает абсолютный
путь `D:\…` и импортирует его без `pathToFileURL`
(`ERR_UNSUPPORTED_ESM_URL_SCHEME`). Воспроизводится на чистом клоне апстрима.

## Сборка схем проекта

```bash
node /d/AI_Projects/diag_builder/archify/bin/archify.mjs deliver architecture <spec>.json <out>.html --quality showcase --json
```

Навык в `~/.claude/skills/archify` остался апстримным — он не знает ни про
`link`, ни про `description`. Схемы, использующие эти поля, собирать только
этим форком.
