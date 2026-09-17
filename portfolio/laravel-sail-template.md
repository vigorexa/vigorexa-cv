# Laravel Sail Template

Стартовый шаблон для Laravel-проектов: базовый Docker-образ, локальный Laravel Sail, модульная структура, авторизация, Filament, production-ready стеки и GitLab CI в одном репозитории.

**Репозиторий:** https://github.com/vigorexa/laravel-sail-template

Спроектировал и разработал шаблон с нуля для ускорения повторяющихся этапов в разработке: установка и конфигурация базовых пакетов, разработка процесса авторизации, деплой.
Большой упор делал на соблюдение [Twelve-Factor](https://12factor.net/ru/) прямо из шаблона. 

На основе этого шаблона и его предыдущих версий были созданы десятки приложений и сервисов.
Что позволило стандартизировать процесс создания проектов и сократить сотни часов разработки.

Форк шаблона внутри компании также содержит методы аутентификации через Password провайдер пакета `laravel/passport`. 
А также .stub файлы, AI скиллы и правила для генерации [Laravel JSON:API](https://laraveljsonapi.io/) схем.

Этот файл - краткое человекопонятное описание шаблона исключительно для портфолио. 
Более подробно написано в `../../../readme.md` репозитория 

---

## Основные изменения
- Сборка основана на Laravel Sail — [документация](https://laravel.com/docs/12.x/sail). Базовый образ приложения: [`vigorexa/laravel-sail-core:php85-alpine-slim-latest`](https://hub.docker.com/r/vigorexa/laravel-sail-core)
- Предустановлен фреймворк для создания админ панелей Filament [https://filamentphp.com/docs](https://filamentphp.com/docs/4.x/panels/installation)
- Предустановлена система модулей [`nwidart/laravel-modules`](https://laravelmodules.com/docs/13) с кастомной конфигурацией и stub файлами.
- Предустановлены базовые пакеты:
    - [`laravel/pint`](https://laravel.com/docs/12.x/pint). Конфигурация: https://raw.githubusercontent.com/vigorexa/pint-config/refs/heads/main/pint.json
    - [`laravel/octane`](https://laravel.com/docs/12.x/octane). Сборка готова к использованию с сервером Swoole
    - [`laravel/boost`](https://laravel.com/docs/12.x/boost). Написаны кастомные скиллы.
    - А также [`laravel/horizon`](https://laravel.com/docs/12.x/horizon), [`laravel/telescope`](https://laravel.com/docs/12.x/telescope)
- Созданы docker-compose файлы инфраструктуры и приложения для деплоя проекта на удаленный сервер.
- Добавлен `../../../project.gitlab-ci.yml` с lint > build > test > deploy стадиями для develop и prod окружений.
- Модуль AccessControl с базовым управлением Пользователями, Ролями и Правами. Основа: [spatie/laravel-permission](https://spatie.be/docs/laravel-permission/v7)

---

## Разработка

Для локальной разработки используется `laravel/sail`. `../../../docker-compose.yml` поднимает приложение, PostgreSQL 17, KeyDB и Mailpit. 
Образ собирается из `./docker/laravel/Dockerfile`, базовый слой — [`vigorexa/laravel-sail-core`](https://hub.docker.com/r/vigorexa/laravel-sail-core) на Alpine.

Скрипты упрощающие локальную разработку расположены в Makefile

### Laravel Boost

Шаблон позволяет гораздо эффективнее использовать AI инструменты за счет точной настройки `laravel/boost`.
Этот пакет публикует MCP сервер и генерирует генеральные гайдлайны для выбранных агентов из установленных библиотек.

Также в шаблоне есть кастомные Skills и Rules для модульной архитектуры проекта.

### Модульная система

Шаблон заточен под использование модульной системы [`nwidart/laravel-modules`](https://laravelmodules.com/docs/13).
Это позволяет разделять функционал на независимые части, упростить поддержку и проще переиспользовать части других приложений.

Модуль **Access Control** содержит в себе основу для Идентификации и Авторизации через систему Ролей/Прав [`spatie/laravel-permission`](https://spatie.be/docs/laravel-permission/v7).


---

## Инфраструктура и деплой

Каталог `deployment/` — набор Swarm-стеков, которые поднимаются независимо: Traefik, PostgreSQL (+ pgAdmin и exporter), KeyDB, Mailpit, RustFS (S3), monitoring (Prometheus, Grafana, Loki, OTEL, Alloy, cAdvisor), само приложение (Octane, Horizon, cron).

Один родительский `deployment/.env`; для каждого стека `sh-process-env.sh` подставляет переменные в `../../../.env.example` сервиса, `sh-process-compose-file.sh` готовит compose под `docker stack deploy`. 
Стеки разнесены, взаимодействуют через docker сети.

Сборка образа laravel разделена на development и production target'ы. Dev зависимости не передаются в production. 


### CI/CD

`../../../project.gitlab-ci.yml` копируется в `.gitlab-ci.yml` с заполнением объявленных переменных. 
После этого пайплайн полностью готов.

Настроен для триггеров: merge-request, push в master или production ветки. И состоит из стадий:
- `lint` - проверка кодстайл через pint
- `build` - сборка docker образа и публикация его в registry с тегом \$CI_REGISTRY_IMAGE:\$CI_COMMIT_SHA
- `test` - прогон автотестов на собранном образе
- `deploy` - публикация готового образа на dev/prod окружении

---

## Полный стек

- **Backend:** Laravel 12, PHP 8.5, Filament 4, Nwidart Modules, Laravel Octane (Swoole), Horizon
- **Хранение данных:** PostgreSQL 17, KeyDB, RustFS (S3-compatible) 
- **Инфраструктура:** Docker, Docker Swarm, Traefik, GitLab CI/CD
- **Наблюдаемость:** Grafana, Prometheus, Loki, OpenTelemetry, Alloy
- **Разработка:** Laravel Boost, XDebug, Mailpit, Laravel Pint, PHPUnit
