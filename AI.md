## 1
- Запит: я вибрав домен spotify.com напиши мені readme файли як написані в умові завдання
- Відповідь: Домен: Базовий музичний сервіс (спрощена модель)

Опис домену

Цей проект моделює мінімальну структуру даних для музичного каталогу та створення плейлистів.

Система дозволяє зберігати інформацію про виконавців, їхні альбоми та окремі треки. З іншого боку, користувачі можуть реєструватися в системі та створювати власні плейлисти, додаючи туди улюблені треки.

Ключові процеси

Каталог: Збереження базової ієрархії музики (Виконавець випускає Альбоми, Альбом містить Треки).

Плейлисти: Користувач може створювати власні добірки музики (плейлисти) і додавати туди будь-які треки з каталогу.

## 2
- Запит: Як найкраще оформити сутність у spec.md
- Відповідь: # Специфікація моделі даних (Spotify)

## Сутності та атрибути

**User (Користувач)**
   - `id` (int) [PK]

   - `username` (string)

   - `email` (string)  

   - `subscription_type` (string)

2. **Artist (Виконавець)**

   - `id` (int) [PK]

   - `name` (string)

   - `bio` (string)  

   - `is_verified` (boolean)

3. **Album (Альбом)**

   - `id` (int) [PK]

   - `title` (string)

   - `release_date` (date)

   - `album_type` (string)

4. **Track (Трек)**

   - `id` (int) [PK]

   - `title` (string)  

   - `duration_ms` (number)  

   - `is_explicit` (boolean)

5. **Playlist (Плейлист)**

   - `id` (int) [PK]

   - `title` (string)  

   - `description` (string)  

   - `is_public` (boolean)  

## Зв'язки

**Artist - Album:** Один виконавець може випустити багато альбомів, і один альбом належить одному основному виконавцю (Один-до-багатьох, 1:N).
**Album - Track:** Альбом містить багато треків, кожен трек належить рівно одному альбому (Один-до-багатьох, 1:N).
**Artist - Track:** Один трек може мати кілька виконавців (колаборація), а виконавець має багато треків (Багато-до-багатьох, M:N).
**User - Playlist:** Користувач може створити багато плейлистів, але кожен плейлист має лише одного власника-користувача (Один-до-багатьох, 1:N).
**Playlist - Track:** Плейлист містить багато треків, і один трек може бути у багатьох плейлистах (Багато-до-багатьох, M:N).
**User - Track (Liked Tracks):** Користувач може вподобати багато треків, і трек може бути вподобаний багатьма користувачами (Багато-до-багатьох, M:N).


## 3
- Запит: напиши мне код для er діаграми за моїм spec.md
- Відповідь: erDiagram
    %% Зв'язки (1:N)
    Artist ||--o{ Album : "releases"
    Artist ||--o{ Track : "main_artist"
    Album ||--|{ Track : "contains"
    User ||--o{ Playlist : "creates"

    %% Зв'язки (M:N - чистий багато-до-багатьох)
    Artist }o--o{ Track : "collaborates"
    Playlist }o--o{ Track : "includes"
    User }o--o{ Track : "likes"

    %% Сутності
    User {
        int user_id PK
        string username
        string email
        string subscription_type
    }

    Artist {
        int artist_id PK
        string name
        string bio
        boolean is_verified
    }

    Album {
        int album_id PK
        int artist_id FK
        string title
        date release_date
        string album_type
    }

    Track {
        int track_id PK
        int artist_id FK
        int album_id FK
        string title
        int duration_ms
        boolean is_explicit
        date release_date
    }

    Playlist {
        int playlist_id PK
        int user_id FK
        string playlist_name
        string description
        boolean is_public
        date created_at
    }
