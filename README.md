# amnezia_web

Открытая **базовая** веб-панель для **просмотра** клиентов **AmneziaWG** на своём VPS: таблица клиентов, статус «в туннеле», AllowedIPs, несколько инстансов через **`AWG_PROFILES`**, часы сервера и браузера. В редакции **FREE/community** можно **удалить** клиента с сервера (peer и строка в таблице).  
**Нет** в интерфейсе и по API в FREE: включение/выключение peer, правка дат отключения, переименование, экспорт `.conf`, «Новый клиент под каскад», Cloudflare WARP, синхронизация времени хоста по SSH — это **[версия PRO](https://boosty.to/andrey27/purchase/3906453?ssource=DIRECT&share=subscription_link)** (приватный репозиторий **amnezia_web-PRO**, доступ подписчикам).

Редакция **`community`** задаётся автоматически файлом **`.amnezia-panel-edition`** в корне репозитория (`community`) или переменной окружения **`AMNEZIA_EDITION=community`** в контейнере. Кнопка и текст про подписку настраиваются **`COMMUNITY_UPGRADE_URL`** и **`COMMUNITY_UPGRADE_PITCH`**.

**Безопасность:** доступ к Docker-сокету в контейнере панели эквивалентен root на хосте — используйте сложный пароль и ограничьте доступ по IP / TLS. Поле установки PRO по GitHub‑токену (`ALLOW_COMMUNITY_GITHUB_ACTIVATION=1`) отключено по умолчанию: при включении любой авторизованный администратор может запускать произвольный `install.sh` на хосте через Docker‑сокет.

Справочник по типичным сбоям и API (в т.ч. для PRO): в полной документации репозитория PRO.

---

## Установка одной командой

```bash
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/install.sh | sudo bash
```

Форк или ветка:

```bash
GITHUB_REPO=вы/репо BRANCH=main curl -fsSL https://raw.githubusercontent.com/вы/репо/main/scripts/install.sh | sudo bash
```

Уже скачали проект:

```bash
cd /opt/amnezia-admin && chmod +x scripts/install.sh && sudo SKIP_DOWNLOAD=1 bash scripts/install.sh
```

### Переменные окружения и `sudo`

Если передаёте **`AWG_PROFILES`** или **`ADMIN_PASSWORD`** в одной строке с `curl`, используйте **`sudo -E bash`**, иначе `sudo` не увидит переменные. Альтернатива — записать JSON профилей в **`/root/amnezia-admin.awg-profiles.json`** и запустить обычный `curl … | sudo bash`. Подробнее см. историю коммитов и зеркальный README в репозитории PRO.

### Важные переменные

| Переменная | По умолчанию | Назначение |
|------------|--------------|------------|
| `GITHUB_REPO` | `andrey271192/amnezia_web` | Архив для установки |
| `HOST_PORT` | `8080` | Порт панели |
| `AWG_CONTAINER` | `amnezia-awg2` | Контейнер WG по умолчанию |
| `AWG_PROFILES` | _(нет)_ | Несколько инстансов; см. примеры в репозитории PRO |
| `COMMUNITY_UPGRADE_URL` | Страница подписки PRO на Boosty (зашита в код по умолчанию) | Переопределите, если смените уровень подписки |
| `COMMUNITY_UPGRADE_PITCH` | _(текст по умолчанию в коде)_ | Текст под заголовком базовой версии |
| `SKIP_LANDING` | `0` | `1` — без лендинга на порту 80 |
| `ALLOW_COMMUNITY_GITHUB_ACTIVATION` | `0` | `1` — в редакции community показывать ввод GitHub-токена и запускать приватный `install.sh` (доступ к репо с Contents Read / repo; **высокая чувствительность**, см. раздел безопасности) |
| `COMMUNITY_PRIVATE_INSTALL_SCRIPT_URL` | `https://raw.githubusercontent.com/andrey271192/amnezia_web-pro/main/scripts/install.sh` | Сырой URL `scripts/install.sh` в **вашем** приватном репозитории PRO |
| `COMMUNITY_INSTALL_FETCH_MS` | `60000` | Таймаут HTTP при скачивании установщика из GitHub |
| `PRIVATE_INSTALL_SCRIPT_MAX_BYTES` | `2097152` | Максимальный размер скачанного скрипта (байты) |
| `COMMUNITY_INSTALL_LOG_TAIL_BYTES` | `98304` | Первый ответ **`GET /api/community/install-log`** без `since` отдаёт **хвост** последних N байт (для живого блока в UI при установке PRO). Диапазон 1 KiB … 512 KiB через env. |
| `COMMUNITY_INSTALL_LOG_API_CHUNK_MAX` | `524288` | Максимум байт **за один** запрос «догонять» журнал (~512 KiB; верхний предел через env ограничен). |
| `COMMUNITY_DISABLE_DOCKER_CLI_HELPER` | _(не задано)_ | Если `1`, установка из UI снова только «bash внутри панели» (часто конфликт **:8080** с PRO). По умолчанию используется одноразовый контейнер **docker:cli**, который **снимает FREE** и зависший PRO перед `install.sh`. |
| `COMMUNITY_SKIP_REMOVE_FREE_BEFORE_PRIVATE_PRO` | `0` | `1` — не выполнять `docker rm` FREE/лендинга перед install (если знаете, что делаете). |
| `COMMUNITY_PRO_INSTALL_HELPER_IMAGE` | `docker:26-cli` | Образ с бинарём `docker` для фонового контейнера (нужен доступ к демону по сокету). |
| `COMMUNITY_HELPER_SKIP_PREPARE_TOOLS` | _(не задано)_ | `1` — не ставить `curl` перед приватным `install.sh` в helper (обычно не нужно; образ Alpine часто без `curl`, без него ваш скрипт падает). |
| `FREE_PANEL_CONTAINER_FOR_PRO_INSTALL` и др. | см. `server.js` | Имена контейнеров для `docker rm` перед install (по умолчанию `amnezia-admin`, `amnezia-web-landing`, `amnezia-admin-pro`). |

Чтобы включить форму установки из панели, задайте в контейнере **`AMNEZIA_EDITION=community`** и **`ALLOW_COMMUNITY_GITHUB_ACTIVATION=1`** (`install.sh`: добавьте `-e ALLOW_COMMUNITY_GITHUB_ACTIVATION=1` или пересоздайте контейнер после правки переменных). URL приватного `install.sh` при необходимости переопределите через **`COMMUNITY_PRIVATE_INSTALL_SCRIPT_URL`**.

**Важно:** при запуске `curl … | bash` переменная должна быть **экспортирована**, иначе её не увидит `install.sh`:

```bash
export ALLOW_COMMUNITY_GITHUB_ACTIVATION=1
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/install.sh | sudo -E bash
```

(Под не-root добавьте `sudo`; `sudo -E` сохраняет экспортированные переменные.)

Остальные переменные совместимы с образом панели (см. Dockerfile / `server.js` в этом репозитории).

---

## Обновление

```bash
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/install.sh | sudo bash
```

После сообщения **`→ Клонирование релиза …`** несколько минут **может ничего не печататься**: идёт `curl … | tar` с GitHub. Для **полосы прогресса**: `sudo INSTALL_SCRIPT_VERBOSE=1 bash` (или передайте переменную в окружение до `|`). Если обрывает по времени или «тишина» часами задайте таймауты: **`CURL_CONNECT_TIMEOUT`** (по умолчанию 30 с к подключению), **`CURL_MAX_TIME`** (по умолчанию до 900 с на загрузку), **`CURL_RETRY`**. Полный URL tar.gz можно подменить: **`GITHUB_REPO_URL_OVERRIDE`** (должен указывать на архив того же вида **`${REPO}-${BRANCH}.tar.gz`**, после распаковки каталог будет **`ИмяРепозитория-${BRANCH}`**).

Принудительная пересборка образа: **`NO_CACHE=1`**.

---

## Удаление

```bash
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/uninstall.sh | sudo bash
```

---

## Частые проблемы

### Пропало поле «токен GitHub» под плашкой FREE

Форма показывается только если в контейнере **`ALLOW_COMMUNITY_GITHUB_ACTIVATION=1`** (и редакция **community**). Переменную нужно хотя бы один раз задать с **`export`** (см. выше). Начиная с актуального **`install.sh`**, при следующем обновлении панели **`ALLOW_COMMUNITY_*`**, **`COMMUNITY_UPGRADE_*`**, **`AMNEZIA_EDITION`** и ряд других значений **подтягиваются из предыдущего контейнера `amnezia-admin`**, чтобы не терять настройку.

После успешной кнопки **«Установить PRO»** в той же панели открывается **живой блок «Журнал установки»** (поллинг каждые 2 с к **`GET /api/community/install-log`**; для запросов нужна сессия в браузере). Пока старый контейнер FREE снят и новый образ ещё не поднял ту же вкладку, поток журнала в браузере **прерывается** — откройте адрес панели снова после старта PRO; файл **`community-install-last.log`** на томе данных тот же. Кнопка **«Пауза»** останавливает опрос журнала.

### `Bind for 0.0.0.0:8080 failed: port is already allocated` у `amnezia-admin-pro`

На **8080** уже слушает **FREE**‑панель (**`amnezia-admin`**). Пока она работает, второй контейнер панели (PRO) на тот же порт не поднимется. **Кнопка «Установить PRO» из UI по умолчанию** использует одноразовый контейнер **`docker:cli`** (пауза → **`docker rm`** FREE / лендинг / **`amnezia-admin-pro` и любые имена вида `_…_…-amnezia-admin-pro`** из docker compose → затем ваш **`install.sh`**). Иначе вручную: **`docker rm -f amnezia-admin`**, затем установщик PRO. Отключить авто-снос: **`COMMUNITY_DISABLE_DOCKER_CLI_HELPER=1`**.

### В `community-install-last.log`: **`curl: command not found`** и затем **`tar: invalid magic`**

Образ **`docker:26-cli`** на Alpine часто **без `curl`**; ваш приватный `install.sh`, скачивающий архив через `curl … | tar`, падает, а **`tar`** читает пустые данные («invalid magic»). Актуальная панель **перед запуском** `install.sh` в helper ставит **`curl`** через **`apk`** / **`apt-get`**. Если сетевая политика блокирует `apk add`/`apt-get`, укажите другой образ в **`COMMUNITY_PRO_INSTALL_HELPER_IMAGE`** (со встроенным `curl`), либо замените `curl` в своём **`install.sh`** на **`wget`**. Отключить авто-доставку `curl`: **`COMMUNITY_HELPER_SKIP_PREPARE_TOOLS=1`** (обычно не нужно).

### Установка «зависла» на сообщении **`→ Клонирование релиза …`** и долго без вывода

Это этап **скачивания и распаковки** архива с GitHub. Запустите установщик с **`INSTALL_SCRIPT_VERBOSE=1`** (полоска загрузки `curl`). Задайте **`CURL_MAX_TIME`** если нужно усечь ожидание. При блокировках GitHub с хоста — зеркальный полный URL в **`GITHUB_REPO_URL_OVERRIDE`** (см. раздел «Обновление»).

### Лендинг: порт 80 занят

Если на хосте уже что-то слушает **TCP 80**, установщик сообщит об этом и, при дефолтном **`LANDING_PORT=80`**, сам попробует другой свободный порт (обычно начиная с **8081**). Либо явно: **`LANDING_PORT=8083`**, либо **`SKIP_LANDING=1`**.

### AmneziaWG Legacy: удаление клиента не срабатывает (`wg0.conf is world accessible` / `No such device`)

Панель синхронизирует живой интерфейс через **`wg-quick strip` + `syncconf`**. У Legacy конфиг часто **`/opt/amnezia/awg/wg0.conf`**, интерфейс — **`wg0`**, не **`awg0`**. В актуальном коде интерфейс выводится из **имени файла** (**`wg0.conf` → `wg0`**). Если путь другой — задайте в профиле **`AWG_IFACE`** / в **`AWG_PROFILES`** поле **`iface`**. Предупреждение про world-accessible: перед применением конфига файл на стороне контейнера выставляется в **`chmod 600`**.

---

## Лицензия

MIT — см. [LICENSE](LICENSE).
