# AniConnect API Integration

## Data Sync Service

```csharp
public interface IMediaSyncService
{
    Task SyncMediaUpdatesAsync();
    Task SyncNewMediaAsync();
    Task UpdateMediaDetailsAsync(int aniListId);
}

public class MediaSyncService : IMediaSyncService
{
    private readonly IMediaRepository _mediaRepository;
    private readonly IAniListClient _aniListClient;
    private readonly ILogger<MediaSyncService> _logger;
    
    // Implementation details...
}
```

## AniList GraphQL Queries

```graphql
# Base Media Query
query ($id: Int) {
  Media (id: $id) {
    id
    title {
      romaji
      english
      native
    }
    type
    format
    status
    description
    startDate {
      year
      month
      day
    }
    episodes
    chapters
    volumes
    coverImage {
      large
    }
    bannerImage
    genres
    averageScore
    popularity
    studios {
      nodes {
        name
      }
    }
  }
}
```

## Sync Schedule

1. Full Sync (Daily at 3 AM UTC):
   - Fetch all updated media from last 24 hours
   - Update existing records
   - Add new media items

2. Partial Sync (Every 6 hours):
   - Update popular and trending media
   - Sync user-requested media

3. On-Demand Sync:
   - When users search for media not in database
   - When accessing media details not yet cached

## Rate Limiting

- Implement token bucket algorithm
- Respect AniList's rate limits:
  - 90 requests per minute
  - Exponential backoff on errors

## Error Handling

```csharp
public class AniListSyncException : Exception
{
    public string ResourceId { get; }
    public SyncOperation Operation { get; }
    public DateTime Timestamp { get; }
    
    // Implementation details...
}

public enum SyncOperation
{
    FullSync,
    PartialSync,
    SingleItemSync,
    UserRequested
}
```
