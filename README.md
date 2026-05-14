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

Принудительная пересборка образа: **`NO_CACHE=1`**.

---

## Удаление

```bash
curl -fsSL https://raw.githubusercontent.com/andrey271192/amnezia_web/main/scripts/uninstall.sh | sudo bash
```

---

## Лицензия

MIT — см. [LICENSE](LICENSE).
