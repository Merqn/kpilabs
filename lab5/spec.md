# SPEC - Завдання 5: Архітектура UC-02

## Межі та призначення моделі

Компонентна модель описує архітектуру музичного стримінгового сервісу в
частині UC-02 «Організувати особистий плейлист»: створення плейлиста,
додавання треку та відмови для недопустимих операцій. Усередині межі системи
розміщено клієнтський інтерфейс, API, сервіси, компоненти доступу до даних і
реляційну БД. Слухач і зовнішній Identity Provider розташовані поза нею.

Діаграма побудована як Mermaid flowchart із компонентами та явно названими
provided/required-інтерфейсами. Стрілка залежності йде від споживача
інтерфейсу до компонента, який його надає. Це одна декларативна компонентна
модель; інші UML-нотації паралельно не використовуються.

Артефакт: [Mermaid source](artifact/playlist-component.mmd) |
[SVG render](artifact/playlist-component.svg).

## Компоненти

| Компонент | Відповідальність | Інтерфейси |
|---|---|---|
| Web / Mobile UI | Дає слухачеві змогу створювати плейлисти й додавати треки; підтримує платформи з NFR-3/NFR-4. | Requires `IPlaylistAPI`. |
| `PlaylistAPI` | Приймає HTTPS/JSON-запити, перевіряє сесію/власника та маршрутизує каталогову перевірку відповідно до сценарію: для S2 виконує її перед делегуванням, для S1/S5 її виконує `PlaylistService`. | Provides `IPlaylistAPI`; requires `IAuthService`, `ICatalogLookup`, `IPlaylistUseCases`. |
| `AuthService` | Перевіряє сесію та право власності на плейлист. | Provides `IAuthService`; requires `IIdentityProvider`, `IPlaylistRepository`. |
| `PlaylistService` | Виконує бізнес-операції створення плейлиста й додавання треку, включно з перевіркою каталогу для S1/S5, перевіркою дубліката та транзакційним записом. | Provides `IPlaylistUseCases`; requires `IPlaylistRepository`, `ICatalogLookup`. |
| `CatalogService` | Перевіряє наявність `Track` за `track_id`. | Provides `ICatalogLookup`; requires `ITrackRepository`. |
| `PlaylistRepository` | Створює й читає `Playlist`, читає власника, перевіряє `PlaylistTrack`, обчислює позицію та виконує зміни в транзакції. | Provides `IPlaylistRepository`; requires `IRelationalStore`. |
| `TrackRepository` | Читає дані `Track` із реляційного сховища. | Provides `ITrackRepository`; requires `IRelationalStore`. |
| Relational Database | Зберігає `User`, `Artist`, `Album`, `Track`, `Playlist`, `PlaylistTrack`; забезпечує ACID, зовнішні ключі та унікальність пари `(playlist_id, track_id)`. | Provides `IRelationalStore`. |
| External Identity Provider | Перевіряє identity token для автентифікованої сесії. | Provides `IIdentityProvider` через OIDC. Розташований поза межею системи. |

## Контракти інтерфейсів

| Інтерфейс | Надавач | Споживач(і) | Контракт |
|---|---|---|---|
| `IPlaylistAPI` | `PlaylistAPI` | Web / Mobile UI | HTTPS/JSON: `createPlaylist`, `addTrack`; транспорт відповідає NFR-1. |
| `IAuthService` | `AuthService` | `PlaylistAPI` | `validateSession`, `validateOwner`: перевірка автентифікації та права власності до зміни плейлиста. |
| `IPlaylistUseCases` | `PlaylistService` | `PlaylistAPI` | `createPlaylist`, `addTrack`: бізнес-операції UC-02. |
| `IIdentityProvider` | External Identity Provider | `AuthService` | OIDC-перевірка identity token. Це архітектурне припущення; випуск токена й інтерактивний login не входять до UC-02. |
| `IPlaylistRepository` | `PlaylistRepository` | `AuthService`, `PlaylistService` | Отримання власника, `createPlaylist`, читання `PlaylistTrack` за складеним ключем, поточна позиція та транзакційний `addAtPosition`. |
| `ICatalogLookup` | `CatalogService` | `PlaylistAPI` (S2), `PlaylistService` (S1/S5) | `findTrack(trackId)` повертає наявний або відсутній трек; виклик розташовано відповідно до сценарію. |
| `ITrackRepository` | `TrackRepository` | `CatalogService` | `findTrackById(trackId)` читає `Track` за ідентифікатором. |
| `IRelationalStore` | Relational Database | `PlaylistRepository`, `TrackRepository` | Реляційні читання й записи, зовнішні ключі, унікальність пари `PlaylistTrack` та ACID-транзакції. |

## Узгодження з попередніми моделями

- `User`, `Playlist`, `Track` і `PlaylistTrack` відповідають ER-моделі
  Завдання 1. Власність перевіряється через `Playlist.user_id`; трек
  шукається через `Track.track_id`; зв'язок і позиція зберігаються в
  `PlaylistTrack(playlist_id, track_id, position)`.
- `PlaylistAPI`, `AuthService`, `PlaylistService`, `CatalogService`,
  `PlaylistRepository` і `Database` зберігають ролі учасників Sequence
  Завдання 3. `TrackRepository` уточнює шар доступу до сутності `Track`.
- Компоненти реалізують FR-01–FR-07 та NFR-1/NFR-5 із Завдання 2.
  Цілісність дублікатів і транзакційність опираються на складений ключ
  `PlaylistTrack` і ADR-0001.
- Процес Завдання 4 відповідає оркестрації S2: `PlaylistAPI` перевіряє
  доступ і каталог до делегування `PlaylistService`, який працює з
  `PlaylistRepository`. У сценаріях S1/S5 `PlaylistService` сам викликає
  `CatalogService`, як показано в Sequence. Повідомлення слухача проходить
  через UI та `PlaylistAPI`.

Identity Provider додано як явне архітектурне припущення для вимоги
автентифікації. Попередні сценарії починаються з автентифікованої сесії та
абстрагують провайдера; інтеграцію не слід трактувати як нову бізнес-вимогу.
Перевірка каталогу є спільною залежністю: `PlaylistAPI` використовує її в S2,
а `PlaylistService` - у S1/S5, відповідно до Sequence та Activity-процесу.
Різні точки виклику є сценарно-залежною внутрішньою оркестрацією; зовнішній
контракт інтерфейсу `ICatalogLookup` спільний.
Платіжний, email/SMS та зовнішній музичний каталог не включені: вони не
беруть участі в UC-02, а `Track` є сутністю локальної ER-моделі.
