erDiagram
    %% Зв'язки
    Artist ||--o{ Album : "releases"
    Album ||--|{ Track : "contains"
    Artist }o--o{ Track : "collaborates on"
    User ||--o{ Playlist : "creates"
    Playlist }o--o{ Track : "includes"
    User }o--o{ Track : "likes"

    %% Сутності та атрибути
    User {
        int id PK
        string username
        string email
        string subscription_type
    }

    Artist {
        int id PK
        string name
        string bio
        boolean is_verified
    }

    Album {
        int id PK
        string title
        date release_date
        string album_type
    }

    Track {
        int id PK
        string title
        number duration_ms
        boolean is_explicit
    }

    Playlist {
        int id PK
        string title
        string description
        boolean is_public
    }