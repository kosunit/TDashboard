# TDashboard — Trading Dashboard

Персональный торговый дашборд для отслеживания уровней по инструментам на американском рынке.

![preview](https://i.imgur.com/placeholder.png)

---

## Что делает

- Показывает свечные графики D1, H1, M5 с данными из [Twelve Data API](https://twelvedata.com)
- Отображает SPY как серую линию-ориентир на каждом графике
- Рассчитывает ATR(5) с фильтрацией паранормальных баров
- Позволяет задать ценовой уровень — он рисуется горизонтальной линией на всех трёх графиках
- Автоматически считает «Закрытие до уровня» в % от ATR D1
- Показывает текущую фазу торгов (Pre-market / Main / After-hours) с таймером
- Сектор и индустрию берёт из API автоматически

---

## Стек

| Что | Чем |
|---|---|
| Frontend | Чистый HTML + JS, без фреймворков |
| Данные | [Twelve Data](https://twelvedata.com) API |
| Графики | Canvas 2D API |
| Деплой | nginx на DigitalOcean Droplet |
| Шрифты | JetBrains Mono, Onest |

---

## Структура

```
TDashboard/
└── index.html      # всё приложение — один файл
```

---

## Запуск локально

### Вариант 1 — Python (без установки)

```bash
python3 -m http.server 8080
```

Открыть: [http://localhost:8080](http://localhost:8080)

### Вариант 2 — любой статический сервер

```bash
npx serve .
# или
npx http-server .
```

> ⚠️ Открывать через `file://` нельзя — браузер заблокирует fetch-запросы к API (CORS).

---

## Деплой на сервер

### Первоначальная настройка (Ubuntu 22.04)

```bash
# Установить nginx
apt install -y nginx

# Создать папку и настроить конфиг
mkdir -p /var/www/homework

cat > /etc/nginx/sites-enabled/default << 'EOF'
server {
    listen 80 default_server;
    root /var/www/homework;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
EOF

nginx -t && systemctl reload nginx
```

### Настройка деплоя через Git

```bash
cd /var/www/homework
git init
git remote add origin https://github.com/kosunit/TDashboard.git
git fetch origin && git reset --hard origin/main

# Создать команду deploy
echo '#!/bin/bash
cd /var/www/homework && git fetch origin && git reset --hard origin/main
echo "Deployed!"' > /usr/local/bin/deploy && chmod +x /usr/local/bin/deploy
```

### Обновление

```bash
deploy
```

---

## Как пользоваться

1. Откройте [http://139.59.142.240](http://139.59.142.240)
2. В левом списке выберите инструмент или добавьте новый (поле внизу)
3. Дождитесь загрузки графиков
4. В блоке **Уровень** введите цену уровня → линия появится на всех графиках
5. Выберите тип уровня и критерии входа
6. **Закр. до уровня** рассчитается автоматически

---

## Блок Уровень

| Поле | Описание |
|---|---|
| Цена | Вводится вручную — рисует горизонтальную линию |
| Дата | Дата закрытия последнего торгового дня |
| Тип | Излом тренда / Парабар / Лимитник + High/Low |
| Критерии | Возврат к уровню / Поджатие и ЛП / Без поджатия |
| Закр. до уровня | `|текущая цена − уровень| / ATR D1 × 100%` |

---

## ATR — формула

```
TR = Max(High − Low, |High − PrevClose|, |Low − PrevClose|)
ATR(5) = среднее последних 5 чистых TR
```

**Фильтр паранормальных баров:** если `TR > 2 × avg(TR предыдущих 20 баров)` — бар пропускается, берётся следующий предыдущий день.

---

## API ключи

Приложение использует [Twelve Data](https://twelvedata.com) — API ключ хранится прямо в `index.html`.

> При необходимости замените ключ в строке:
> ```js
> const TD_KEY = 'ваш_ключ_здесь';
> ```

---

## Кеширование

Данные кешируются в `localStorage` браузера:

| Тип данных | TTL |
|---|---|
| Цена (quote) | 1 минута |
| D1 бары | 1 час |
| H1 бары | 15 минут |
| M5 бары | 5 минут |
| Профиль (сектор/индустрия) | 24 часа |
