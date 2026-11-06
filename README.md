# МЕГАПОЛИС — версия 0.1

Первая серверная основа Telegram Mini App: профиль игрока, баланс, ресурсы, строительство и сбор производства.

## Важно
- Игра рассчитана на запуск внутри Telegram.
- Вход проверяется по `Telegram.WebApp.initData` на сервере.
- Не включайте тестовый обход авторизации на публичном сервере.
- Это стартовый прототип, не финальная 3D-игра. Город пока изображён CSS-сценой; полноценный WebGL/Three.js город добавим отдельным этапом.

## Состав
- `public/` — интерфейс Mini App
- `src/index.js` — API и серверная проверка Telegram
- `migrations/0001_init.sql` — таблицы D1
- `wrangler.jsonc` — настройки Cloudflare Worker


## Рекомендуемый способ с Android: GitHub Codespaces

Codespaces — это редактор и терминал в облаке, поэтому Node.js и Wrangler запускаются не на телефоне, а на сервере GitHub.

1. Создайте личный аккаунт на https://github.com/ .
2. Создайте пустой репозиторий `megapolis-game` (можно Public или Private).
3. Откройте репозиторий → Add file → Upload files и загрузите ZIP-архив проекта. Нажмите Commit changes.
4. Откройте `https://github.com/codespaces` → New codespace → выберите репозиторий `megapolis-game` и ветку `main`.
5. Когда откроется редактор VS Code в браузере, откройте Terminal → New Terminal. Распакуйте архив:
```bash
unzip MEGAPOLIS_v0.1_Android_Starter.zip
cd megapolis_v01
npm install -g wrangler
npx wrangler login
```
6. Перейдите в Cloudflare и создайте аккаунт: https://dash.cloudflare.com/ .
7. В терминале Codespaces создайте базу:
```bash
npx wrangler d1 create megapolis-db
```
8. Команда покажет `database_id`. Откройте `wrangler.jsonc` в редакторе Codespaces, найдите `PASTE_DATABASE_ID_HERE` и замените только этот текст на свой Database ID. Сохраните файл.
9. Создайте таблицы:
```bash
npx wrangler d1 execute megapolis-db --remote --file=migrations/0001_init.sql
```
10. Создайте у @BotFather бота через `/newbot` и скопируйте токен. Не отправляйте токен в чат и не добавляйте его в файлы проекта.
11. Добавьте токен в Cloudflare Worker как секрет:
```bash
npx wrangler secret put BOT_TOKEN
```
Вставьте токен в терминал, когда появится приглашение.
12. Опубликуйте игру:
```bash
npx wrangler deploy
```
13. В конце появится адрес вида `https://megapolis-game.<твой-аккаунт>.workers.dev`. Скопируйте его.
14. В @BotFather откройте `/mybots` → ваш бот → Bot Settings → Menu Button → Configure Menu Button. Вставьте адрес Worker и задайте подпись `Играть`.
15. Откройте бота в Telegram и нажмите `Играть`.

### Как не выйти за бесплатный лимит Codespaces
- Выбирайте машину на 2 ядра.
- После работы остановите Codespace через меню `...` → Stop codespace.
- В личном GitHub Free включено 120 core-hours и 15 GB-month хранилища в месяц (для 2-ядерной машины это до 60 часов работы). После исчерпания бесплатного лимита работа приостанавливается, если не добавлять способ оплаты. Источник: https://docs.github.com/en/billing/concepts/product-billing/github-codespaces .

## Быстрый запуск через Cloudflare Dashboard (без компьютера)

1. Создайте аккаунт на https://dash.cloudflare.com/ .
2. Откройте Workers & Pages → Create → Create Worker. Назовите Worker `megapolis-game`.
3. Для полноценной сборки удобнее загрузить этот проект в GitHub, затем подключить репозиторий в Cloudflare Workers & Pages. Если Cloudflare предлагает выбрать тип проекта, выберите Worker с конфигурацией Wrangler.
4. В Cloudflare D1 создайте базу `megapolis-db`.
5. Скопируйте Database ID в `wrangler.jsonc` вместо `PASTE_DATABASE_ID_HERE`.
6. Создайте D1 таблицы, открыв базу → Console, и выполните весь SQL из `migrations/0001_init.sql`.
7. В настройках Worker добавьте Secret с именем `BOT_TOKEN` и значением токена, полученного у @BotFather. Не публикуйте токен и не вставляйте его в HTML/JavaScript.
8. Убедитесь, что у Worker есть D1 binding с именем `DB`, привязанный к `megapolis-db`. В `wrangler.jsonc` это уже описано; при настройке через Dashboard значения должны совпадать.
9. Разместите содержимое папки `public` как статические assets, а `src/index.js` как Worker entry point. Если используете GitHub-интеграцию, проверьте, что сборка использует `wrangler.jsonc`.
10. Откройте URL Worker. Сам по себе браузер может показать сообщение, что игра запускается только из Telegram — это ожидаемо.
11. В @BotFather выполните `/mybots` → выберите бота → Bot Settings → Menu Button → Configure Menu Button. Укажите URL Worker и название `Играть`.
12. Откройте бота и нажмите кнопку меню.

## Настройка Wrangler (если появится возможность запускать команды)
```bash
npx wrangler d1 create megapolis-db
# вставьте полученный database_id в wrangler.jsonc
npx wrangler d1 execute megapolis-db --remote --file=migrations/0001_init.sql
npx wrangler secret put BOT_TOKEN
npx wrangler deploy
```

## Проверка
- Откройте Mini App из кнопки меню Telegram.
- Новый игрок должен получить стартовый баланс 25 000 ₵.
- Постройте лесопилку, подождите несколько минут и нажмите «Собрать».
- Перезагрузка приложения не должна сбрасывать баланс и здания.

## Что реализовано в v0.1
- Telegram initData validation (HMAC-SHA-256).
- Автоматическое создание профиля.
- Баланс, древесина, камень, энергия.
- Четыре типа зданий.
- Серверная проверка стоимости и списание средств.
- Производство с ограничением офлайн-накопления 8 часов.

## Что ещё не реализовано
- Общий рынок, сделки и контракты.
- Полноценная 3D-сцена и камера.
- Админ-панель и модерация.
- Защита от ботов на уровне масштабного публичного запуска.
- Резервное копирование и аналитика.
