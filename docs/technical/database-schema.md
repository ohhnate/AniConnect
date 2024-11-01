# Database Schema

## Users Table

```sql
CREATE TABLE Users (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    Username VARCHAR(50) UNIQUE NOT NULL,
    Email VARCHAR(255) UNIQUE NOT NULL,
    PasswordHash VARCHAR(255) NOT NULL,
    ProfilePicture VARCHAR(255),
    JoinDate TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    LastActive TIMESTAMP,
    Bio TEXT,
    PreferredLanguage VARCHAR(10) DEFAULT 'en',
    IsPrivate BOOLEAN DEFAULT FALSE,
    IsBanned BOOLEAN DEFAULT FALSE,
    ThemePreference VARCHAR(20) DEFAULT 'light',
    NotificationPreferences JSONB,
    CONSTRAINT proper_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

CREATE INDEX idx_users_username ON Users(username);
CREATE INDEX idx_users_email ON Users(email);
```

## MediaItems Table

```sql
CREATE TABLE MediaItems (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    AniListId INTEGER UNIQUE NOT NULL,
    Type VARCHAR(20) NOT NULL CHECK (Type IN ('Anime', 'Manga', 'LightNovel')),
    Title VARCHAR(255) NOT NULL,
    EnglishTitle VARCHAR(255),
    JapaneseTitle VARCHAR(255),
    Description TEXT,
    CoverImage VARCHAR(255),
    BannerImage VARCHAR(255),
    ReleaseDate DATE,
    Status VARCHAR(20) NOT NULL,
    Episodes INTEGER,
    Chapters INTEGER,
    Volumes INTEGER,
    AverageScore DECIMAL(4,2),
    Popularity INTEGER,
    Genres TEXT[],
    Studios TEXT[],
    Source VARCHAR(50),
    LastUpdated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT valid_score CHECK (AverageScore >= 0 AND AverageScore <= 100)
);

CREATE INDEX idx_mediaitems_anilistid ON MediaItems(AniListId);
CREATE INDEX idx_mediaitems_type ON MediaItems(Type);
```

## UserMediaStatus Table

```sql
CREATE TABLE UserMediaStatus (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    UserId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    MediaItemId UUID REFERENCES MediaItems(Id) ON DELETE CASCADE,
    Status VARCHAR(20) NOT NULL CHECK (Status IN ('Watching', 'Reading', 'Completed', 'Planning', 'Dropped', 'Paused')),
    Progress INTEGER DEFAULT 0,
    Rating DECIMAL(3,1) CHECK (Rating >= 0 AND Rating <= 10),
    Review TEXT,
    ReviewVisibility VARCHAR(20) DEFAULT 'public' CHECK (ReviewVisibility IN ('public', 'friends', 'private')),
    StartDate DATE,
    CompleteDate DATE,
    DateUpdated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_media UNIQUE (UserId, MediaItemId)
);

CREATE INDEX idx_usermedia_user ON UserMediaStatus(UserId);
CREATE INDEX idx_usermedia_media ON UserMediaStatus(MediaItemId);
```

## StatusUpdates Table

```sql
CREATE TABLE StatusUpdates (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    UserId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    Content TEXT NOT NULL,
    MediaItemId UUID REFERENCES MediaItems(Id) ON DELETE SET NULL,
    PostDate TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    IsPrivate BOOLEAN DEFAULT FALSE,
    Likes INTEGER DEFAULT 0,
    Comments INTEGER DEFAULT 0,
    Visibility VARCHAR(20) DEFAULT 'public' CHECK (Visibility IN ('public', 'friends', 'private')),
    AttachmentUrl VARCHAR(255)
);

CREATE INDEX idx_statusupdates_user ON StatusUpdates(UserId);
CREATE INDEX idx_statusupdates_date ON StatusUpdates(PostDate DESC);
```

## Comments Table

```sql
CREATE TABLE Comments (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    UserId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    ProfileUserId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    Content TEXT NOT NULL,
    PostDate TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    Likes INTEGER DEFAULT 0,
    IsEdited BOOLEAN DEFAULT FALSE,
    EditedDate TIMESTAMP,
    CONSTRAINT content_not_empty CHECK (LENGTH(TRIM(Content)) > 0)
);

CREATE INDEX idx_comments_user ON Comments(UserId);
CREATE INDEX idx_comments_profile ON Comments(ProfileUserId);
CREATE INDEX idx_comments_date ON Comments(PostDate DESC);
```

## PrivateMessages Table

```sql
CREATE TABLE PrivateMessages (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    SenderId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    ReceiverId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    Content TEXT NOT NULL,
    SendDate TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ReadDate TIMESTAMP,
    IsDeleted BOOLEAN DEFAULT FALSE,
    CONSTRAINT content_not_empty CHECK (LENGTH(TRIM(Content)) > 0)
);

CREATE INDEX idx_messages_sender ON PrivateMessages(SenderId);
CREATE INDEX idx_messages_receiver ON PrivateMessages(ReceiverId);
CREATE INDEX idx_messages_date ON PrivateMessages(SendDate DESC);
```

## UserConnections Table

```sql
CREATE TABLE UserConnections (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    RequesterId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    AddresseeId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    Status VARCHAR(20) NOT NULL DEFAULT 'pending' 
        CHECK (Status IN ('pending', 'accepted', 'rejected', 'blocked')),
    ConnectionDate TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_connection UNIQUE (RequesterId, AddresseeId),
    CONSTRAINT no_self_connection CHECK (RequesterId != AddresseeId)
);

CREATE INDEX idx_connections_requester ON UserConnections(RequesterId);
CREATE INDEX idx_connections_addressee ON UserConnections(AddresseeId);
```

## MediaLikes Table

```sql
CREATE TABLE MediaLikes (
    Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    UserId UUID REFERENCES Users(Id) ON DELETE CASCADE,
    MediaItemId UUID REFERENCES MediaItems(Id) ON DELETE CASCADE,
    LikeDate TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_media_like UNIQUE (UserId, MediaItemId)
);

CREATE INDEX idx_medialikes_user ON MediaLikes(UserId);
CREATE INDEX idx_medialikes_media ON MediaLikes(MediaItemId);
```

## Entity Relationships

### One-to-Many Relationships

- User → StatusUpdates (One user can have many status updates)
- User → UserMediaStatus (One user can have many media statuses)
- MediaItem → UserMediaStatus (One media item can have many user statuses)
- User → Comments (One user can make many comments)
- User → PrivateMessages (One user can send many messages)

### Many-to-Many Relationships

- Users ↔ Users (Through UserConnections table)
- Users ↔ MediaItems (Through UserMediaStatus table)
- Users ↔ MediaItems (Through MediaLikes table)

### Cascade Deletions

- When a User is deleted:
  - All their StatusUpdates are deleted
  - All their UserMediaStatus entries are deleted
  - All their Comments are deleted
  - All their PrivateMessages are deleted
  - All their UserConnections are deleted

- When a MediaItem is deleted:
  - All associated UserMediaStatus entries are deleted
  - Associated StatusUpdates have their MediaItemId set to NULL
  - All associated MediaLikes are deleted

## Indexes and Constraints

### Primary Keys

- All tables use UUID as primary keys with auto-generation

### Foreign Keys

- All foreign keys use CASCADE deletion except for MediaItemId in StatusUpdates (SET NULL)
- All references maintain referential integrity

### Unique Constraints

- Users: Username, Email
- UserMediaStatus: (UserId, MediaItemId)
- UserConnections: (RequesterId, AddresseeId)
- MediaLikes: (UserId, MediaItemId)

### Check Constraints

- Email format validation
- Status enums for UserMediaStatus
- Visibility options for content
- Rating ranges (0-10 for user ratings, 0-100 for average scores)
- Content non-empty checks for messages and comments
- No self-connections in UserConnections

### Indexes

- Username and Email indexes for User lookups
- AniListId index for external API synchronization
- Type index for media filtering
- Date-based indexes for chronological queries
- User-based indexes for relationship queries
