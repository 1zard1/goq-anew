# Анализ системы Spotify

## 1. Функциональные и нефункциональные требования

### Функциональные:
- Регистрация и аутентификация пользователей (Auth Service).
- Создание и управление плейлистами (Playlist Service).
- Генерация рекомендаций (Recommendation Service).
- Поиск контента (Search Service).
- Воспроизведение потокового контента (Streaming Service).
- Хранение и обработка пользовательских данных (User Profile Service).

### Нефункциональные:
- Высокая доступность и масштабируемость.
- Низкая задержка при запросах.
- Надёжность при передаче данных (особенно для стриминга).
- Поддержка больших объёмов данных (много пользователей и контента).
- Безопасность данных (авторизация, шифрование).
- Мониторинг и логирование (Log/Monitor Service).

---

## 2. Формализация юзер-сценариев

### Примеры:
1. **Регистрация нового пользователя**:
   - Пользователь вводит данные → запрос обрабатывается Auth Service → данные сохраняются в БД профилей.
   
2. **Авторизация пользователя**:
   - Пользователь вводит логин и пароль → запрос проходит через API Gateway → Auth Service проверяет данные.

3. **Прослушивание песни**:
   - Пользователь выбирает трек → запрос идёт к Playlist Service → Streaming Service обеспечивает передачу через CDN.

4. **Создание плейлиста**:
   - Пользователь добавляет песни в новый плейлист → данные сохраняются в Playlist Service.

5. **Поиск трека или исполнителя**:
   - Пользователь вводит запрос → Search Service обращается к ElasticSearch → возвращает результаты.

---

## 3. Примерная оценка нагрузки

- **Ежедневная аудитория**: 10 млн пользователей.
- **Количество запросов**: в среднем 5 запросов в секунду на пользователя.
- **Пиковая нагрузка**: до 50 млн запросов в секунду.
- **Размер данных**:
  - Пользовательские данные: 100 МБ на 1 млн пользователей.
  - Музыкальная библиотека: ~10 ТБ.
  - Логи и мониторинг: ~50 ГБ в день.

---

## 4. Примерная оценка требуемых ресурсов

- **БД**: масштабируемая SQL (PostgreSQL) или NoSQL (MongoDB) для профилей.
- **Распределённое хранилище** (например, S3) для контента.
- **Кэш**: Redis для быстрых операций (рекомендации, сессии).
- **Сообщения**: Kafka/RabbitMQ для передачи событий.
- **CDN**: локальные кэши в разных регионах.
- **Сервера**: ~500-1000 экземпляров при пиковых нагрузках.

---

## 5. Форматы и структуры данных

### Пример для профилей:
```json
{
  "user_id": "12345",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "playlists": ["playlist_1", "playlist_2"]
}
```
### Пример для плейлиста:
```json
{
  "playlist_id": "playlist_1",
  "user_id": "12345",
  "tracks": ["track_1", "track_2", "track_3"],
  "created_at": "2025-01-01T12:00:00Z"
}
```
## 6. Слои приложения

- **Клиентский слой**: мобильное приложение, веб-интерфейс.
- **API Gateway**: маршрутизация запросов.
- **Сервисный слой**: микросервисы для Auth, Recommendation, Playlist, Search, Streaming.
- **Слой данных**: базы данных, кэши, хранилища.
- **Инфраструктурный слой**: мониторинг, логирование, CI/CD.
- **Сервера**: ~500-1000 экземпляров при пиковых нагрузках.

## 7. Технологии общения

- **HTTP/REST**: запросы от клиента через API Gateway.
- **gRPC**: взаимодействие между микросервисами.
- **Сообщения**: Kafka/RabbitMQ для асинхронных задач (логирование, рекомендации).

## 8. Гарантии и архитектурные подходы

- **Высокая доступность**: репликация данных, отказоустойчивые CDN.
- **Масштабируемость**: горизонтальное масштабирование микросервисов.
- **Безопасность**:  OAuth 2.0, шифрование данных.
- **Архитектура**: микросервисный подход.

## 9. Хэндлеры/ручки
# API для Spotify-подобного приложения

## Основные эндпоинты

- **POST /auth/register**  
  Регистрация нового пользователя.  

- **GET /playlist/{id}**  
  Получение информации о плейлисте.  

- **POST /search**  
  Поиск по ключевым словам.  

- **GET /stream/{track_id}**  
  Стриминг трека.  

## Дополнительные эндпоинты

### Треки

- **POST /track**  
  Добавление нового трека в библиотеку.  

- **DELETE /track/{id}**  
  Удаление трека из библиотеки.  

- **GET /track/{id}/lyrics**  
  Получение текста песни, если доступно.  

### Плейлисты

- **POST /playlist/{id}/track**  
  Добавление трека в плейлист.  
  *Параметры*: ID трека и позиция в плейлисте (опционально).  

- **DELETE /playlist/{id}/track/{track_id}**  
  Удаление трека из плейлиста.  

- **PATCH /playlist/{id}**  
  Обновление информации о плейлисте (название, описание).  

- **PUT /playlist/{id}/order**  
  Изменение порядка треков в плейлисте.  
  *Параметры*: массив с ID треков в новом порядке.  

- **POST /playlist**  
  Создание нового плейлиста.  
  *Параметры*: название, описание, приватность (публичный/приватный).  

### Пользователи

- **GET /user/{id}/playlists**  
  Получение всех плейлистов пользователя.  

- **PUT /user/profile**  
  Обновление профиля пользователя (например, имя, аватар, страну).  

### Лайки и рекомендации

- **POST /like/{track_id}**  
  Лайк трека.  

- **DELETE /like/{track_id}**  
  Удаление лайка с трека.  

- **POST /recommendations**  
  Получение списка рекомендованных треков/плейлистов.  
  *Параметры*: жанры, настроение, темп и т.д.  

### История и статистика

- **GET /history**  
  Получение истории прослушанных треков.  

- **POST /track/{id}/listen**  
  Отправка информации о прослушивании трека (например, когда трек прослушан полностью).  

- **GET /stats/user/{id}**  
  Получение статистики пользователя (количество прослушанных треков, любимые жанры).  

### Стриминг и реальное время

- **POST /stream/{track_id}/like**  
  Лайк трека во время прослушивания.  

- **POST /stream/{track_id}/position**  
  Отправка текущей позиции воспроизведения трека (например, для синхронизации между устройствами).  

- **GET /stream/session**  
  Получение текущей активной стриминговой сессии.  

 

## 10. Пример кода ручки
```go
func GetPlaylistHandler(w http.ResponseWriter, r *http.Request) {
    playlistID := mux.Vars(r)["id"]
    playlist, err := GetPlaylistFromDB(playlistID)
    if err != nil {
        http.Error(w, "Playlist not found", http.StatusNotFound)
        return
    }
    json.NewEncoder(w).Encode(playlist)
}
```
#  Структура базы данных для системы Spotify

## **1. Таблица пользователей (users)**
```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
## **2. Таблица плейлистов (playlists)**
```sql
CREATE TABLE playlists (
    playlist_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## **3. Таблица треков (tracks)**
```sql
CREATE TABLE tracks (
    track_id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    artist VARCHAR(200) NOT NULL,
    album VARCHAR(200),
    duration INT NOT NULL, -- Время в секундах
    genre VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## **4. Таблица треков в плейлисте (playlist_tracks)**
```sql
CREATE TABLE playlist_tracks (
    playlist_id INT NOT NULL REFERENCES playlists(playlist_id) ON DELETE CASCADE,
    track_id INT NOT NULL REFERENCES tracks(track_id),
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (playlist_id, track_id)
);
```

## **5. Таблица истории прослушиваний (listening_history)**
```sql
CREATE TABLE listening_history (
    history_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id),
    track_id INT NOT NULL REFERENCES tracks(track_id),
    listened_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    duration INT NOT NULL -- Длительность прослушивания в секундах
);
```

## **6. Таблица рекомендаций (recommendations)**
```sql
CREATE TABLE recommendations (
    user_id INT NOT NULL REFERENCES users(user_id),
    track_id INT NOT NULL REFERENCES tracks(track_id),
    recommended_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, track_id)
);
```

## **7. Таблица сессий (sessions)**
```sql
CREATE TABLE sessions (
    session_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id),
    token VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP NOT NULL
);
```
## **Связи и пояснения**
- users → playlists: Один пользователь может иметь множество плейлистов.
- playlists → playlist_tracks → tracks: Один плейлист может содержать много треков, а один трек может находиться в нескольких плейлистах (связь многие ко многим).
- users → listening_history → tracks: История прослушиваний связывает пользователей с треками.
- users → recommendations → tracks: Система рекомендаций хранит данные о рекомендованных треках для каждого пользователя.
- users → sessions: Сессии помогают управлять аутентификацией пользователя.





