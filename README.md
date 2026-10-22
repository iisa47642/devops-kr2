# Контрольная работа 2 по DevOps

Исаев Иса Абдулмуслимович, ЭФБО-02-24. Номер по списку 9, вариант 1.

Приложение Flask считает посещения в Redis. Адрес на учебном стенде: https://api.devops.local.

Запрос проходит через Nginx на Ubuntu к Gunicorn в контейнере. Приложение обращается к Redis по имени `redis` в сети `kr2_backend`. Данные хранятся в томе `kr2_redis_data`.

## Файлы

- `Dockerfile` — сборка приложения, порт 8001, пользователь app.
- `docker-compose.yml` — приложение и Redis 7 Alpine.
- `nginx/kr2` — отдельный сайт Nginx с HTTPS.
- `scripts/audit.sh` — прежние 14 проверок и 5 проверок КР.
- `docs/defects.md` — ошибки исходного шаблона.
- `.env.example` — пример переменных без пароля.

Логика `app.py` и зависимости не изменены. В начало приложения добавлен только комментарий `# KR2: Isaev`.

## Запуск на VM

Проект расположен в `/home/devops/kr2`. В `.env` на сервере хранится только `REDIS_PASSWORD`, права файла — 600, владелец — devops. Пароль создан заново на сервере. Старое значение из истории шаблона не используется.

```bash
cd ~/kr2
unset REDIS_PASSWORD
docker compose up -d --build --wait
docker compose ps
curl http://127.0.0.1:8001/health
curl http://127.0.0.1:8001/
```

В `/etc/hosts` на Mac добавлено:

```text
192.168.56.101 api.devops.local
```

Сертификат сервера — `/etc/ssl/certs/kr2.crt`, ключ — `/etc/ssl/private/kr2.key`. Ключ принадлежит root и имеет права 600. На Mac скопирован только открытый сертификат `~/kr2.crt`.

```bash
curl --cacert ~/kr2.crt https://api.devops.local/
curl --cacert ~/kr2.crt https://api.devops.local/whoami
```

## Проверки

Оба процесса работают от UID 999. Redis без пароля отвечает `NOAUTH`. Порт 8001 опубликован только на `127.0.0.1`, порт 6379 не опубликован.

После `docker compose down` и повторного запуска счётчик продолжился с 4 до 5. Флаг `-v` не использовался.

Сайт `devops.local` и путь `/app/` сохранены. Файл `devops-site` и правила UFW не изменены. Расширенный аудит дал 19 успешных проверок. При неправильной публикации порта он обнаружил ошибку и вернул код 1; после исправления — код 0.

Аудит из ПР 7 остаётся в `~/devops-course-2026/scripts/audit.sh`. Расширенная версия КР хранится в этом репозитории:

```bash
sudo env HOME=/home/devops bash /home/devops/kr2/scripts/audit.sh
```

Для обычного перезапуска данные удалять не нужно. Команда `docker compose down -v` удалит том Redis.
