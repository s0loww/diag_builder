# Форк archify для Project_BL

Форк `tt-a1i/archify` (MIT, v2.16.0), клон от 2026-08-31. Живёт локально,
апстриму ничего не отправляется.

## Зачем форк

Апстрим умеет открыть из Semantic Passport только *исходники на публичном
GitHub* — блок Repository Evidence требует `meta.repository` с `https://github.com/…`
и полным SHA коммита. Нам нужен другой переход: из узла на **соседний артефакт**
с детальной схемой этого узла, локально и без GitHub. Штатного поля для этого
нет, поэтому оно добавлено здесь.

## Что добавлено: Detail Link

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

## Изменённые файлы

| Файл | Что |
|---|---|
| `archify/schemas/common.schema.json` | `$defs.nodeLink` — форма и паттерн `href` |
| `archify/schemas/{architecture,dataflow,lifecycle,sequence,workflow}.schema.json` | поле `link` у узла |
| `archify/renderers/shared/cli.mjs` | атрибуты `data-node-link`, `data-node-link-label` в SVG-узле |
| `archify/renderers/*/render-*.mjs`, `workflow-compiler.mjs` | проброс `link` в паспорт узла |
| `archify/renderers/shared/i18n.mjs` | ключи `viewer.passport.detailLink*` (en + zh) |
| `archify/assets/template.html` | разметка `#focus-detail-link`, стиль, заполнение и сброс |
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

Известный дефект апстрима, не связанный с правкой: `npm test` на Windows падает
на `test/workflow-migration.test.mjs` — дочерний процесс получает абсолютный
путь `D:\…` и импортирует его без `pathToFileURL`
(`ERR_UNSUPPORTED_ESM_URL_SCHEME`). Воспроизводится на чистом клоне апстрима.

## Сборка схем проекта

```bash
node /d/AI_Projects/diag_builder/archify/bin/archify.mjs deliver architecture <spec>.json <out>.html --quality showcase --json
```

Навык в `~/.claude/skills/archify` остался апстримным — он не знает про `link`.
Схемы со ссылками собирать только этим форком.
