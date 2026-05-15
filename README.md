# amnezia_web

Открытая **базовая** веб-панель для **просмотра** клиентов **AmneziaWG** на своём VPS: таблица клиентов, статус «в туннеле», AllowedIPs, несколько инстансов через **`AWG_PROFILES`**, часы сервера и браузера. В редакции **FREE/community** можно **удалить** клиента с сервера (peer и строка в таблице).  
**Нет** в интерфейсе и по API в FREE: включение/выключение peer, правка дат отключения, переименование, экспорт `.conf`, «Новый клиент под каскад», Cloudflare WARP, синхронизация времени хоста по SSH — это **[версия PRO](https://boosty.to/andrey27/purchase/3906453?ssource=DIRECT&share=subscription_link)** (приватный репозиторий **amnezia_web-PRO**, доступ подписчикам).

**Редакция по умолчанию в этом репозитории — FREE (**`community`**, файл **`.amnezia-panel-edition`**). Чтобы клиент мог ввести GitHub-токен и запустить **«Установить PRO»**, используйте **только установку из раздела ниже:** **`INSTALL_FREE_COMMUNITY_ACTIVATION=1`** (создаётся признак **`/opt/amnezia-admin-data/.install-free-github-pro-opt-in`**; следующие апдейты сохранят эту возможность без повторного экспорта). Кнопка Boosty и текст настраиваются **`COMMUNITY_UPGRADE_URL`** и **`COMMUNITY_UPGRADE_PITCH`**.

**Безопасность:** доступ к Docker-сокету в контейнере панели эквивалентен root на хосте — используйте сложный пароль и ограничьте доступ по IP / TLS. Режим **«Установить PRO» из панели** (`ALLOW_COMMUNITY_GITHUB_ACTIVATION=1`): любой авторизованный администратор может запускать приватный `install.sh` на хосте через Docker-сокет. Отключить позже — удалите **`${DATA_DIR}/.install-free-github-pro-opt-in`** и перезапустите установщик **без** `INSTALL_FREE…`, переменную **`ALLOW_COMMUNITY_GITHUB_ACTIVATION`** из контейнера не переопределяйте экспортом.

Справочник по типичным сбоям и API (в т.ч. для PRO): в полной документации репозитория PRO.

---

## Установка для клиента (FREE с «Установить PRO» по GitHub)

Рекомендуемый вариант: сразу **FREE-панель** с полем **GitHub-токена** под баннером Boosty и кнопкой **«Установить PRO»**:

```bash
export INSTALL_FREE_COMMUNITY_ACTIVATION=1
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/install.sh | sudo -E bash
```

Переменная **`INSTALL_FREE_COMMUNITY_ACTIVATION`** сохранится в томе как файл **`.install-free-github-pro-opt-in`** в **`/opt/amnezia-admin-data`**: дальше достаточно обычного **`curl … | sudo bash`** при обновлении (можно без `export`).

---

## Прочее: установка без поля GitHub в панели

Если **специально** не нужно поле для токена (только просмотр FREE и удаление клиентов, без апгрейда из UI):

```bash
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/install.sh | sudo bash
```

Форк или ветка:

```bash
export GITHUB_REPO="вы/репо"
export BRANCH="main"
export INSTALL_FREE_COMMUNITY_ACTIVATION=1
curl -fsSL "https://raw.githubusercontent.com/${GITHUB_REPO}/${BRANCH}/scripts/install.sh" | sudo -E bash
```

Если нужен только FREE **без** поля для токена, не задавайте **`INSTALL_FREE…`** и не используйте **`-E`** (достаточно `sudo bash`).

Уже скачали проект:

```bash
cd /opt/amnezia-admin && chmod +x scripts/install.sh && sudo SKIP_DOWNLOAD=1 bash scripts/install.sh
```

### Переменные окружения и `sudo`

Если передаёте **`AWG_PROFILES`** или **`ADMIN_PASSWORD`** в одной строке с `curl`, используйте **`sudo -E bash`**, иначе `sudo` не увидит переменные. Альтернатива — записать JSON профилей в **`/root/amnezia-admin.awg-profiles.json`** и запустить обычный `curl … | sudo bash`. Подробнее см. историю коммитов и зеркальный README в репозитории PRO.

### Важные переменные

| Переменная | По умолчанию | Назначение |
|------------|--------------|------------|
| `INSTALL_FREE_COMMUNITY_ACTIVATION` | _(не задано)_ | `1` (**один раз** при установке клиента при необходимости): **`AMNEZIA_EDITION=community`** + **`ALLOW_COMMUNITY_GITHUB_ACTIVATION=1`** + маркер в **`${DATA_DIR}/.install-free-github-pro-opt-in`** для следующих запусков установщика. |
| `GITHUB_REPO` | `andrey271192/amnezia_web` | Архив для установки |
| `HOST_PORT` | `8080` | Порт панели |
| `AWG_CONTAINER` | `amnezia-awg2` | Контейнер WG по умолчанию |
| `AWG_PROFILES` | _(нет)_ | Несколько инстансов; см. примеры в репозитории PRO |
| `COMMUNITY_UPGRADE_URL` | Страница подписки PRO на Boosty (зашита в код по умолчанию) | Переопределите, если смените уровень подписки |
| `COMMUNITY_UPGRADE_PITCH` | _(текст по умолчанию в коде)_ | Текст под заголовком базовой версии |
| `PANEL_FOOTER_DONATE_URL` | Как **`COMMUNITY_UPGRADE_URL`** | Ссылка «донат» в **шапке и подвале** веб‑панели (**не** добавляется на nginx‑лендинг пользователей). |
| `PANEL_FOOTER_TELEGRAM_URL` | `https://t.me/lot_andrey` | Ссылка Telegram‑спонсора / канала (там же: только панель). |
| `PANEL_FOOTER_OZON_URL` | Ссылка на **Ozon СБП** автора (дефолт в `server.js`) | Донат через Ozon — **шапка и подвал** панели с остальными ссылками. |
| `PANEL_FOOTER_DONATE_LABEL` | `Поддержать проект` | Текст якоря доната |
| `PANEL_FOOTER_OZON_LABEL` | `Донат · Ozon СБП` | Текст якоря Ozon |
| `PANEL_FOOTER_TELEGRAM_LABEL` | `Telegram‑канал спонсора` | Текст якоря Telegram |
| `PANEL_FOOTER_PROMO_SUBTITLE` | _(строка по умолчанию в коде)_ | Поясняющий текст в верхней полосе панели |
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
| `COMMUNITY_HELPER_SKIP_PREPARE_TOOLS` | _(не задано)_ | `1` — helper не ставит **`bash`** и **`curl`** в образе Alpine (обычно **не нужно**; без них `#!/usr/bin/env bash` или `curl` в PRO `install.sh` падают). |
| `MTPRO_PROXY_CONTAINER` | `mtproto-proxy` | Имя контейнера MTProto‑прокси Telegram (официальный образ **telegrammessenger/proxy**, см. Dockerfile на Docker Hub). |
| `MTPRO_PROXY_IMAGE` | `telegrammessenger/proxy:latest` | Образ **`docker pull` + `docker run`** при установке из панели (FREE/community). |
| `MTPRO_INTERNAL_PORT` | `443` | Порт процесса **внутри** контейнера (официальный прокси слушает 443/tcp). Мапится на **`MTPRO_PUBLISH_*`**. |
| `MTPRO_PUBLISH_PORT` | `8443` | Порт хоста VPS (**внешний**), проброшенный в контейнер. |
| `MTPRO_PUBLISH_BIND` | `0.0.0.0` | Адрес биндинга на хосте (`-p` в Docker); при нескольких сетевых интерфейсах при необходимости уточняют. |
| `MTPRO_PUBLIC_HOST` | _(нет)_ | Публичный IP или DNS для ссылки **`tg://proxy`** в UI; можно вместо него задать **`CLIENT_CONFIG_ENDPOINT`**. |
| `UI_HIDE_MTPROTO` | _(не задано)_ | `1` или **`UI_HIDE_SECTIONS=...,mtproto`** — скрыть раздел установки MTProto в веб‑интерфейсе (по умолчанию виден даже FREE). |
| `FREE_PANEL_CONTAINER_FOR_PRO_INSTALL` и др. | см. `server.js` | Имена контейнеров для `docker rm` перед install (по умолчанию `amnezia-admin`, `amnezia-web-landing`, `amnezia-admin-pro`). |

Примечание для MTProto на панели: **`GET /api/mtproto/status`** отдаёт состояние и **`tg://` без синхронного `docker logs`**; хвост логов подгружается вторым запросом **`GET /api/mtproto/logs`**, чтобы блок со ссылкой открывался быстрее.

Чтобы включить форму установки из панели после «обычной» установки без **`INSTALL_FREE…`**, задайте в контейнере **`AMNEZIA_EDITION=community`** и **`ALLOW_COMMUNITY_GITHUB_ACTIVATION=1`** (или проще переустановите с **`INSTALL_FREE_COMMUNITY_ACTIVATION=1`**). URL приватного `install.sh` при необходимости переопределите через **`COMMUNITY_PRIVATE_INSTALL_SCRIPT_URL`**.

**Важно:** при одноразовом запуске с переменными в одной строке с `curl … | sudo bash` они **пропадают** — нужен **`export`** перед `curl` и **`sudo -E bash`**, см. блок **«Установка для клиента»** выше.

Остальные переменные совместимы с образом панели (см. Dockerfile / `server.js` в этом репозитории).

---

## Обновление

Клиентская панель (после первой установки с **`INSTALL_FREE_COMMUNITY_ACTIVATION=1`** маркер в томе подхватывается автоматически):

```bash
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/install.sh | sudo bash
```

Или явно всегда **`export INSTALL_FREE_COMMUNITY_ACTIVATION=1`** и **`sudo -E bash`** как при первой установке.

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

Нужны **`ALLOW_COMMUNITY_GITHUB_ACTIVATION=1`** и редакция **community** (в образе они попадают из **`install.sh`**). Рекомендуется раздел **«Установка для клиента»**: **`INSTALL_FREE_COMMUNITY_ACTIVATION=1`** (создаётся **`…/data/.install-free-github-pro-opt-in`**; дальнейшие **`curl … | sudo bash`** подхватят это сами). При ручном отключении убедитесь, что файл-маркер удалён. Начиная с актуального **`install.sh`**, при апдейте панели многие **`COMMUNITY_*`**, **`ALLOW_COMMUNITY_*`**, **`AMNEZIA_EDITION`** восстанавливаются из **предыдущего** контейнера `amnezia-admin`; маркер в томе остаёт запасным вариантом, если переменная в контейнере пропала.

После успешной кнопки **«Установить PRO»** в той же панели открывается **живой блок «Журнал установки»** (поллинг каждые 2 с к **`GET /api/community/install-log`**; для запросов нужна сессия в браузере). Пока старый контейнер FREE снят и новый образ ещё не поднял ту же вкладку, поток журнала в браузере **прерывается** — откройте адрес панели снова после старта PRO; файл **`community-install-last.log`** на томе данных тот же. Кнопка **«Пауза»** останавливает опрос журнала.

В режиме **docker-helper** панель пишет в **`community-install-last.log`** финальную строку **`--- helper завершён code=…`** **внутри** одноразового контейнера: её видно даже если процесс FREE-панели убрал **`docker rm`** и Node не успевает добавить строку **`--- завершено …`**. Если в логе после преамбулы одноразового установщика **нет** ни одной финальной строки — обновите образ панели с актуального **`amnezia_web`** и повторите установку.

### `Bind for 0.0.0.0:8080 failed: port is already allocated` у `amnezia-admin-pro`

На **8080** уже слушает **FREE**‑панель (**`amnezia-admin`**). Пока она работает, второй контейнер панели (PRO) на тот же порт не поднимется. **Кнопка «Установить PRO» из UI по умолчанию** использует одноразовый контейнер **`docker:cli`** (пауза → **`docker rm`** FREE / лендинг / **`amnezia-admin-pro` и любые имена вида `_…_…-amnezia-admin-pro`** из docker compose → затем ваш **`install.sh`**). Иначе вручную: **`docker rm -f amnezia-admin`**, затем установщик PRO. Отключить авто-снос: **`COMMUNITY_DISABLE_DOCKER_CLI_HELPER=1`**.

### В `community-install-last.log`: **`curl: command not found`**, **`tar: invalid magic`**, **`code=2` сразу после helper

Образ **`docker:26-cli`** на Alpine обычно **без `bash` и без `curl`**. PRO `install.sh` с **`#!/usr/bin/env bash`** и загрузкой архива через **`curl | tar`** без них не стартуют (**`curl: command not found`**, **`tar: invalid magic`**). Если после строки «одноразовый установщик» почти сразу **`--- завершено … code=2`** — возможны ошибка сборки prelude в **`sh`**, ошибка **`docker run … -e TOKEN=…`** из‑за спецсимволов в PAT, или **apk** не смог добавить **`bash`** (сеть из контейнера). Обновите панель с **`main`**: helper целиком шлёт **stdout/stderr в лог**, токены передаются средой процесса (без интерполяции в аргументе `-e`), prelude разделён **`;`**; перед **`install.sh`** скрипт сообщает если **нет bash/curl** (**выход по коду ~126**). Если **apk недоступен** — свой **`COMMUNITY_PRO_INSTALL_HELPER_IMAGE`** с **`bash` и `curl`**, либо в приватном `install.sh` перейти на **`#!/bin/sh`** и **`wget`**. Отключить подготовку: **`COMMUNITY_HELPER_SKIP_PREPARE_TOOLS=1`**.

### Установка «зависла» на сообщении **`→ Клонирование релиза …`** и долго без вывода

Это этап **скачивания и распаковки** архива с GitHub. Запустите установщик с **`INSTALL_SCRIPT_VERBOSE=1`** (полоска загрузки `curl`). Задайте **`CURL_MAX_TIME`** если нужно усечь ожидание. При блокировках GitHub с хоста — зеркальный полный URL в **`GITHUB_REPO_URL_OVERRIDE`** (см. раздел «Обновление»).

### `SKIP_DOWNLOAD=1 bash scripts/install.sh` завершается сразу без вывода («нифига» не происходит)

В старых версиях под **`set -o pipefail`** подсчёт контейнеров через **`grep`** падал, если **нет ни одного запущенного имени `amnezia-awg…`**; отдельно тот же эффект давал пайплайн **`docker port … | head | awk`** при **остановленном** `amnezia-admin`: `docker inspect` ещё «видит» контейнер, а **`docker port` возвращает ошибку**. Скрипт обрывался до **«Сборка образа»**. Обновите **`scripts/install.sh`** до актуального из `main` или временно выполните **`docker rm -f amnezia-admin`** перед установкой.

### Лендинг: порт 80 занят

Если на хосте уже что-то слушает **TCP 80**, установщик сообщит об этом и, при дефолтном **`LANDING_PORT=80`**, сам попробует другой свободный порт (обычно начиная с **8081**). Либо явно: **`LANDING_PORT=8083`**, либо **`SKIP_LANDING=1`**.

### AmneziaWG Legacy: удаление клиента не срабатывает (`wg0.conf is world accessible` / `No such device`)

Панель синхронизирует живой интерфейс через **`wg-quick strip` + `syncconf`**. У Legacy конфиг часто **`/opt/amnezia/awg/wg0.conf`**, интерфейс — **`wg0`**, не **`awg0`**. В актуальном коде интерфейс выводится из **имени файла** (**`wg0.conf` → `wg0`**). Если путь другой — задайте в профиле **`AWG_IFACE`** / в **`AWG_PROFILES`** поле **`iface`**. Предупреждение про world-accessible: перед применением конфига файл на стороне контейнера выставляется в **`chmod 600`**.

---

## Лицензия

MIT — см. [LICENSE](LICENSE).
