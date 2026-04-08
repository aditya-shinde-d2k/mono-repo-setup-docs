# Technical Implementation Documentation

**Enterprise-Grade Monorepo for Crismac NPA Axis**

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture Decisions](#architecture-decisions)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [Development Environment](#development-environment)
6. [Build & Deployment](#build--deployment)
7. [Code Organization](#code-organization)
8. [API Design](#api-design)
9. [Frontend Architecture](#frontend-architecture)
10. [Security Considerations](#security-considerations)
11. [Performance Optimization](#performance-optimization)
12. [Developer Experience](#developer-experience)
13. [Best Practices](#best-practices)
14. [Troubleshooting](#troubleshooting)

---

## Overview

### What is This?

This is an enterprise-grade monorepo implementation for the NPA Axis management system, combining Angular 18 frontend and .NET 9 backend in a single, cohesive repository.

### Why Monorepo?

**Benefits:**
- **Single Source of Truth**: All code in one place
- **Atomic Changes**: Frontend and backend changes in single commits
- **Consistent Tooling**: Shared configuration across projects
- **Simplified Dependencies**: Centralized package management
- **Better Code Reuse**: Shared libraries between projects
- **Easier Refactoring**: Cross-project changes are simpler
- **Single CI/CD Pipeline**: Unified deployment process

**Trade-offs:**
- Larger repository size
- Build time considerations (mitigated with incremental builds)
- Requires discipline in code organization

---

## Architecture Decisions

### ADR-001: Monorepo vs Polyrepo

**Decision**: Monorepo

**Context**: Need to manage frontend and backend with frequent cross-cutting changes.

**Reasoning**:
1. API contracts change frequently, requiring frontend updates
2. Shared types between frontend and backend reduce errors
3. Atomic commits ensure consistency
4. Single CI/CD pipeline reduces deployment complexity

**Consequences**:
- ✅ Easier coordination between teams
- ✅ Simplified version management
- ⚠️ Need clear module boundaries
- ⚠️ Require good build optimization

---

### ADR-002: pnpm Workspaces

**Decision**: Use pnpm with workspace support

**Context**: Need efficient package management for monorepo.

**Reasoning**:
1. **Disk Efficiency**: pnpm uses hard links, saving ~40% disk space
2. **Speed**: Faster than npm/yarn for installs
3. **Strictness**: Better dependency isolation
4. **Workspace Support**: Native monorepo support

**Comparison**:

| Feature | npm | yarn | pnpm |
|---------|-----|------|------|
| Speed | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Disk Usage | ⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| Security | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Workspaces | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

### ADR-003: Volta for Node Version Management

**Decision**: Use Volta instead of nvm

**Context**: Need consistent Node.js versions across team.

**Reasoning**:
1. **Automatic Version Switching**: Changes version per project automatically
2. **Fast**: Written in Rust, extremely fast
3. **Cross-Platform**: Works on Windows, Mac, Linux
4. **Tool Chaining**: Manages npm/yarn/pnpm versions too
5. **Zero Config**: Works automatically after initial setup

**vs nvm**:
- nvm requires manual switching (`nvm use`)
- Volta switches automatically when entering directory
- Volta is faster and more reliable on Windows

---

### ADR-004: Angular Standalone Components

**Decision**: Use Angular standalone API over NgModules

**Context**: Angular 18 supports standalone components.

**Reasoning**:
1. **Simpler Mental Model**: No NgModule boilerplate
2. **Better Tree-Shaking**: Smaller bundle sizes
3. **Easier Testing**: Components are self-contained
4. **Future-Proof**: Angular's direction going forward
5. **Faster Compilation**: Less metadata to process

**Example**:
```typescript
// Old way (NgModule)
@NgModule({
  declarations: [MyComponent],
  imports: [CommonModule],
  exports: [MyComponent]
})
export class MyModule {}

// New way (Standalone)
@Component({
  standalone: true,
  imports: [CommonModule],
  // ...
})
export class MyComponent {}
```

---

### ADR-004: Angular Standalone Components

**Decision**: Use Angular standalone API over NgModules

**Context**: Angular 18 supports standalone components.

**Reasoning**:
1. **Simpler Mental Model**: No NgModule boilerplate
2. **Better Tree-Shaking**: Smaller bundle sizes
3. **Easier Testing**: Components are self-contained
4. **Future-Proof**: Angular's direction going forward
5. **Faster Compilation**: Less metadata to process

**Example**:
```typescript
// Old way (NgModule)
@NgModule({
  declarations: [MyComponent],
  imports: [CommonModule],
  exports: [MyComponent]
})
export class MyModule {}

// New way (Standalone)
@Component({
  standalone: true,
  imports: [CommonModule],
  // ...
})
export class MyComponent {}
```

---

### ADR-005: Multi-Project Angular Workspace

**Decision**: Use Angular multi-project workspace with `--create-application=false`

**Context**: Need to manage main application and shared libraries with clear separation.

**Reasoning**:
1. **Code Reusability**: Shared library can be used across multiple apps
2. **Clear Boundaries**: Explicit public API through `public-api.ts`
3. **Independent Versioning**: Library can be versioned separately
4. **Better Organization**: Apps and libs clearly separated in `projects/` folder
5. **Development Flexibility**: Can point to source files or built dist for different workflows

**Structure Benefits**:
```
projects/
├── axis-masters-app/    # Main application
│   ├── public/         # Static assets (Angular 18+)
│   └── src/            # Source code
└── shared-lib/          # Shared code library
```

**Angular 18+ Changes**:
- Uses `public/` folder for static assets instead of `src/assets/`
- Files in `public/` are copied as-is to output (no processing)
- Use for: favicon.ico, robots.txt, static images
- For processed assets, still use `src/assets/`

**TypeScript Path Configuration**:
- **Development**: Points to `public-api.ts` for hot-reload
- **Production**: Points to `dist/` for optimized builds

**Implementation**:
```bash
# Create workspace without default app
ng new frontend --create-application=false

# Generate app in projects/
ng generate application axis-masters-app

# Generate library in projects/
ng generate library shared-lib
```

**Consequences**:
- ✅ Centralized shared code
- ✅ Easier to add more apps later
- ✅ Clear import paths via `@axis/shared-lib`
- ⚠️ Need to manage library builds
- ⚠️ Must maintain public API surface

---

### ADR-006: .NET 9 Minimal APIs Style

**Decision**: Use minimal API style with structured organization

**Context**: .NET 9 supports minimal APIs.

**Reasoning**:
1. **Less Boilerplate**: Simpler Program.cs
2. **Performance**: Slightly faster startup
3. **Modern**: Current .NET best practice
4. **Flexibility**: Easy to refactor to controllers if needed

**Implementation**: We use minimal API pattern but maintain controller structure for organization.

---

## Technology Stack

### Frontend Stack

```yaml
Core:
  - Angular: 18.x (latest stable)
  - TypeScript: 5.3+
  - RxJS: 7.8+

UI Framework:
  - PrimeNG: Latest
  - PrimeIcons: Latest

Build Tools:
  - Angular CLI: 18.x
  - esbuild: (via Angular CLI)

Development:
  - ESLint: Code linting
  - TypeScript: Type checking

Package Management:
  - pnpm: 8.15.0 (locked with Volta)
  - Node.js: 20.11.0 (locked with Volta)
```

### Backend Stack

```yaml
Core:
  - .NET: 9.0
  - ASP.NET Core: 9.0
  - C#: 12

ORM:
  - Entity Framework Core: 9.0

Database:
  - SQL Server: 2022 (or compatible)

Documentation:
  - Swagger/OpenAPI: via Swashbuckle

Authentication:
  - JWT Bearer Tokens
```

---

## Project Structure

### High-Level Structure

```
crismac-npa-axis/
├── apps/              # Applications
│   ├── frontend/     # Angular workspace
│   └── backend/      # .NET solution
├── libs/             # Shared libraries (future)
├── docs/             # Documentation
├── .vscode/          # VS Code configuration
└── package.json      # Workspace root
```

### Frontend Structure (Detailed)

```
apps/frontend/
├── projects/                      # Multi-project workspace
│   ├── axis-masters-app/         # Main application
│   │   ├── public/               # Static assets (Angular 18+)
│   │   │   └── favicon.ico
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── core/        # Core module (singleton services)
│   │   │   │   ├── shared/      # Shared within main app
│   │   │   │   ├── features/    # Feature modules
│   │   │   │   ├── app.component.ts
│   │   │   │   ├── app.config.ts     # App configuration
│   │   │   │   └── app.routes.ts     # Routing
│   │   │   ├── environments/    # Environment configs
│   │   │   │   ├── environment.ts
│   │   │   │   └── environment.prod.ts
│   │   │   ├── index.html
│   │   │   ├── main.ts
│   │   │   └── styles.scss      # Global styles
│   │   ├── tsconfig.app.json
│   │   └── tsconfig.spec.json
│   └── shared-lib/               # Shared library
│       ├── src/
│       │   ├── lib/
│       │   │   ├── components/  # Reusable UI components
│       │   │   ├── services/    # Shared services
│       │   │   ├── models/      # TypeScript interfaces
│       │   │   ├── guards/      # Route guards
│       │   │   ├── interceptors/ # HTTP interceptors
│       │   │   ├── directives/  # Custom directives
│       │   │   └── pipes/       # Custom pipes
│       │   └── public-api.ts    # Public API surface
│       ├── ng-package.json      # Library build config
│       ├── package.json         # Library dependencies
│       ├── README.md
│       ├── tsconfig.lib.json
│       ├── tsconfig.lib.prod.json
│       └── tsconfig.spec.json
├── .angular/                     # Angular cache
├── angular.json                  # Angular workspace config
├── package.json                  # Dependencies
└── tsconfig.json                 # Base TypeScript config
```

**Key Directories Explained**:

- **`projects/axis-masters-app/`**: Main application created with `ng generate application`
- **`projects/axis-masters-app/public/`**: Static assets (Angular 18+ uses `public/` instead of `assets/` at root)
- **`projects/shared-lib/`**: Buildable library for code shared across apps (created with `ng generate library`)
- **`projects/shared-lib/src/public-api.ts`**: Defines what's exported from the library
- **`projects/axis-masters-app/src/app/core/`**: Singleton services (AuthService, ApiService)
- **`projects/axis-masters-app/src/app/shared/`**: Reusable components within main app
- **`projects/axis-masters-app/src/app/features/`**: Feature modules (Auth, Dashboard, Reports)

### Backend Structure (Detailed)

```
apps/backend/
├── src/
│   └── Crismac.Npa.Axis.Api/
│       ├── Controllers/        # API endpoints
│       │   └── HealthController.cs
│       ├── Models/            # Domain models
│       ├── Services/          # Business logic
│       ├── Data/              # EF Core context
│       │   ├── ApplicationDbContext.cs
│       │   └── Configurations/
│       ├── DTOs/              # Data Transfer Objects
│       ├── Middleware/        # Custom middleware
│       ├── Extensions/        # Service extensions
│       ├── Properties/        # Launch settings
│       ├── Program.cs         # Entry point
│       ├── appsettings.json
│       └── *.csproj
├── tests/                     # Test projects (future)
├── Crismac.Npa.Axis.sln      # Solution file
└── README.md
```

**Layered Architecture**:

```
┌─────────────────────────┐
│   Controllers (API)     │ ← HTTP endpoints
├─────────────────────────┤
│   Services (Business)   │ ← Business logic
├─────────────────────────┤
│   Data (Persistence)    │ ← Database access
└─────────────────────────┘
```

---

## Development Environment

### Prerequisites Setup

#### 1. Install Volta

**macOS/Linux:**
```bash
curl https://get.volta.sh | bash
```

**Windows:**
```powershell
# Download and run installer from https://volta.sh
```

**Verify:**
```bash
volta --version
```

#### 2. Install .NET 9 SDK

Download from: https://dotnet.microsoft.com/download/dotnet/9.0

**Verify:**
```bash
dotnet --version  # Should show 9.0.x
```

#### 3. Clone Repository

```bash
git clone <repository-url>
cd crismac-npa-axis
```

Volta will automatically install correct Node.js (20.11.0) and pnpm (8.15.0).

### IDE Setup

#### VS Code (Recommended)

**Extensions Auto-Install:**
Open the project and VS Code will prompt to install recommended extensions:
- Angular Language Service
- C# Dev Kit
- ESLint

**Manual Install:**
```bash
code --install-extension angular.ng-template
code --install-extension ms-dotnettools.csdevkit
code --install-extension dbaeumer.vscode-eslint
```

#### JetBrains Rider (Alternative)

Rider has built-in support for both Angular and .NET.

---

### Environment Configuration

#### Frontend Environment Setup

**Development** (`apps/frontend/projects/axis-masters-app/src/environments/environment.ts`):
```typescript
export const environment = {
  production: false,
  apiUrl: 'https://localhost:5001/api',
  appName: 'Axis Masters',
  version: '1.0.0'
};
```

**Production** (`apps/frontend/projects/axis-masters-app/src/environments/environment.prod.ts`):
```typescript
export const environment = {
  production: true,
  apiUrl: '/api',  // Relative URL for production
  appName: 'Axis Masters',
  version: '1.0.0'
};
```

#### Backend Configuration

**Development** (`apps/backend/src/Crismac.Npa.Axis.Api/appsettings.Development.json`):
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=AxisNPA_Dev;Trusted_Connection=true;TrustServerCertificate=true;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Information"
    }
  }
}
```

**Production**: Use environment variables or Azure Key Vault for sensitive data.

---

## Build & Deployment

### Local Development Build

**Frontend:**

Two approaches based on `tsconfig.json` paths configuration:

**Approach 1: Development Mode (Hot-Reload)**
```bash
cd apps/frontend

# Configure tsconfig.json to point to source:
# "paths": { "@axis/shared-lib": ["projects/shared-lib/src/public-api.ts"] }

pnpm run start
# No library build needed - changes reflect immediately
```

**Approach 2: Production-Like Mode**
```bash
cd apps/frontend

# Configure tsconfig.json to point to dist:
# "paths": { "@axis/shared-lib": ["dist/shared-lib"] }

# Build library first
pnpm run build:lib

# Then start application
pnpm run start
# Output: dist/axis-masters-app/
```

**Backend:**
```bash
cd apps/backend
dotnet build
# Output: bin/Debug/net9.0/
```

### Production Build

**Frontend:**
```bash
cd apps/frontend

# Ensure tsconfig.json paths point to dist:
# "paths": { "@axis/shared-lib": ["dist/shared-lib"] }

# Build shared library first
pnpm run build:lib

# Build application
ng build axis-masters-app --configuration production

# Outputs optimized bundle with:
# - Minification
# - Tree-shaking
# - AOT compilation
# - Source maps (optional)

# Output location: dist/axis-masters-app/
```

**Backend:**
```bash
cd apps/backend
dotnet publish -c Release -o ./publish
# Outputs self-contained deployment
```

### Build Optimization

#### Frontend Bundle Analysis

```bash
cd apps/frontend
pnpm run build -- --stats-json
# Analyze with webpack-bundle-analyzer
```

**Target Bundle Sizes:**
- Initial Bundle: < 500KB
- Lazy-loaded chunks: < 100KB each

#### Backend Build Optimization

```xml
<!-- In .csproj -->
<PropertyGroup>
  <PublishTrimmed>true</PublishTrimmed>
  <PublishSingleFile>true</PublishSingleFile>
</PropertyGroup>
```

---

## Code Organization

### Frontend Code Organization

#### Feature-Based Structure

```
projects/axis-masters-app/src/app/features/
├── auth/
│   ├── components/
│   │   ├── login/
│   │   └── register/
│   ├── services/
│   │   └── auth.service.ts
│   └── auth.routes.ts
├── dashboard/
│   ├── components/
│   │   └── dashboard/
│   ├── services/
│   └── dashboard.routes.ts
└── reports/
    ├── components/
    ├── services/
    └── reports.routes.ts
```

#### Shared Library Organization

```
projects/shared-lib/src/lib/
├── components/          # UI components
│   ├── button/
│   ├── table/
│   └── index.ts        # Barrel export
├── services/           # Shared services
│   ├── api.service.ts
│   ├── storage.service.ts
│   └── index.ts
├── models/             # TypeScript interfaces
│   ├── user.model.ts
│   ├── api-response.model.ts
│   └── index.ts
├── guards/             # Route guards
│   ├── auth.guard.ts
│   └── index.ts
├── interceptors/       # HTTP interceptors
│   ├── auth.interceptor.ts
│   ├── error.interceptor.ts
│   └── index.ts
├── directives/         # Custom directives
│   ├── tooltip.directive.ts
│   └── index.ts
└── pipes/              # Custom pipes
    ├── safe-html.pipe.ts
    └── index.ts
```

**TypeScript Path Configuration for Development:**

In `apps/frontend/tsconfig.json`:
```json
{
  "compilerOptions": {
    "paths": {
      "@axis/shared-lib": [
        "projects/shared-lib/src/public-api.ts"
      ],
      "@axis/shared-lib/*": [
        "projects/shared-lib/src/*"
      ]
    }
  }
}
```

**For Production**, change paths to use built library:
```json
{
  "compilerOptions": {
    "paths": {
      "@axis/shared-lib": ["dist/shared-lib"],
      "@axis/shared-lib/*": ["dist/shared-lib/*"]
    }
  }
}
```

This allows hot-reload during development without rebuilding the library.

### Backend Code Organization

#### Controller Organization

```csharp
// One controller per resource
namespace Crismac.Npa.Axis.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // GET api/users
    // GET api/users/{id}
    // POST api/users
    // PUT api/users/{id}
    // DELETE api/users/{id}
}
```

#### Service Layer Pattern

```csharp
// Interface
public interface IUserService
{
    Task<User> GetByIdAsync(int id);
    Task<IEnumerable<User>> GetAllAsync();
}

// Implementation
public class UserService : IUserService
{
    private readonly ApplicationDbContext _context;
    
    public UserService(ApplicationDbContext context)
    {
        _context = context;
    }
    
    // Implementation...
}

// Registration in Program.cs
builder.Services.AddScoped<IUserService, UserService>();
```

---

## API Design

### RESTful Conventions

| Method | URI | Action |
|--------|-----|--------|
| GET | `/api/users` | List all users |
| GET | `/api/users/{id}` | Get user by ID |
| POST | `/api/users` | Create new user |
| PUT | `/api/users/{id}` | Update user |
| DELETE | `/api/users/{id}` | Delete user |

### Request/Response Format

**Success Response:**
```json
{
  "data": {
    "id": 1,
    "name": "John Doe"
  },
  "message": "Success",
  "timestamp": "2024-12-05T10:30:00Z"
}
```

**Error Response:**
```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 not found",
    "details": []
  },
  "timestamp": "2024-12-05T10:30:00Z"
}
```

### HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation errors |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 500 | Server Error | Unhandled exceptions |

---

## Frontend Architecture

### Component Design Principles

#### 1. Single Responsibility

```typescript
// ❌ Bad: Component doing too much
@Component({
  // Component handles API calls, business logic, and UI
})

// ✅ Good: Separated concerns
@Component({
  // Component handles only UI logic
  // Delegates to service for data
})
```

#### 2. Smart vs Presentational Components

**Smart Component (Container):**
```typescript
// Handles business logic, API calls
@Component({
  selector: 'app-user-list-container',
  template: '<app-user-list [users]="users$ | async"></app-user-list>'
})
export class UserListContainerComponent {
  users$ = this.userService.getUsers();
  
  constructor(private userService: UserService) {}
}
```

**Presentational Component:**
```typescript
// Pure UI, no business logic
@Component({
  selector: 'app-user-list',
  template: '...',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent {
  @Input() users: User[] = [];
  @Output() userSelected = new EventEmitter<User>();
}
```

### State Management

**For Simple Apps:** RxJS BehaviorSubjects
```typescript
@Injectable({ providedIn: 'root' })
export class StateService {
  private state$ = new BehaviorSubject<AppState>(initialState);
  
  getState() {
    return this.state$.asObservable();
  }
  
  updateState(newState: Partial<AppState>) {
    this.state$.next({ ...this.state$.value, ...newState });
  }
}
```

**For Complex Apps:** Consider NgRx (not included in scaffold)

---

### Using Shared Library Code

Import from shared library using the configured TypeScript path alias:

```typescript
// projects/axis-masters-app/src/app/some-feature/component.ts
import { Component } from '@angular/core';
// Import from shared library using @axis/shared-lib
import { ButtonComponent } from '@axis/shared-lib';
import { ApiService } from '@axis/shared-lib';
import { User } from '@axis/shared-lib';

@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [ButtonComponent],
  template: `
    <axis-button label="Click me" />
  `
})
export class FeatureComponent {
  constructor(private apiService: ApiService) {}
}
```

**Benefits of Shared Library**:
- Single source of truth for common code
- Type-safe imports
- Easy refactoring across projects
- Can be versioned independently
- Clear public API surface

### Routing Strategy

```typescript
// projects/axis-masters-app/src/app/app.routes.ts

// Lazy loading for performance
export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/dashboard.component')
      .then(m => m.DashboardComponent)
  },
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.routes')
      .then(m => m.USERS_ROUTES)
  }
];
```

---

## Security Considerations

### Authentication Flow

```
1. User submits credentials
2. Backend validates and returns JWT
3. Frontend stores JWT (HttpOnly cookie preferred)
4. Frontend sends JWT in Authorization header
5. Backend validates JWT on each request
```

### CORS Configuration

**Backend (`Program.cs`):**
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();  // For cookies
    });
});
```

**Production:** Replace with actual domain.

### Input Validation

**Backend:**
```csharp
// Use Data Annotations
public class CreateUserDto
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
}
```

**Frontend:**
```typescript
// Use Angular Reactive Forms
this.form = this.fb.group({
  name: ['', [Validators.required, Validators.minLength(2)]],
  email: ['', [Validators.required, Validators.email]]
});
```

### SQL Injection Prevention

**Always use parameterized queries:**
```csharp
// ❌ Bad: String concatenation
var sql = $"SELECT * FROM Users WHERE Id = {id}";

// ✅ Good: Entity Framework (parameterized automatically)
var user = await _context.Users.FindAsync(id);

// ✅ Good: Dapper with parameters
var user = await connection.QueryFirstAsync<User>(
    "SELECT * FROM Users WHERE Id = @Id",
    new { Id = id }
);
```

---

## Performance Optimization

### Frontend Performance

#### 1. OnPush Change Detection

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class MyComponent {
  // Only checks when:
  // - @Input() changes
  // - Events fire
  // - Async pipe emits
}
```

#### 2. Virtual Scrolling

```typescript
// For large lists
<cdk-virtual-scroll-viewport itemSize="50">
  <div *cdkVirtualFor="let item of items">
    {{ item.name }}
  </div>
</cdk-virtual-scroll-viewport>
```

#### 3. Lazy Loading Images

```html
<img [src]="imageUrl" loading="lazy" />
```

### Backend Performance

#### 1. Async/Await Everything

```csharp
// ❌ Bad: Blocking
public User GetUser(int id)
{
    return _context.Users.Find(id);
}

// ✅ Good: Async
public async Task<User> GetUserAsync(int id)
{
    return await _context.Users.FindAsync(id);
}
```

#### 2. Use IQueryable for Filtering

```csharp
// ✅ Good: Query executed in database
public async Task<List<User>> GetActiveUsersAsync()
{
    return await _context.Users
        .Where(u => u.IsActive)  // Filters in database
        .ToListAsync();
}
```

#### 3. Response Caching

```csharp
[HttpGet]
[ResponseCache(Duration = 60)]  // Cache for 60 seconds
public async Task<IActionResult> GetUsers()
{
    var users = await _userService.GetAllAsync();
    return Ok(users);
}
```

---

## Developer Experience

### Fast Feedback Loops

**Frontend Hot Reload:**
- Changes reflect instantly in browser
- No page refresh needed
- State preserved

**Backend Hot Reload:**
```bash
dotnet watch run
# Automatically rebuilds and restarts on file changes
```

### Debugging

#### Frontend Debugging (Chrome)

1. Start dev server: `pnpm run frontend:dev`
2. Open Chrome DevTools (F12)
3. Use Angular DevTools extension
4. Set breakpoints in TypeScript files

#### Backend Debugging (VS Code)

1. Press F5 or use Debug panel
2. Set breakpoints in .cs files
3. Inspect variables, call stack
4. Step through code

### Code Quality Tools

**Frontend:**
- **ESLint**: Linting TypeScript
- **Prettier**: Code formatting (if added)
- **TypeScript**: Type checking

**Backend:**
- **Roslyn Analyzers**: C# code analysis
- **EditorConfig**: Consistent formatting
- **.editorconfig**: Cross-editor settings

---

## Best Practices

### Git Commit Messages

Use Conventional Commits:

```
feat: add user authentication
fix: resolve CORS issue in production
docs: update API documentation
chore: upgrade Angular to v18
```

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code refactoring
- `test`: Tests
- `chore`: Maintenance

### Code Review Checklist

- [ ] Code follows style guidelines
- [ ] No console.log statements
- [ ] All variables/functions have meaningful names
- [ ] Comments explain "why", not "what"
- [ ] No hardcoded values
- [ ] Error handling implemented
- [ ] Tests added (if applicable)
- [ ] Documentation updated

### Testing Strategy

**Frontend Testing:**
```typescript
// Unit tests for services
describe('UserService', () => {
  it('should fetch users', async () => {
    // Test implementation
  });
});

// Component tests
describe('UserListComponent', () => {
  it('should display users', () => {
    // Test implementation
  });
});
```

**Backend Testing:**
```csharp
// Unit tests for services
[Fact]
public async Task GetUser_ReturnsUser_WhenUserExists()
{
    // Arrange
    // Act
    // Assert
}

// Integration tests for API
[Fact]
public async Task GetUsers_ReturnsOkResult()
{
    // Test implementation
}
```

---

## Troubleshooting

### Common Issues

#### Issue: "Command not found: pnpm"

**Solution:**
```bash
# Install Volta first
curl https://get.volta.sh | bash

# Volta will install pnpm automatically when you enter project directory
cd crismac-npa-axis
```

#### Issue: Port 4200 already in use

**Solution:**
```bash
# macOS/Linux
lsof -ti:4200 | xargs kill -9

# Windows
netstat -ano | findstr :4200
taskkill /PID <PID> /F
```

#### Issue: Port 5001 already in use

**Solution:**
```bash
# macOS/Linux
lsof -ti:5001 | xargs kill -9

# Windows
netstat -ano | findstr :5001
taskkill /PID <PID> /F
```

#### Issue: CORS errors in browser

**Symptoms:** Console shows CORS policy errors

**Solution:**
1. Verify backend CORS configuration in `Program.cs`
2. Ensure frontend URL matches CORS allowed origins
3. Check if backend is running on correct port (5001)

#### Issue: Angular build fails with memory error

**Solution:**
```bash
# Increase Node memory limit
export NODE_OPTIONS="--max-old-space-size=4096"
pnpm run build
```

#### Issue: .NET restore fails

**Solution:**
```bash
# Clear NuGet cache
dotnet nuget locals all --clear

# Restore again
dotnet restore
```

---

## Monitoring & Logging

### Frontend Logging

```typescript
// Production-safe logging service
@Injectable({ providedIn: 'root' })
export class LogService {
  private isProduction = environment.production;
  
  log(message: string, data?: any) {
    if (!this.isProduction) {
      console.log(message, data);
    }
  }
  
  error(message: string, error?: any) {
    console.error(message, error);
    // Send to monitoring service (e.g., Sentry)
  }
}
```

### Backend Logging

```csharp
// Use ILogger
public class UserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        _logger.LogInformation("Fetching user {UserId}", id);
        
        try
        {
            return await _context.Users.FindAsync(id);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error fetching user {UserId}", id);
            throw;
        }
    }
}
```

---

## Deployment Considerations 

### Environment Variables

**Frontend (Angular):**
- Build-time variables via `environment.ts`
- Runtime variables via `window.__env` (if needed)

**Backend (.NET):**
- Environment variables
- User secrets (development)
- Azure Key Vault (production)

### Health Checks

**Implement health endpoints:**

```csharp
// Already implemented in HealthController.cs
GET /api/health
{
  "status": "Healthy",
  "timestamp": "2024-12-05T10:30:00Z",
  "version": "1.0.0"
}
```

**Use for:**
- Load balancer health checks
- Monitoring systems
- CI/CD pipeline verification

---

## Performance Metrics

### Frontend Targets

- **First Contentful Paint (FCP)**: < 1.8s
- **Largest Contentful Paint (LCP)**: < 2.5s
- **Time to Interactive (TTI)**: < 3.8s
- **Cumulative Layout Shift (CLS)**: < 0.1
- **Bundle Size**: < 500KB (initial)

### Backend Targets

- **Response Time (p95)**: < 200ms
- **Response Time (p99)**: < 500ms
- **Throughput**: > 1000 req/s
- **Error Rate**: < 0.1%

### Measuring Performance

**Frontend:**
```bash
# Lighthouse audit
npm install -g lighthouse
lighthouse http://localhost:4200 --view
```

**Backend:**
```bash
# Using Apache Bench
ab -n 1000 -c 10 https://localhost:5001/api/health
```

---

## Maintenance

### Dependency Updates

**Frontend:**
```bash
cd apps/frontend
pnpm update --latest
pnpm audit
```

**Backend:**
```bash
cd apps/backend
dotnet list package --outdated
dotnet add package <package-name> --version <version>
```

### Security Audits

**Frontend:**
```bash
pnpm audit
pnpm audit --fix
```

**Backend:**
```bash
dotnet list package --vulnerable
```

---

## Conclusion

This technical documentation provides the foundation for building and maintaining an enterprise-grade monorepo. Key takeaways:

1. **Consistency**: Use Volta for version locking, pnpm for package management
2. **Organization**: Clear separation between frontend/backend with shared libraries
3. **Developer Experience**: Fast feedback loops, good debugging, automated checks
4. **Performance**: OnPush change detection, async/await, caching strategies
5. **Security**: JWT auth, CORS, input validation, parameterized queries
6. **Best Practices**: Conventional commits, code reviews, testing

For setup instructions, refer to **SETUP_GUIDE.md**.

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained By**: Crismac Development Team
