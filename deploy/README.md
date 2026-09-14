## Развёртывание

Прод: <https://mitm.kutspv.ru> — reverse-proxy с TLS → `127.0.0.1:8085` → контейнер `departament-mitm`.

- Рецепт развёртывания — каталог `deploy/`: `Dockerfile`, `docker-compose.yml`, шаблон `configs/default.yaml`. Бинарник статический: `CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags "-s -w"`.
- Конфиг и секрет JWT в образ не входят: `configs/` монтируется в контейнер только для чтения, `jwt_secret` подставляется на сервере при установке.
- Данные (`data/`, `export/`) живут только на сервере — в образ и в репозиторий не попадают.
- База разворачивается пустой: переноса данных нет.
- Первый администратор создаётся вручную: зарегистрировать аккаунт на сайте, затем `UPDATE users SET role='admin' WHERE id=<id>` в базе.
- Refresh-cookie выставляется с флагом `Secure`, поэтому приложение доступно только через HTTPS reverse-proxy (при локальных HTTP-тестах cookie не отправляется).
