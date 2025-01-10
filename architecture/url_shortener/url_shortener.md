# Анализ системы сокращения URL (URL Shortener)

## 1. Функциональные и нефункциональные требования

### Функциональные:
- Генерация коротких URL из длинных (Tiny URL Creator).
- Получение исходного URL по короткому адресу.
- Удаление устаревших URL.
- Поддержка пользовательской статистики и аналитики.

### Нефункциональные:
- Высокая доступность (система должна работать 24/7).
- Высокая пропускная способность (миллионы запросов в день).
- Низкая задержка (время обработки запроса < 100 мс).
- Масштабируемость для поддержки растущего числа пользователей.
- Надёжность (гарантия сохранения данных URL).
- Поддержка горизонтального масштабирования базы данных.

---

## 2. Формализация юзер-сценариев

### Примеры:
1. **Создание короткого URL**:
   - Пользователь отправляет длинный URL → запрос обрабатывается через API Gateway → Tiny URL Creator генерирует короткую ссылку → данные сохраняются в Shard SQL DB.

2. **Получение длинного URL по короткому**:
   - Пользователь отправляет короткий URL → запрос проходит через кэш (Cache) или Shard SQL DB → возвращается оригинальная ссылка.

3. **Удаление короткого URL**:
   - Система автоматически удаляет устаревшие или неиспользуемые ссылки через Tiny URL Creator/Deleter.

4. **Сбор аналитики**:
   - Логи запросов отправляются в распределённую очередь (Distributed Log/Queue) → аналитический сервис обрабатывает данные.

---

## 3. Примерная оценка нагрузки

- **Общее количество URL**: ~200 млн записей.
- **Количество запросов**: до 1 млн запросов в день.
- **Размер данных**:
  - Средний URL ~200 байт.
  - Общий объём базы данных: ~40 ГБ.
- **Пиковая нагрузка**:
  - 10 000 запросов/секунда на чтение.
  - 1 000 запросов/секунда на запись.

---

## 4. Примерная оценка требуемых ресурсов

- **База данных**: шардированная SQL база данных (например, PostgreSQL или MySQL).
- **Кэш**: LRU-кэш (Redis) для хранения часто используемых записей.
- **Сообщения**: Kafka/RabbitMQ для логов и аналитики.
- **Сервера**: 
  - Генерация коротких URL: ~20 серверов.
  - Кэш: ~5 серверов Redis.
  - База данных: ~10 шардов + реплики.

---

## 5. Форматы и структуры данных

### Пример для хранения URL:
```json
{
  "short_url": "abc123",
  "long_url": "https://example.com/very/long/url",
  "created_by": "user_id_123",
  "created_at": "2025-01-01T12:00:00Z",
  "expire_at": "2026-01-01T12:00:00Z"
}
```
## 6. Слои приложения

- **Клиентский слой**: веб или мобильное приложение для создания/использования коротких ссылок.
- **API Gateway**: маршрутизация запросов (LB, TLS, rate-limiting).
- **Сервисный слой:**: Tiny URL Creator/Deleter, Tiny URL Getter.
- **Слой данных:**: Кэш (Redis). База данных (шардированная SQL).
- **Инфраструктурный слой**: мониторинг, логирование, аналитика.

## 7. Технологии общения

- **HTTP/REST**: взаимодействие клиента с API Gateway.
- **gRPC**: общение между микросервисами.
- **Сообщения**: Kafka/RabbitMQ для обработки логов.

## 8. Гарантии и архитектурные подходы

- **Высокая доступность**: репликация данных, отказоустойчивость через шардинг.
- **Скорость работы**: использование кэша Redis для популярных URL.
- **Архитектурный подход:**: микросервисная архитектура с возможностью горизонтального масштабирования.
- **Безопасность**: авторизация и шифрование.

## 9. Хэндлеры/ручки

- **POST /long_link**: создание короткой ссылки.
- **GET /{short_url}**: получение длинного URL.
- **DELETE /{short_url}:**: удаление короткой ссылки.
- **GET /analytics/{short_url}**: получение аналитики по ссылке.

## 10. Пример кода

### Пример хэндлера на Go:
```go
func CreateShortURLHandler(w http.ResponseWriter, r *http.Request) {
    var req struct {
        LongURL  string `json:"long_url"`
        CreatedBy string `json:"created_by"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid input", http.StatusBadRequest)
        return
    }

    shortURL := GenerateShortURL(req.LongURL)
    if err := SaveToDB(shortURL, req.LongURL, req.CreatedBy); err != nil {
        http.Error(w, "Database error", http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(map[string]string{"short_url": shortURL})
}

```
# База Данных

## 1. Таблица urls (Хранение длинных и коротких URL)
```sql
CREATE TABLE urls (
    id SERIAL PRIMARY KEY,
    short_url VARCHAR(10) UNIQUE NOT NULL,  -- Короткий идентификатор (например, abc123)
    long_url TEXT NOT NULL,                -- Исходный длинный URL
    created_by VARCHAR(50),                -- Идентификатор пользователя, если есть
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- Время создания
    expire_at TIMESTAMP                    -- Дата истечения срока действия (опционально)
);
```

## 2. Таблица users (Хранение данных пользователей, если предусмотрена авторизация)
```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL, -- Хранение хэша пароля
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 3. Таблица analytics (Хранение статистики использования ссылок)
```sql
CREATE TABLE analytics (
    id SERIAL PRIMARY KEY,
    short_url VARCHAR(10) NOT NULL REFERENCES urls(short_url) ON DELETE CASCADE,
    accessed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- Время доступа
    client_ip VARCHAR(45),                          -- IP-адрес клиента
    user_agent TEXT,                                -- Информация о браузере/устройстве
    referrer TEXT                                   -- Реферальный источник
);
```

## 3. Таблица url_locks (Защита от гонок при генерации коротких URL)
```sql
CREATE TABLE url_locks (
    id SERIAL PRIMARY KEY,
    short_url VARCHAR(10) UNIQUE NOT NULL,
    locked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
## Связи и пояснения
- urls:
  - Хранит короткие и длинные ссылки.
  - Поле expire_at позволяет автоматически удалять устаревшие ссылки.

- users:
  - Предназначена для авторизации, если система поддерживает регистрацию.

- analytics:
  - Сохраняет данные о каждом доступе к короткому URL.
  - Используется для генерации пользовательской статистики.

- url_locks:
  - Помогает синхронизировать операции по генерации коротких URL (например, при использовании Redlock).


