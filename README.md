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

Добавь registry как submodule:

```bash
git submodule add git@github.com:company/skill-registry.git skill-registry
git -C skill-registry checkout v1.0.0
git add .gitmodules skill-registry
git commit -m "chore: подключить реестр навыков"
```

### Выбор профиля

Профиль выбирается в файле `.skill-registry.yaml` целевого проекта. Доступны:

- `frontend` — общие навыки и Vue;
- `backend-python` — общие навыки и Python;
- `backend-elixir` — общие навыки и Elixir;
- `fullstack` — общие навыки, Vue, Python и Elixir.

Например, для Vue-проекта создай файл:

```yaml
profile: frontend
registry_path: skill-registry
skills_path: .skills
```

Для Python-проекта значение будет другим:

```yaml
profile: backend-python
registry_path: skill-registry
skills_path: .skills
```

Значение `profile` должно совпадать с именем каталога в `profiles/`.

После клонирования проекта выполни:

```bash
git submodule update --init --recursive
python3 tools/sync-skills
```

Скрипт читает `profile`, открывает соответствующий `profiles/<profile>/manifest.yaml` и создаёт относительные ссылки `.skills/<группа>` на группы навыков, перечисленные в этом manifest. Каталог `.skills` можно добавить в `.gitignore`.

Чтобы переключить проект на другой профиль, измени только `profile` и повторно запусти:

```bash
python3 tools/sync-skills
```

Например:

```bash
sed -i.bak 's/^profile: frontend$/profile: fullstack/' .skill-registry.yaml
python3 tools/sync-skills
rm .skill-registry.yaml.bak
```

После переключения в `.skills` появятся группы `common`, `vue`, `python` и `elixir`.

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
python3 skill-registry/tools/skill-registry profile --name backend-python --force

# Создать или обновить симлинки
python3 skill-registry/tools/skill-registry sync
```

Обычный сценарий настройки нового проекта:

```bash
git submodule update --init --recursive
python3 skill-registry/tools/skill-registry profile
python3 skill-registry/tools/skill-registry sync
```

## Версионирование

Используется SemVer: `MAJOR.MINOR.PATCH`. Проекты подключают конкретный тег, например `v1.0.0`, и обновляют его отдельным изменением.

Подробности находятся в `docs/`.
