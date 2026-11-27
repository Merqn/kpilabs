# Аудит компонентної моделі Завдання 5

## Трасування компонентів до попередніх моделей

| Компонент / залежність | Трасування | Результат перевірки |
|---|---|---|
| Слухач → Web / Mobile UI | Актор «Слухач» у UC-02; NFR-3/NFR-4 | UI є межовим компонентом для клієнтських платформ, не новим актором. |
| UI → `PlaylistAPI` (`IPlaylistAPI`) | `Listener → PlaylistAPI` у Sequence; запит `addTrack` у Activity; NFR-1 | HTTPS/JSON-запит і відповідь не змінюють зовнішній контракт. |
| `PlaylistAPI` → `AuthService` (`IAuthService`) | FR-06, NFR-1; Sequence `validateOwner`; Activity-перевірка доступу | Автентифікація та власність передують операції зміни. |
| `PlaylistAPI` → `PlaylistService` (`IPlaylistUseCases`) | FR-01–FR-05; Sequence S1–S5; Activity UC-02 | API делегує бізнес-операції сервісу плейлистів. |
| `AuthService` → `PlaylistRepository` (`IPlaylistRepository`) | `Playlist.user_id` в ER; Sequence отримує власника плейлиста; гілки 403/404 у Activity | Перевірка власності читає ту саму модель даних. |
| `PlaylistAPI` → `CatalogService` (`ICatalogLookup`) | FR-02/FR-06/FR-07; Sequence S2; паралельна перевірка каталогу в Activity | API координує перевірку каталогу до делегування операції; каталог лишається внутрішнім сервісом. |
| `PlaylistService` → `PlaylistRepository` (`IPlaylistRepository`) | FR-01–FR-04; Sequence S1–S3; Activity duplicate / `max(position) + 1` | Репозиторій читає й змінює `Playlist` та `PlaylistTrack`. |
| `PlaylistService` → `CatalogService` (`ICatalogLookup`) | Sequence S1/S5: сервіс перевіряє трек; Sequence S2 та Activity: перевірку координує API | Обидва сценарні caller-и показані як споживачі одного каталожного інтерфейсу; зовнішній контракт не змінюється. |
| `CatalogService` → `TrackRepository` (`ITrackRepository`) | `Track.track_id` в ER; перевірки каталогу в Sequence й Activity | Доданий Data Access компонент деталізує доступ до локальної сутності `Track`. |
| Репозиторії → Database (`IRelationalStore`) | ER-модель; NFR-5; ADR-0001 про ACID і складений ключ | На вузлі БД явно показані ACID-транзакції, FK та `UNIQUE(playlist_id, track_id)`. |
| `AuthService` → External Identity Provider (`IIdentityProvider`) | NFR-1 вимагає автентифікації; попередні моделі починають із готової сесії | Зовнішня OIDC-інтеграція позначена припущенням, а не як уже описаний крок UC. |

## Межа системи та інтеграції

- Усередині межі: Web / Mobile UI, API, сервіси, репозиторії та реляційна БД.
- Поза межею: слухач як актор і зовнішній Identity Provider.
- Інтерфейс кожної залежності має назву, надавача й споживача; напрямок
  стрілки на діаграмі йде від споживача до надавача.
- Email/SMS, платежі й зовнішній каталог не додані: вони виключені для
  описаної потреби або дублювали б локальну сутність `Track`.

## Топ-3 розбіжності та рішення

1. **Різне місце виклику `CatalogService` між Sequence-сценаріями.**
   S2 та Activity показують перевірку на рівні API/процесної оркестрації,
   S1/S5 - усередині `PlaylistService`. Component Diagram показує обидві
   залежності від `ICatalogLookup` з позначенням сценаріїв `[S2]` і `[S1/S5]`;
   специфікація й діаграма тепер узгоджені.
2. **UI та межа системи відсутні в Sequence як окремі компоненти.**
   Додано Web / Mobile UI і межу системи, бо архітектурна модель має
   відображати NFR-3/NFR-4 та відокремлювати клієнт від backend. Послідовність
   запитів UC-02 не змінюється.
3. **Деякі архітектурні шари не деталізовані в попередніх моделях.**
   OIDC-провайдера позначено як припущення, а `TrackRepository` виділено для
   Data Access сутності `Track`; це не додає нових кроків UC-02 чи зовнішнього
   каталогу.

## Перевірка інтерфейсів і залежностей

Кожна компонентна залежність має інтерфейс із таблиці [spec.md](spec.md);
назви основних операцій підписані на стрілках, а гарантії сховища - на вузлі
БД.
`PlaylistAPI` споживає `IAuthService` та `ICatalogLookup` для S2, тоді як
`PlaylistService` споживає той самий `ICatalogLookup` для S1/S5; обидва
делегують доступ до даних відповідним репозиторіям. Платіжних, email/SMS та
неописаних бізнес-залежностей у моделі немає.
