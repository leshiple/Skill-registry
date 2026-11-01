# Реестр навыков

Общий реестр навыков для веб-разработки компании. Репозиторий подключается в проекты как `git submodule`, а проекты фиксируют используемую версию тегом.

## Состав

- `skills/common` — общие правила для всех стеков;
- `skills/vue` — единый набор навыков для фронтенд-проектов на Vue;
- `skills/python` — единый набор навыков для бэкенд-проектов на Python;
- `skills/elixir` — единый набор навыков для бэкенд-проектов на Elixir;
- `profiles` — воспроизводимые наборы навыков;
- `tools` — проверка registry и синхронизация ссылок в целевом проекте.

Все документы и навыки в этом репозитории пишутся на русском языке.

## Формат навыка

Каждый навык хранится в отдельном каталоге и имеет главный файл `SKILL.md`.

```yaml
---
name: уникальное-имя
description: Короткое условие применения навыка.
stack: common
scope: shared
---
```

Текст навыка должен отвечать на три вопроса:

1. Когда его использовать?
2. Какой порядок действий выполнить?
3. Как понять, что работа завершена?

Перед pull request запускай `python3 tools/validate-skills`.

## Подключение к проекту

Сначала клонируй registry в целевой проект как submodule:

```bash
git submodule add https://github.com/leshiple/Skill-registry.git skill-registry
```

Затем переключись на последний опубликованный тег:

```bash
git -C skill-registry fetch --tags
git -C skill-registry checkout "$(git -C skill-registry tag --sort=-version:refname | head -n 1)"
```

### Выбор профиля и ИИ-инструментов

Универсальный скрипт сам создаёт `.skill-registry.yaml` и симлинки:

```bash
python3 skill-registry/tools/skill-registry setup
```

Скрипт попросит выбрать профиль и один или несколько ИИ-инструментов:

- `frontend` — общие навыки и Vue;
- `backend-python` — общие навыки и Python;
- `backend-elixir` — общие навыки и Elixir;
- `fullstack` — общие навыки, Vue, Python и Elixir;
- `Claude` — `.claude/skills`;
- `Cursor` — `.cursor/skills`;
- `Codex` — `.agents/skills`;
- `DeepSeek` — `.deepseek/skills`.

После выбора скрипт создаст файл примерно такого вида:

```yaml
profile: frontend
registry_path: skill-registry
ais:
  - claude
  - codex
```

И создаст симлинки выбранных групп в каталогах выбранных ИИ. Например:

```text
.claude/skills/common -> ../../skill-registry/skills/common
.claude/skills/vue    -> ../../skill-registry/skills/vue
.agents/skills/common -> ../../skill-registry/skills/common
.agents/skills/vue    -> ../../skill-registry/skills/vue
```

Профиль и ИИ можно выбрать без интерактива:

```bash
python3 skill-registry/tools/skill-registry setup \
  --profile frontend \
  --ai claude \
  --ai codex
```

Для изменения только симлинков после обновления registry выполни:

```bash
python3 skill-registry/tools/skill-registry sync
```

Скрипт читает профиль из `profiles/<profile>/manifest.yaml`, а затем создаёт ссылки на группы навыков для каждого выбранного ИИ.

## Версионирование

Используется SemVer:

- `PATCH` — исправление текста без изменения поведения;
- `MINOR` — новый навык или совместимое расширение профиля;
- `MAJOR` — изменение структуры, имени или смысла существующего навыка.

После merge создаётся тег:

```bash
git tag v1.1.0
git push origin v1.1.0
```

Проекты обновляют submodule до тега отдельными pull request.

## Проверка

```bash
python3 tools/validate-skills
```

## Единая CLI-команда

Для работы из целевого проекта можно использовать один CLI-скрипт из подключённого registry:

```bash
python3 skill-registry/tools/skill-registry <команда>
```

Доступные команды:

```bash
# Проверить registry
python3 skill-registry/tools/skill-registry validate

# Интерактивно выбрать профиль и создать .skill-registry.yaml
python3 skill-registry/tools/skill-registry profile

# Выбрать профиль без интерактивного режима
python3 skill-registry/tools/skill-registry profile --name frontend

# Перезаписать существующий файл профиля
python3 skill-registry/tools/skill-registry profile --name backend-python

# Создать или обновить симлинки
python3 skill-registry/tools/skill-registry sync
```

Команда `setup` объединяет `profile` и `sync` в один шаг.

Обычный сценарий настройки нового проекта:

```bash
git submodule update --init --recursive
python3 skill-registry/tools/skill-registry profile
python3 skill-registry/tools/skill-registry sync
```

## Версионирование

Используется SemVer: `MAJOR.MINOR.PATCH`. Проекты подключают конкретный тег, например `v1.0.0`, и обновляют его отдельным изменением.
