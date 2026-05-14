# amnezia_web

Открытая **базовая** веб-панель для **просмотра** клиентов **AmneziaWG** на своём VPS: таблица клиентов, статус «в туннеле», AllowedIPs, несколько инстансов через **`AWG_PROFILES`**, часы сервера и браузера.  
**Нет** в интерфейсе и по API: включение/выключение peer, правка дат отключения, переименование, удаление, экспорт `.conf`, «Новый клиент под каскад», Cloudflare WARP, синхронизация времени хоста по SSH — это **[версия PRO](https://boosty.to/andrey27/donate)** (приватный репозиторий **amnezia_web-PRO**, доступ подписчикам).

Редакция **`community`** задаётся автоматически файлом **`.amnezia-panel-edition`** в корне репозитория (`community`) или переменной окружения **`AMNEZIA_EDITION=community`** в контейнере. Кнопка и текст про подписку настраиваются **`COMMUNITY_UPGRADE_URL`** и **`COMMUNITY_UPGRADE_PITCH`**.

**Безопасность:** доступ к Docker-сокету в контейнере панели эквивалентен root на хосте — используйте сложный пароль и ограничьте доступ по IP / TLS.

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
| `COMMUNITY_UPGRADE_URL` | Boosty автора | Куда ведёт кнопка «Разблокировать PRO» |
| `COMMUNITY_UPGRADE_PITCH` | _(текст по умолчанию в коде)_ | Текст под заголовком базовой версии |
| `SKIP_LANDING` | `0` | `1` — без лендинга на порту 80 |

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
