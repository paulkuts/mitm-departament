# Контур кафедры

Веб-приложение кафедры: учёт сотрудников, событий, статей, инвентаря и ключей, рабочие пространства и совместная работа над записями.

Продовый инстанс: <https://mitm.kutspv.ru>

## Стек

- **Бэкенд:** Go 1.23+ — Gin, `sqlx`, SQLite через `modernc.org/sqlite` (чистый Go, сборка без CGO), конфигурация — Viper (YAML), аутентификация — JWT (access + refresh в cookie).
- **Фронтенд:** нативные ES-модули без сборщика, встроены в бинарник через `go:embed`.
- **Развёртывание:** статический бинарник в образе Alpine, процесс работает под пользователем `nobody` (65534); конфиг и данные монтируются снаружи.

## Структура

```
main.go            точка входа
embed.go           встраивание frontend/*
internal/app       сборка приложения
internal/config    конфигурация (Viper)
internal/db        подключение к SQLite и миграции
internal/handler   HTTP-обработчики (Gin)
internal/models    модели данных
internal/repository доступ к БД
internal/service   бизнес-логика
internal/server    маршруты и middleware
pkg/               hasher, logger, valid, ratelimiter, circuitbreaker
frontend/          index.html, css/, js/ (ES-модули)
deploy/            Dockerfile, docker-compose.yml, шаблон конфига
scripts/           служебные скрипты разработки и smoke-тесты
```

## Запуск локально

```bash
cp deploy/configs/default.yaml configs/local.yaml   # задать свой jwt_secret и путь к БД
go run main.go                                      # http://localhost:8080
```

Проверка: `curl -s http://localhost:8080/health`

Сборка под Linux без CGO:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags "-s -w" -o departament .
```

## Запуск в Docker

Каталог `deploy/` — рецепт развёртывания: `Dockerfile` (образ Alpine, работает от `nobody`), `docker-compose.yml` (порт слушает только localhost, наружу отдаётся reverse-proxy) и `configs/default.yaml` с плейсхолдером `__JWT_SECRET__`. Каталоги `configs/`, `data/`, `export/` монтируются с хоста; секрет JWT и база данных в образ не попадают.

```bash
cd deploy
cp configs/default.yaml configs/production.yaml   # подставить реальный jwt_secret
docker compose up -d --build
```

## База данных и миграции

SQL-миграции лежат в `internal/db/migration/`, применяются автоматически при старте. Изменения вносятся **новыми** файлами `YYYYMMDDHHMMSS_описание.up.sql`; уже применённые миграции не правятся — журнал сверяет их контрольные суммы.

## Безопасность и данные

В репозиторий не попадают: `.env`, `configs/local.yaml`, `configs/production.yaml`, каталоги `data/` и `export/`, любые `*.db`. Секрет JWT задаётся на сервере при установке.

## Происхождение

Проект вырос из открытого репозитория [danrulev/departament](https://github.com/danrulev/departament): текущая версия — переработанный интерфейс «Лабораторный журнал» и функции рабочих пространств.
