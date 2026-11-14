# ИПР ДЗ №3 (Docker)
**Писаненко Алексей БПМИ 267-1**

## Запуск
Перед запуском надо указать данные в `.evn`, в корневой папке проекта лежит `.env.example`.

Для первого запуска нужно использовать
```shell
docker compose up -d --build
```
Для остановки (с сохранением данных) нужно использовать
```shell
docker compose down
```
Взаимодействие с приложением осталось неизменным (`README.md` в папке `app` его описывает).

## Compose
Внутри `compose.yaml` указаны:
```yaml
STORAGE: redis
REDIS_URL: redis://redis:6379/0
```
Наружу публикуется только порт приложения. В `compose.yaml` также указано:
```yaml
command: [ "redis-server", "--appendonly", "yes" ]
```
Это необходимо для того, чтобы Redis записывал операции на диск.

Прописаны лимиты по потреблению памяти и CPU, корневая файловая система app доступна только для чтения.

## Проверки

**Тест (сохранение данных при перезапуске):**
```shell
docker compose up -d
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
docker compose down
docker compose up -d
curl -fsS http://127.0.0.1:8080/
```
**Результат:**
```shell
[+] up 4/4
 ✔ Volume ipr-hw-3_redis-data Created                                                                                                                                                                                                0.0s
 ✔ Network ipr-hw-3_default   Created                                                                                                                                                                                                0.0s
 ✔ Container ipr-hw-3-redis-1 Healthy                                                                                                                                                                                                3.1s
 ✔ Container ipr-hw-3-app-1   Started
 
{"greeting": "test", "count": 0, "hostname": "ab549a75a62e", "storage": "redis"}

{"greeting": "test", "count": 1, "hostname": "ab549a75a62e", "storage": "redis"}

{"greeting": "test", "count": 2, "hostname": "ab549a75a62e", "storage": "redis"}

[+] down 3/3
 ✔ Container ipr-hw-3-app-1   Removed                                                                                                                                                                                                3.2s
 ✔ Container ipr-hw-3-redis-1 Removed                                                                                                                                                                                                0.2s
 ✔ Network ipr-hw-3_default   Removed 
 
[+] up 3/3
 ✔ Network ipr-hw-3_default   Created                                                                                                                                                                                                0.0s
 ✔ Container ipr-hw-3-redis-1 Healthy                                                                                                                                                                                                3.1s
 ✔ Container ipr-hw-3-app-1   Started
 
{"greeting": "test", "count": 2, "hostname": "2092ac08ad16", "storage": "redis"}
```
**Логи:**
```shell
redis-1  | 1:C 13 Nov 2026 19:42:03.125 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
app-1    | 2026-11-13 19:42:06,366 listening port=8000 storage=redis
app-1    | 2026-11-13 19:42:09,220 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:14,316 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:19,407 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:24,504 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:29,598 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:32,356 http 192.168.65.1 "GET / HTTP/1.1" 200 -
redis-1  | 1:C 13 Nov 2026 19:42:03.125 * Redis version=7.4.12, bits=64, commit=00000000, modified=0, pid=1, just started
redis-1  | 1:C 13 Nov 2026 19:42:03.125 * Configuration loaded
redis-1  | 1:M 13 Nov 2026 19:42:03.125 * monotonic clock: POSIX clock_gettime
redis-1  | 1:M 13 Nov 2026 19:42:03.126 * Running mode=standalone, port=6379.
redis-1  | 1:M 13 Nov 2026 19:42:03.126 * Server initialized
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * Reading RDB base file on AOF loading...
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * Loading RDB produced by version 7.4.12
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * RDB age 151 seconds
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * RDB memory usage when created 0.94 Mb
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * RDB is base AOF
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * Done loading RDB, keys loaded: 0, keys expired: 0.
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * DB loaded from base file appendonly.aof.1.base.rdb: 0.000 seconds
app-1    | 2026-11-13 19:42:34,693 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:39,787 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:44,874 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:49,969 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:42:55,081 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:43:00,195 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:43:05,298 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:43:10,414 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:43:15,530 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:43:20,639 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * DB loaded from incr file appendonly.aof.1.incr.aof: 0.000 seconds
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * DB loaded from append only file: 0.001 seconds
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * Opening AOF incr file appendonly.aof.1.incr.aof on server start
redis-1  | 1:M 13 Nov 2026 19:42:03.127 * Ready to accept connections tcp
```

**Тест (кладём Redis, но не всё приложение):**
```shell
docker compose up -d
docker compose stop redis
curl -fsS http://127.0.0.1:8080/healthz
curl -fsS http://127.0.0.1:8080/readyz
curl -fsS -X POST http://127.0.0.1:8080/increment
docker compose start redis
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
```
**Результат:**
```shell
[+] up 4/4
 ✔ Network ipr-hw-3_default   Created                                                                                                                                                                                                0.0s
 ✔ Volume ipr-hw-3_redis-data Created                                                                                                                                                                                                0.0s
 ✔ Container ipr-hw-3-redis-1 Healthy                                                                                                                                                                                                3.1s
 ✔ Container ipr-hw-3-app-1   Started
 
[+] stop 1/1
 ✔ Container ipr-hw-3-redis-1 Stopped
 
{"status": "alive"}

curl: (22) The requested URL returned error: 503

curl: (22) The requested URL returned error: 503

[+] start 1/1
 ✔ Container ipr-hw-3-redis-1 Started
 
{"greeting": "test", "count": 0, "hostname": "2b290c2b0b58", "storage": "redis"}

{"greeting": "test", "count": 1, "hostname": "2b290c2b0b58", "storage": "redis"}
```
**Логи:**
```shell
redis-1  | 1:C 13 Nov 2026 19:49:14.492 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-1  | 1:C 13 Nov 2026 19:49:14.492 * Redis version=7.4.12, bits=64, commit=00000000, modified=0, pid=1, just started
redis-1  | 1:C 13 Nov 2026 19:49:14.492 * Configuration loaded
redis-1  | 1:M 13 Nov 2026 19:49:14.493 * monotonic clock: POSIX clock_gettime
redis-1  | 1:M 13 Nov 2026 19:49:14.494 * Running mode=standalone, port=6379.
redis-1  | 1:M 13 Nov 2026 19:49:14.494 * Server initialized
redis-1  | 1:M 13 Nov 2026 19:49:14.495 * Creating AOF base file appendonly.aof.1.base.rdb on server start
redis-1  | 1:M 13 Nov 2026 19:49:14.496 * Creating AOF incr file appendonly.aof.1.incr.aof on server start
redis-1  | 1:M 13 Nov 2026 19:49:14.496 * Ready to accept connections tcp
redis-1  | 1:signal-handler (1794599439) Received SIGTERM scheduling shutdown...
redis-1  | 1:M 13 Nov 2026 19:50:39.626 * User requested shutdown...
redis-1  | 1:M 13 Nov 2026 19:50:39.626 * Calling fsync() on the AOF file.
redis-1  | 1:M 13 Nov 2026 19:50:39.626 * Saving the final RDB snapshot before exiting.
redis-1  | 1:M 13 Nov 2026 19:50:39.628 * DB saved on disk
redis-1  | 1:M 13 Nov 2026 19:50:39.628 # Redis is now ready to exit, bye bye...
redis-1  | 1:C 13 Nov 2026 19:51:54.863 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-1  | 1:C 13 Nov 2026 19:51:54.863 * Redis version=7.4.12, bits=64, commit=00000000, modified=0, pid=1, just started
redis-1  | 1:C 13 Nov 2026 19:51:54.863 * Configuration loaded
redis-1  | 1:M 13 Nov 2026 19:51:54.863 * monotonic clock: POSIX clock_gettime
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * Running mode=standalone, port=6379.
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * Server initialized
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * Reading RDB base file on AOF loading...
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * Loading RDB produced by version 7.4.12
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * RDB age 160 seconds
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * RDB memory usage when created 0.94 Mb
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * RDB is base AOF
redis-1  | 1:M 13 Nov 2026 19:51:54.864 * Done loading RDB, keys loaded: 0, keys expired: 0.
redis-1  | 1:M 13 Nov 2026 19:51:54.865 * DB loaded from base file appendonly.aof.1.base.rdb: 0.000 seconds
redis-1  | 1:M 13 Nov 2026 19:51:54.865 * DB loaded from append only file: 0.000 seconds
redis-1  | 1:M 13 Nov 2026 19:51:54.865 * Opening AOF incr file appendonly.aof.1.incr.aof on server start
redis-1  | 1:M 13 Nov 2026 19:51:54.865 * Ready to accept connections tcp
app-1    | 2026-11-13 19:49:17,703 listening port=8000 storage=redis
app-1    | 2026-11-13 19:49:20,599 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:49:25,699 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:49:30,796 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:49:35,895 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:49:40,992 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:49:46,091 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:49:51,187 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:49:56,282 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:01,362 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:06,457 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:11,554 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:16,656 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:21,755 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:26,856 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:31,963 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:37,053 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:42,154 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:47,251 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:52,350 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:50:57,447 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:01,231 http 192.168.65.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:02,546 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:07,651 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:12,749 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:17,367 storage_unavailable: ConnectionError
app-1    | 2026-11-13 19:51:17,367 http 192.168.65.1 "GET /readyz HTTP/1.1" 503 -
app-1    | 2026-11-13 19:51:17,848 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:22,948 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:28,031 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:33,126 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:34,876 storage_unavailable: ConnectionError
app-1    | 2026-11-13 19:51:34,876 http 192.168.65.1 "POST /increment HTTP/1.1" 503 -
app-1    | 2026-11-13 19:51:38,224 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:43,311 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:48,409 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:53,504 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:51:58,602 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:03,707 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:08,804 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:13,913 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:16,602 http 192.168.65.1 "GET / HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:19,018 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:24,127 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:29,225 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:34,318 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:36,435 http 192.168.65.1 "POST /increment HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:39,410 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:44,507 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:49,609 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:54,718 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
app-1    | 2026-11-13 19:52:59,821 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
```

## Автопроверка
Через `./scripts/check.sh` запускается скрипт с автопроверкой работоспособности приложения. Перед этим нужно права доступа для исполнения файла дать. (На маке `chmod +x scripts/check.sh `).

**Тест (проверка скрипта):**
```shell
./scripts/check.sh
```
**Результат:**
```shell
[+] Building 2.6s (13/13) FINISHED                                                                                                                                                                                                       
 => [internal] load local bake definitions                                                                                                                                                                                          0.0s
 => => reading from stdin 538B                                                                                                                                                                                                      0.0s
 => [internal] load build definition from Dockerfile                                                                                                                                                                                0.0s 
 => => transferring dockerfile: 285B                                                                                                                                                                                                0.0s 
 => [internal] load metadata for docker.io/library/python:3.13-slim-bookworm                                                                                                                                                        2.4s 
 => [internal] load .dockerignore                                                                                                                                                                                                   0.0s
 => => transferring context: 80B                                                                                                                                                                                                    0.0s 
 => [1/6] FROM docker.io/library/python:3.13-slim-bookworm@sha256:a5a7017efc5969e00ae2b91c8eb0631049a569652aa4ffc4501769049ae45c72                                                                                                  0.0s 
 => [internal] load build context                                                                                                                                                                                                   0.0s 
 => => transferring context: 94B                                                                                                                                                                                                    0.0s 
 => CACHED [2/6] WORKDIR /app                                                                                                                                                                                                       0.0s 
 => CACHED [3/6] COPY app/requirements.txt .                                                                                                                                                                                        0.0s 
 => CACHED [4/6] RUN pip install --no-cache-dir -r requirements.txt                                                                                                                                                                 0.0s 
 => CACHED [5/6] COPY app/app.py .                                                                                                                                                                                                  0.0s 
 => CACHED [6/6] RUN mkdir /data && chown 10001:10001 /data                                                                                                                                                                         0.0s 
 => exporting to image                                                                                                                                                                                                              0.0s 
 => => exporting layers                                                                                                                                                                                                             0.0s 
 => => writing image sha256:9ffea1cb7e8b3638e966d91d2ab776b4ad03bf339cbb6ed716096ce6df02a635                                                                                                                                        0.0s 
 => => naming to docker.io/library/ipr-hw-3-check-1766-app                                                                                                                                                                          0.0s 
 => resolving provenance for metadata file                                                                                                                                                                                          0.0s 
[+] up 5/5
 ✔ Image ipr-hw-3-check-1766-app         Built                                                                                                                                                                                       2.7s
 ✔ Network ipr-hw-3-check-1766_default   Created                                                                                                                                                                                     0.0s
 ✔ Volume ipr-hw-3-check-1766_redis-data Created                                                                                                                                                                                     0.0s
 ✔ Container ipr-hw-3-check-1766-redis-1 Healthy                                                                                                                                                                                     3.1s
 ✔ Container ipr-hw-3-check-1766-app-1   Started                                                                                                                                                                                     3.2s
ok: начальный count = 0
{"greeting": "test", "count": 1, "hostname": "da192c0c0442", "storage": "redis"}{"greeting": "test", "count": 2, "hostname": "da192c0c0442", "storage": "redis"}{"greeting": "test", "count": 3, "hostname": "da192c0c0442", "storage": "redis"}{"greeting": "test", "count": 4, "hostname": "da192c0c0442", "storage": "redis"}{"greeting": "test", "count": 5, "hostname": "da192c0c0442", "storage": "redis"}ok: count после 5 POST = 5
[+] up 1/1
 ✔ Container ipr-hw-3-check-1766-app-1 Started                                                                                                                                                                                       3.4s
ok: count после пересоздания app = 5
[+] stop 1/1
 ✔ Container ipr-hw-3-check-1766-redis-1 Stopped                                                                                                                                                                                     0.1s
ok: /healthz без Redis = 200
ok: /readyz без Redis = 503
ok: POST без Redis = 503
[+] start 1/1
 ✔ Container ipr-hw-3-check-1766-redis-1 Started                                                                                                                                                                                     0.1s
ok: count после запуска Redis = 5
{"greeting": "test", "count": 6, "hostname": "4fff5f386baa", "storage": "redis"}ok: count после ещё одного POST = 6
[+] stop 2/2
 ✔ Container ipr-hw-3-check-1766-redis-1 Stopped                                                                                                                                                                                     0.1s
 ✔ Container ipr-hw-3-check-1766-app-1   Stopped                                                                                                                                                                                     3.1s
Проверка успешно пройдена.
[+] down 6/6
 ✔ Container ipr-hw-3-check-1766-redis-1 Removed                                                                                                                                                                                     0.0s
 ✔ Container ipr-hw-3-check-1766-app-1   Removed                                                                                                                                                                                     0.0s
 ✔ Volume ipr-hw-3-check-1766_redis-data Removed                                                                                                                                                                                     0.0s
 ✔ Network ipr-hw-3-check-1766_default   Removed                                                                                                                                                                                     0.1s
 ✔ Image redis:7-alpine                  Removed                                                                                                                                                                                     0.0s
 ✔ Image ipr-hw-3-check-1766-app:latest  Removed
```