# TapTap [README-DEPLOY.md](https://github.com/user-attachments/files/27299891/README-DEPLOY.md)
# Pro Clicker → APK через PWABuilder

Этот пакет — готовая PWA-версия Pro Clicker. Дальше нужно (1) выложить файлы на HTTPS-хостинг, (2) скормить URL [PWABuilder](https://pwabuilder.com), (3) скачать готовый подписанный APK.

## Что в пакете

```
index.html                  ← твоя игра + регистрация service worker
manifest.json               ← PWA-манифест (имя, цвета, иконки)
service-worker.js           ← оффлайн-кэш (без него PWABuilder не даст APK)
icon-192.png                ← обычная иконка
icon-512.png                ← обычная иконка (для splash)
icon-192-maskable.png       ← иконка с safe-zone (для круглых масок Android)
icon-512-maskable.png       ← иконка с safe-zone
```

Что добавлено к твоему `index.html`: только один блок в конце `window.addEventListener('load', ...)` — регистрация service worker. Геймплей не тронут, сохранения совместимы.

---

## Шаг 1. Выложить на HTTPS

PWABuilder работает только с публичным HTTPS-URL. Самые простые варианты — оба бесплатные:

### Вариант A — GitHub Pages (рекомендую)

1. Создай репозиторий на github.com, например `pro-clicker`.
2. Загрузи в корень все 7 файлов из этого пакета.
3. Settings → Pages → Source: `Deploy from a branch` → Branch: `main` / `/ (root)` → Save.
4. Через 1–2 минуты получишь URL вида `https://<твой-логин>.github.io/pro-clicker/`.
5. Открой этот URL — игра должна работать. **Обязательно** проверь, что Service Worker регистрируется: открой DevTools (F12) → вкладка Application → Service Workers → должен быть зелёный «activated and is running».

### Вариант B — Netlify Drop (без git, в один клик)

1. Зайди на https://app.netlify.com/drop
2. Перетащи всю папку с файлами в окно браузера.
3. Получишь URL вида `https://<random-name>.netlify.app`.

### Вариант C — Cloudflare Pages, Vercel, Firebase Hosting

Любой из них тоже подойдёт — лишь бы был HTTPS.

---

## Шаг 2. Прогнать через PWABuilder

1. Открой https://www.pwabuilder.com/
2. Вставь свой URL → **Start**.
3. Подожди, пока он проверит манифест, SW и иконки. Должны быть зелёные галочки. Если что-то жёлтое (warning) — обычно это не критично. Если красное — см. раздел «Если ругается» ниже.
4. Жми **Package For Stores** → **Android**.
5. В диалоге выбери **Other Android** (если нет планов в Google Play) — он сразу даст подписанный APK. Если планируешь Play Store — выбирай **Google Play**, тогда получишь AAB + ключ подписи.
6. Поля заполни:
   - **Package ID:** `com.proclicker.game` (как в твоём промте)
   - **App name:** `Pro Clicker`
   - **Launcher name:** `Pro Clicker`
   - **App version:** `1.0.0`
   - **Version code:** `1`
   - **Display mode:** `standalone`
   - **Status bar color / Nav color:** `#0f172a`
   - Остальное оставь по умолчанию.
7. **Generate** → скачается `.zip`. Внутри будет `app-release-signed.apk` — это и есть готовый файл для установки на Android.

---

## Шаг 3. Установить APK на устройство

1. Перекинь APK на телефон (Telegram «Saved Messages», Google Drive, USB — что удобнее).
2. На телефоне открой файл — Android попросит разрешить установку из неизвестных источников для того приложения, через которое ты открываешь APK. Разреши.
3. Установи. Готово.

> Под капотом PWABuilder использует Trusted Web Activity (TWA) — это легальный способ обернуть PWA в нативное Android-приложение. APK получается ~3 МБ, открывает игру в полноэкранном Chrome Custom Tab. Для пользователя выглядит как обычное приложение.

---

## Если ругается

| Симптом | Что проверить |
|---|---|
| PWABuilder не находит manifest | URL манифеста должен открываться напрямую в браузере (`<твой-домен>/manifest.json`) и отдавать JSON, а не HTML 404. |
| «Service worker not found» | Открой `<домен>/service-worker.js` в браузере — должен отдать JS, а не 404. На GitHub Pages это бывает, если выложил в подпапку. |
| «Icons missing» | Иконки тоже должны открываться по прямым URL. Проверь регистр: `icon-192.png` ≠ `Icon-192.png`. |
| APK устанавливается, но открывает белый экран | Service worker не успел зарегистрироваться при первом запуске. Закрой и открой приложение ещё раз — на втором запуске уже есть кэш. |
| TWA показывает адресную строку Chrome сверху | Нужна верификация через Digital Asset Links. Без неё TWA работает, но с минибаром. Для дома/тестов это ок; для Play Store надо настроить `assetlinks.json` — PWABuilder сам подскажет, как. |

---

## Обновление игры

После релиза правь `index.html` и пушь в тот же репозиторий. Чтобы у игроков подтянулся новый код, **в `service-worker.js` поменяй** `CACHE_VERSION = 'pro-clicker-v1'` на `'v2'`, `'v3'` и т.д. Без этого старый кэш будет жить вечно и игроки не увидят апдейтов.

Сам APK пересобирать не надо — TWA каждый раз дёргает свежую версию с твоего хостинга.
