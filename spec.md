# Специфікація моделі даних (Spotify)

## Сутності та атрибути

1. **User (Користувач)**
   - `user_id` (int) [PK]
   - `username` (string) 
   - `email` (string)  
   - `subscription_type` (string) 

2. **Artist (Виконавець)**
   - `artist_id` (int) [PK]
   - `user_id` (int) [FK]
   - `name` (string) [UK]
   - `bio` (string)  
   - `is_verified` (boolean) 

3. **Album (Альбом)**
   - `album_id` (int) [PK]
   - `artist_id` (int) [FK]
   - `title` (string) 
   - `release_date` (date) 
   - `album_type` (string) 

4. **Track (Трек)**
   - `track_id` (int) [PK]
   - `title` (string)   
   - `duration_ms` (number)  
   - `is_explicit` (boolean) 
   - `artist_id` (int) [FK]
   - `album_id` (int) [FK]
   - `release_date` (date) 

5. **Playlist (Плейлист)**
   - `playlist_id` (int) [PK]
   - `user_id` (int) [FK]
   - `playlist_name` (string)   
   - `description` (string)   
   - `is_public` (boolean)  
   - `created_at` (date) 

   6. **PlaylistTrack (Трек в Плейлисті)**
   - `playlist_id` (int) [PK,FK]
   - `track_id` (int) [PK,FK]
   - `position` (int)
   - `added_at` (date)


## Зв'язки

- **Artist - Album:** Один виконавець може випустити багато альбомів, і один альбом належить одному основному виконавцю (Один-до-багатьох, 1:N).
- **User - Artist:** Користувач може мати нуль або один профіль виконавця, а кожен профіль виконавця належить одному користувачу (Один-до-нуль-або-одного, 1:0..1).
- **Album - Track:** Альбом містить багато треків, кожен трек належить рівно одному альбому (Один-до-багатьох, 1:N).
- **Artist - Track:** Один трек має одного основного виконавця через `artist_id`, але також може мати додаткових виконавців для колаборацій. Один виконавець може брати участь у багатьох треках (Багато-до-багатьох, M:N для колаборацій).
- **User - Playlist:** Користувач може створити багато плейлистів, але кожен плейлист має лише одного власника-користувача (Один-до-багатьох, 1:N).
- **Playlist - PlaylistTrack:** Один плейлист може містити багато записів `PlaylistTrack`, кожен запис належить одному плейлисту (Один-до-багатьох, 1:N).
- **Track - PlaylistTrack:** Один трек може входити до багатьох записів `PlaylistTrack`, кожен запис стосується одного треку (Один-до-багатьох, 1:N).
- **Playlist - Track:** Плейлист містить багато треків, і один трек може бути у багатьох плейлистах через `PlaylistTrack` (Багато-до-багатьох, M:N).
