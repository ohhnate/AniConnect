# AniConnect Architecture

## Project Structure

```Markdown
AnimeConnect/
├── src/
│   ├── AniConnect.Web/           # Blazor WebAssembly Project
│   ├── AniConnect.API/           # .NET Web API
│   ├── AniConnect.Core/          # Business Logic
│   ├── AniConnect.Infrastructure/# Data Access & External Services
│   └── AniConnect.Shared/        # Shared DTOs & Constants
├── tests/
│   ├── AniConnect.UnitTests/
│   └── AniConnect.IntegrationTests/
├── docs/
└── tools/
```

## Development Workflow

1. Feature branches from `develop`
2. Pull requests with code review
3. Merge to `develop`
4. Release branches for production
5. Tag releases with semantic versioning

## GitHub Repository Setup

- Main Branch Protection
- PR Template
- Issue Templates
- GitHub Actions for CI/CD
- Dependabot Configuration

## Deployment Strategy

1. Development Environment
   - Local Development
   - GitHub Codespaces Support

2. Staging Environment (Render.com)
   - Automated Deployment from `develop`
   - Integration Testing

3. Production Environment (Render.com)
   - Manual Promotion from Staging
   - Database Backups
   - Monitor Resource Usage
