# Data Description

Сырые данные коммерческого проекта **не публикуются** по соглашению о конфиденциальности. 
В этом файле — описание схемы для воспроизводимости.

## Источник

PostgreSQL-таблица `raw_events` — event-level логи мобильного сервиса за период ~67 дней (~30 000 событий, ~1 400 уникальных пользователей).

## Схема таблицы `raw_events`

| Колонка | Тип | Описание |
|---|---|---|
| `event_id` | UUID | Уникальный ID события |
| `event_name` | TEXT | Тип события (см. ниже) |
| `event_time` | TIMESTAMPTZ | Время события (UTC) |
| `telegram_user_id` | BIGINT | ID пользователя (NULL для анонимных событий) |
| `session_id` | UUID | ID сессии |
| `platform` | TEXT | `telegram`, `web`, и т.д. |
| `city_id` | INT | ID города |
| `properties` | JSONB | Вложенные атрибуты события |

## Типы событий

**Воронка обмена:**
- `calculator_opened` — открыл калькулятор
- `rate_viewed` — посмотрел курс
- `order_checkout_started` — начал оформление обмена
- `order_set_amount_entered` — ввёл сумму
- `order_submitted` — отправил заявку
- `order_completed` — обмен завершён
- `order_failed` — ошибка обмена
- `order_checkout_cancelled` — отменил оформление

**Бонусы и кампании:**
- `bonus_tx_created_rate_viewed` — бонус за просмотр курса
- `bonus_tx_created_bot_start` — бонус за запуск бота
- `order_feedback_bonus_granted` — бонус за отзыв
- `wheel_spin_win` — выигрыш в колесе фортуны
- `promo_redeemed` — применил промокод
- `bonuses_page_viewed` — посмотрел страницу бонусов

**Реклама и партнёры:**
- `partner_offer_impression` — показ партнёрского оффера
- `partner_offer_click` — клик по офферу

## Поля внутри `properties` (JSONB)

Ключевые поля для feature engineering:
- `set_amount`, `base_amount`, `total_amount` — суммы обмена
- `set_currency`, `get_currency` — валюты пары
- `bonus_amount`, `total_bonus_amount` — суммы бонусов
- `balance`, `new_balance` — балансы
- `orders_count` — счётчик обменов
- `has_referral`, `referral_code` — реферальная программа
- `is_new_user` — флаг нового пользователя
- `entrypoint` — точка входа
- `rule_key` — правило срабатывания бонуса
- `rating` — оценка после обмена
- `order_status` — статус заявки

## Воспроизводимость

Для запуска ноутбука нужны переменные окружения:

```bash
export DB_PASSWORD=your_password
export DB_HOST=localhost
export DB_NAME=churn_project
```

Либо положите эти переменные в `.env` (файл уже в `.gitignore`).
