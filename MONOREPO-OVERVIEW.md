# CrisMAC AXIS NPA — Monorepo Comprehensive Overview

> **Purpose:** Full architectural map of the monorepo for planning monorepo expansion.
> **Generated:** 2026-04-04
> **Based on:** Direct codebase analysis — no assumptions.

---

## Table of Contents

1. [Root-Level Structure](#1-root-level-structure)
2. [Monorepo Tooling](#2-monorepo-tooling)
3. [Frontend Workspace — Angular](#3-frontend-workspace--angular)
   - [projects/npa-app](#31-projectsnpa-app)
   - [projects/shared-lib](#32-projectsshared-lib)
4. [Backend Structure — ASP.NET Core](#4-backend-structure--aspnet-core)
5. [AREAS/ Folder](#5-areas-folder)
6. [doc/ Folder](#6-doc-folder)
7. [deploy/ Folder](#7-deploy-folder)
8. [Key Config Files](#8-key-config-files)
9. [Technology Stack Summary](#9-technology-stack-summary)
10. [Architecture Overview](#10-architecture-overview)
11. [Data Flows](#11-data-flows)
12. [Security Layers](#12-security-layers)
13. [Performance Configuration](#13-performance-configuration)
14. [Expansion Planning Notes](#14-expansion-planning-notes)

---

## 1. Root-Level Structure

```
crismac-npa-axis/                         # Git root
├── frontend/                             # Angular 18 workspace
├── backend/                              # ASP.NET Core 9 Web API
├── doc/                                  # Project documentation
├── deploy/                               # Deployment configs & scripts
├── .claude/                              # Claude Code worktrees/settings
├── .husky/                               # Git hooks (commit-msg, pre-commit)
├── .vscode/                              # VS Code workspace settings
├── .editorconfig                         # Universal editor config
├── .gitattributes
├── .gitignore
├── .lintstagedrc.js                      # Lint-staged configuration
├── .npmrc                                # pnpm settings
├── .prettierrc / .prettierignore
├── crismac-npa-axis.code-workspace       # VS Code multi-root workspace
├── global.json                           # .NET SDK version pin → 9.0.111
├── package.json                          # Root-level meta (husky, commitlint)
└── pnpm-lock.yaml                        # Lockfile (54 KB)
```

### Root package.json

```json
{
  "name": "crismac-axis-monorepo",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "preinstall": "npx only-allow pnpm",
    "prepare": "husky",
    "lint-staged": "lint-staged"
  },
  "devDependencies": {
    "husky": "^9.0.11",
    "lint-staged": "^15.2.0",
    "@commitlint/cli": "^18.4.3",
    "@commitlint/config-conventional": "^18.4.3"
  },
  "volta": {
    "node": "20.19.5",
    "pnpm": "9.15.0"
  }
}
```

**Notes:**
- Root `package.json` is meta-only — no `workspaces` field. Frontend and backend are independent sub-projects.
- `preinstall` enforces pnpm as the only allowed package manager.
- Volta pins **Node 20.19.5** and **pnpm 9.15.0** for all contributors.
- `global.json` pins **.NET SDK 9.0.111** for the backend.

---

## 2. Monorepo Tooling

| Tool | Usage | Config File |
|------|-------|-------------|
| **pnpm** | JS package manager (enforced) | `.npmrc`, `pnpm-lock.yaml` |
| **Volta** | Node/pnpm version pinning | `package.json` → `"volta"` |
| **Husky** | Git hooks | `.husky/` |
| **Commitlint** | Conventional commit enforcement | `@commitlint/config-conventional` |
| **Lint-staged** | Pre-commit lint on changed files | `.lintstagedrc.js` |
| **Angular CLI** | Frontend build/serve/test | `frontend/angular.json` |
| **ng-packagr** | Angular library build | `frontend/projects/shared-lib/ng-package.json` |
| **.NET SDK 9** | Backend build/run | `global.json` |

**What is NOT present:**
- No Nx (`nx.json` absent)
- No Lerna (`lerna.json` absent)
- No Turborepo (`turbo.json` absent)
- No `pnpm-workspace.yaml` — not a pnpm workspace
- No Rush, Bazel, or other build orchestrators

**Conclusion:** This is a *simple co-located monorepo* — Angular workspace (pnpm) + .NET solution, sharing a git repo with root-level tooling for git hooks and commit standards only.

---

## 3. Frontend Workspace — Angular

**Location:** `frontend/`

```
frontend/
├── projects/
│   ├── npa-app/                          # Angular application
│   └── shared-lib/                       # Angular library
├── shared-public/                        # Shared static assets
├── angular.json                          # Angular workspace config
├── package.json                          # Frontend dependencies
├── tsconfig.json                         # Root TypeScript config
├── tsconfig.app.json
├── tsconfig.spec.json
├── .eslintrc.json
└── karma.conf.js
```

### Frontend package.json — Dependencies

**Runtime (39 packages):**

| Package | Version | Purpose |
|---------|---------|---------|
| `@angular/*` | 18.2.14 | Framework core, platform, router, forms, material, ssr |
| `primeng` | 18.0.2 | UI component library |
| `primeflex` | 3.3.1 | CSS utility classes |
| `primeicons` | 7.0.0 | Icon set |
| `rxjs` | ~7.8.0 | Reactive programming |
| `chart.js` | latest | Charts for dashboard |
| `quill` | latest | Rich text editor |
| `crypto-js` | latest | AES-256-CBC encryption |
| `file-saver` | latest | File download |
| `uuid` | latest | Unique IDs |
| `ngx-cookie-service` | latest | Cookie management |
| `ngx-otp-input` | latest | OTP input component |
| `ngx-pagination` | latest | Pagination |
| `ng-angular-popup` | latest | Popup dialogs |
| `angular-captcha` | latest | CAPTCHA |
| `rest-api-handler` / `apihandler` | latest | HTTP abstraction |

**DevDependencies (35 packages):**

| Package | Version | Purpose |
|---------|---------|---------|
| `@angular/cli` | 18.2.14 | CLI build/serve/generate |
| `@angular-devkit/build-angular` | latest | Build pipeline |
| `@angular-eslint/*` | 18.4.2 | Angular ESLint rules |
| `eslint`, `prettier` | latest | Code quality |
| `ng-packagr` | latest | Library packaging |
| `karma`, `jasmine` | latest | Unit testing |
| `concurrently` | latest | Parallel watch scripts |
| `wait-on` | latest | Wait for dist before serving |

### Frontend Scripts

```bash
pnpm run start:dev        # Concurrent: watch shared-lib + serve npa-app with HMR
pnpm run watch-lib        # ng build shared-lib --watch
pnpm run serve-app:dev    # wait-on dist/shared-lib → ng serve --hmr
pnpm run build            # build:shared-lib THEN build:npa-app (sequential)
pnpm run build:shared-lib # ng build shared-lib --configuration production
pnpm run build:npa-app    # ng build npa-app --configuration production
pnpm run test             # ng test
pnpm run lint             # ng lint
pnpm run format           # prettier --write
pnpm run format:check     # prettier --check
```

### tsconfig.json (Frontend Root)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "strict": true,
    "paths": {
      "shared-lib": ["./dist/shared-lib"]
    }
  }
}
```

**Important:** `shared-lib` is consumed from `dist/shared-lib` — the built output, not source. This is the standard Angular library pattern requiring a prior `build:shared-lib` step.

---

### 3.1 projects/npa-app

**Type:** Angular Application
**Prefix:** `app`
**Build output:** `dist/npa-app`

#### Directory Layout

```
projects/npa-app/src/
├── app/
│   ├── core/
│   │   ├── interfaces/
│   │   │   ├── icomplaint.ts
│   │   │   ├── ilog.ts
│   │   │   └── iuser.ts
│   │   ├── menu/
│   │   │   ├── menu-api.service.ts
│   │   │   ├── menu.model.ts
│   │   │   ├── menu.service.ts
│   │   │   └── menu-list-response-model.ts
│   │   └── services/
│   │       ├── auth.service.ts
│   │       ├── complaint-details.service.ts
│   │       ├── core.service.ts
│   │       └── (10+ service files)
│   ├── layout/
│   │   ├── app.footer.component.ts
│   │   ├── app.main.component.ts / .html
│   │   ├── app.menu.component.ts
│   │   ├── app.menu.service.ts
│   │   ├── app.menuitem.component.ts
│   │   ├── app.rightpanel.component.ts / .html
│   │   └── app.topbar.component.ts
│   ├── pages/
│   │   ├── auth/
│   │   │   ├── login/                    # login.component.ts (14 KB)
│   │   │   ├── reset-password/           # reset-password.component.ts (17 KB)
│   │   │   └── change-password/          # change-password.component.ts (16 KB)
│   │   ├── main-dashboard/
│   │   │   └── dashboard/               # dashboard.component.ts (stub)
│   │   ├── master-uploads/              # master-uploads.component.ts (27 KB — MAIN FEATURE)
│   │   └── report-access/
│   ├── utilities/
│   │   ├── base-href.service.ts
│   │   ├── common-module.ts
│   │   └── constant.ts
│   ├── environments/
│   │   ├── environment.ts               # base
│   │   ├── environment.dev.ts           # development
│   │   └── environment.prod.ts          # production
│   ├── app.component.ts / .html
│   ├── app.config.ts                    # 4.9 KB — bootstrap providers
│   └── app.routes.ts                    # route definitions
└── styles.scss
```

#### app.config.ts — Key Providers

```typescript
provideRouter(AppRoutes)              // custom RouteReuseStrategy (no reuse)
provideAnimationsAsync()
providePrimeNG({ theme: Aura })       // custom primary: #AE285D
provideHttpClient(
  withInterceptors([
    errorInterceptor,
    encryptionInterceptor,            // conditional on requestEncryptionEnabled
    authInterceptor
  ])
)
// APP_INITIALIZER: session handler (refresh vs new tab detection)
// ConfigService with injection tokens: API_BASE_URL, ENCRYPTION_KEY, etc.
// LOGOUT_HANDLER injection
```

#### app.routes.ts — Route Map

| Path | Component | Guard |
|------|-----------|-------|
| `/home` | HomeComponent | AuthGuard |
| `/dashboard` | DashboardComponent | AuthGuard |
| `/report-access` | ReportAccessComponent | MenuGuard (7000) |
| `/master-uploads` | MasterUploadsComponent | MenuGuard (6001) |
| `/login` | LoginComponent | — |
| `/reset-password` | ResetPasswordComponent | — |
| `/change-password` | ChangePasswordComponent | — |
| `**` | — | ErrorPagesRoutes (from shared-lib) |
| (default) | — | redirect → `/login` |

#### app.component.ts — Bootstrap Behavior

- Handles `beforeunload` event → sends logout beacon on tab close
- Tab close detection via sessionStorage flag
- Initializes PrimeNG ripple effect globally
- Subscribes to router events → logs page views via `ActivityLogService`

---

### 3.2 projects/shared-lib

**Type:** Angular Library
**Prefix:** `lib`
**Build output:** `dist/shared-lib`
**Entry point:** `projects/shared-lib/src/public-api.ts`

#### Directory Layout

```
projects/shared-lib/src/lib/
├── auth/
│   ├── auth.interceptor.ts             # JWT token attachment (3.8 KB)
│   ├── auth.token.ts                   # Injection token
│   ├── auth-state.service.ts           # Angular Signals auth state (4.2 KB)
│   └── provide-auth-interceptor.ts     # provideAuthInterceptor() helper
├── components/
│   ├── grid/
│   │   ├── grid.component.ts           # Reusable data grid (PrimeNG Table)
│   │   └── model/
│   │       ├── grid-column-definition.ts
│   │       └── table-parameter.ts
│   ├── remark-modal/
│   │   └── remark-modal.component.ts   # Authorize/Reject remark dialog
│   └── session-timer/
│       ├── session-timer.component.ts  # Session countdown UI
│       └── session-timer.service.ts    # Idle detection trigger
├── directives/
│   ├── lowerCaseInput.directive.ts
│   ├── restrict-decimal-input.directive.ts
│   ├── restrictInput.directive.ts
│   └── upperCaseText.directive.ts
├── error-pages/
│   ├── app.accessdenied.component.ts
│   ├── app.notfound.component.ts
│   ├── comming-soon.component.ts
│   ├── session-expired.component.ts
│   └── error-pages.routes.ts           # Catch-all error routes
├── guards/
│   ├── auth.guard.ts                   # Redirects to /login if unauthenticated (928 B)
│   └── menu.guard.ts                   # MenuId-based feature access (1.5 KB)
├── helpers/
│   └── jwt.helper.ts                   # JWT decode helpers
├── interceptors/
│   ├── encryption.interceptor.ts       # AES-256-CBC request encryption
│   └── error.interceptor.ts            # Global HTTP error handler
├── models/
│   ├── icomplaint.ts
│   └── iuser.ts
└── services/
    ├── activity-log.service.ts         # Page view + action logging (2.8 KB)
    ├── common.service.ts               # Shared API helpers (3.7 KB)
    ├── config.service.ts               # Reads injection tokens (1.2 KB)
    ├── crypto.service.ts               # AES-256-CBC encrypt/decrypt (3.7 KB)
    ├── idle-detection.service.ts       # User inactivity tracker (2.9 KB)
    ├── local-storage.service.ts        # Encrypted localStorage wrapper (4 KB)
    └── utils.service.ts                # General utilities (19.7 KB — LARGEST)
```

#### public-api.ts — Full Export Surface

```typescript
// Auth
export { AuthInterceptor, AuthToken, AuthStateService, provideAuthInterceptor }

// Components
export { GridComponent }
export { GridColumnDefinition, TableParameter }   // grid models
export { RemarkModalComponent }
export { SessionTimerComponent, SessionTimerService }

// Error Pages
export { AppNotFoundComponent, ErrorPagesRoutes }

// Guards
export { AuthGuard, MenuGuard }

// Helpers
export { JwtHelper }

// Interceptors
export { EncryptionInterceptor, ErrorInterceptor }

// Models
export { IComplaint, IUser }

// Services
export {
  ActivityLogService, CommonService, ConfigService,
  CryptoService, IdleDetectionService, LocalStorageService, UtilsService
}

// Utilities
export { COMMON_MODULE }
```

#### AuthStateService — Signals-Based Auth State

```typescript
// Signals
authUser          signal<IUser | null>
isLoggedIn        computed(() => authUser() !== null)
userLoginId       computed from authUser
timeKey           computed from authUser
token             computed from authUser
refreshToken      computed from authUser
isTokenExpired    computed (with -10s offset for safety)
tokenExpirationTime computed from token

// Methods
loginUser(user: IUser)
logoutUser()
updateToken(accessToken, refreshToken)
shouldRefreshToken(thresholdInSeconds = 300)   // true if expiring within 5 min
updateADData(adData)                            // role determination for AD logins
```

---

## 4. Backend Structure — ASP.NET Core

**Location:** `backend/src/CrismacAxisNpaApi/`
**Framework:** .NET 9, ASP.NET Core Web API
**ORM:** Dapper (micro-ORM, raw SQL)
**DB:** SQL Server
**Port:** 7294 (HTTPS), 5243 (HTTP)

### Project File — Key NuGet Packages

| Package | Version | Purpose |
|---------|---------|---------|
| `Dapper` | 2.1.66 | Micro-ORM for SQL Server |
| `Microsoft.Data.SqlClient` | 5.2.0 | SQL Server driver |
| `Microsoft.EntityFrameworkCore` | 8.0.2 | EF Core (migrations/scaffolding) |
| `Microsoft.AspNetCore.Authentication.JwtBearer` | 8.0.4 | JWT middleware |
| `BCrypt.Net-Next` | 4.0.3 | Password hashing |
| `EPPlus` | 8.2.1 | Excel read/write |
| `ExcelDataReader` | 3.8.0 | Alternative Excel parser |
| `Serilog.AspNetCore` | 5.0.0 | Structured logging |
| `Swashbuckle.AspNetCore` | 6.6.2 | Swagger/OpenAPI |
| `Newtonsoft.Json` | 13.0.4 | JSON serialization |
| `System.DirectoryServices` | 10.0.0 | Active Directory (LDAP) |
| `SonarAnalyzer.CSharp` | 10.15.0 | Static analysis |

### Directory Layout

```
backend/src/CrismacAxisNpaApi/
├── Constants/                           # Application constants
├── Controllers/
│   ├── AppLogsController.cs             # 1.7 KB — Activity log retrieval
│   ├── ChunkUploadController.cs         # 26 KB — Excel upload pipeline (MAIN)
│   ├── LoginController.cs               # 23 KB — Auth: login/OTP/SSO/refresh
│   ├── ReportAcessController.cs         # Report access management
│   ├── ResourcesController.cs           # 6.9 KB — Master data resources
│   └── UsersController.cs               # 1.8 KB — User management
├── Data/
│   ├── DapperContext.cs                 # IDbConnection factory
│   └── IDapperContext.cs
├── Entities/                            # DTOs / Request-Response models
│   ├── ApiResponseDto.cs
│   ├── AppLogDto.cs
│   ├── AutheticateRequestwithOtp.cs
│   ├── AutheticateResponse.cs
│   ├── AuthorizationResult.cs
│   ├── HeaderDefinition.cs
│   ├── RefreshToken.cs
│   ├── RequestLogEntry.cs
│   ├── TokenApiModel.cs
│   ├── UploadMasterEntity.cs            # 4.2 KB
│   └── User.cs                          # 7.9 KB
├── Extensions/
│   ├── AppSettingsExtension.cs
│   ├── JwtSecurityTokenHandlerExtension.cs
│   └── (other extension methods)
├── Filters/
│   └── ActionLoggingFilter.cs           # Logs all controller actions
├── Helpers/
│   ├── AppSettings.cs
│   ├── CorsSettings.cs
│   ├── DbErrorHandler.cs
│   ├── UploadSettings.cs
│   └── (other helpers)
├── Middlewares/
│   ├── CorrelationHeadersMiddleware.cs  # Injects X-Correlation-Id per request
│   ├── CurrentUserEnrichmentMiddleware.cs # Adds user context to log scope
│   ├── RequestResponseLoggingMiddleware.cs # Full req/res body logging
│   └── SqlDebugMiddleware.cs
├── Repositories/
│   ├── ChunkUploadRepository/           # IChunkUploadRepository + impl
│   ├── IRefreshTokenRepository/         # IRefreshTokenRepository + impl
│   ├── LoginRepository/                 # ILoginRepository + impl
│   ├── ReportAccessRepository/          # IReportAcessRepository + impl
│   ├── UserRpository/                   # IUserRepository + impl
│   └── DatabaseLogService.cs
├── Resources/
│   └── Export/                          # Excel export templates
├── Services/
│   ├── AuthorizationService.cs          # 13 KB — role & access checks
│   ├── ChunkUploadService/              # 7-step upload pipeline
│   ├── Crypto/                          # AES-256-CBC encrypt/decrypt
│   ├── ExcelService/                    # EPPlus-based Excel parsing
│   ├── ExceptionErrorFileLogService.cs
│   ├── FileLogService.cs
│   ├── FileValidationService.cs         # 18.8 KB — MIME, signature, whitelist
│   ├── JwtService.cs                    # 5.4 KB — JWT generation & validation
│   ├── NotificationService.cs           # 5.4 KB — Email/SMS
│   ├── PasswordHasher/                  # BCrypt wrapper
│   └── UserService.cs                   # 13.3 KB — user management
├── Utilities/
├── Logs/
│   ├── ApiLogs/
│   └── AppLogs/
├── Properties/PublishProfiles/
├── appsettings.json
├── appsettings.Development.json
└── Program.cs                           # 12.7 KB — app bootstrap
```

### Controllers — API Surface

#### ChunkUploadController (Main Feature)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/ChunkUpload/upload` | Upload Excel file |
| GET | `/api/ChunkUpload/meta` | Get upload type metadata |
| POST | `/api/ChunkUpload/list/{operationFlag}` | Paginated grid data |
| POST | `/api/ChunkUpload/authorize/{uniqueUploadId}` | Maker-Checker authorize |
| POST | `/api/ChunkUpload/reject/{uniqueUploadId}` | Maker-Checker reject |
| GET | `/api/ChunkUpload/download/{uniqueUploadId}` | Download file/error report |

#### LoginController

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/Login/CrisMAc/SelectLoginDetails` | Standard login + OTP |
| POST | `/api/Login/CrisMAc/SSOAuth` | SSO auto-login |
| POST | `/api/Login/CrisMAc/retrieveToken` | Refresh token |
| GET | `/api/Login/logout/{userId}` | Logout (invalidate refresh token) |
| POST | `/api/Login/CrisMAc/verifyOtp` | OTP verification |
| POST | `/api/Login/CrisMAc/resetPassword` | Password reset |
| POST | `/api/Login/CrisMAc/changePassword` | Change password |

### Program.cs — Middleware Pipeline Order

```
1. Kestrel limits (body: 100 MB, header timeout: 5 min, keepalive: 10 min)
2. Serilog request logging
3. CORS (from CorsSettings)
4. CorrelationHeadersMiddleware
5. RequestResponseLoggingMiddleware (if enabled)
6. CurrentUserEnrichmentMiddleware
7. JWT Authentication
8. Authorization
9. ActionLoggingFilter (global)
10. Controller routing
```

### appsettings.json — Key Configuration

```json
{
  "ConnectionStrings": {
    "ConnectionString": "Server=...;Database=SAMPLEDB;...",
    "ConnectionStringDailyDB": "Server=...;Database=SAMPLE_DAILY;..."
  },
  "JWTSettings": {
    "Secret": "YourSecretKey",
    "Issuer": "https://localhost:5001",
    "Audience": "http://localhost:4200",
    "TokenExpiryInSeconds": 600
  },
  "AppSettings": {
    "AreaName": "LAB",
    "RefreshTokenTTL": 2,
    "SessionTimeout": 30,
    "LoginOtpEnabled": false,
    "CheckUserLogged": false,
    "RequestEncryptionEnabled": false,
    "UseAuthenticatedAD": false
  },
  "UploadSettings": {
    "MaxFileSizeMB": 100,
    "MaxRowsPerUpload": 1100000,
    "AllowedExtensions": [".xls", ".xlsx"],
    "BulkCopyTimeoutSeconds": 900,
    "ValidationTimeoutSeconds": 600,
    "InsertTimeoutSeconds": 900,
    "RateLimitPerMinute": 60,
    "EnableFileSignatureValidation": true,
    "EnableSecurityLogging": true
  }
}
```

---

## 5. AREAS/ Folder

> **Status: NOT PRESENT in this repository.**

The top-level directory structure contains:
- `frontend/`
- `backend/`
- `doc/`
- `deploy/`

There is **no `AREAS/` folder** at the monorepo root. The concept of "AREAS" may relate to the `AreaName` configuration value in `appsettings.json` (currently set to `"LAB"`), which is a runtime string used to differentiate deployment areas/regions — not a source code folder.

**If `AREAS/` is planned:** This would be a new top-level folder to be created as part of monorepo expansion. See [Section 14](#14-expansion-planning-notes) for recommendations.

---

## 6. doc/ Folder

**Location:** `doc/`

```
doc/
├── README.md                            # Documentation index & navigation guide
├── HANDOVER-PLAN.md                     # Project handover planning
├── TECHNICAL_DOCUMENTATION (1).md      # 33 KB — Combined technical doc
├── MONOREPO-OVERVIEW.md                 # This file
├── PART-1-BUSINESS-LOGIC/
│   ├── 1-business-requirements.md      # What the upload system does
│   ├── 2-technical-documentation.md   # Architecture & module details
│   ├── 3-flow-tree.md                  # Execution flow (text format)
│   ├── 4-mermaid-diagram.md            # Mermaid flowcharts
│   ├── 5-requirements-edge-cases.md   # 30+ edge cases, security
│   └── 6-test-coverage.md             # 100+ test cases
└── PART-2-AUTHENTICATION/
    ├── 1-business-requirements.md      # Auth user flows
    ├── 2-technical-documentation.md   # JWT/OTP/SSO architecture
    ├── 3-flow-tree.md                  # Login/logout flow trees
    ├── 4-mermaid-diagram.md            # Auth flow diagrams
    ├── 5-requirements-edge-cases.md   # 18+ edge cases
    └── 6-test-coverage.md             # Security & integration tests
```

### Documentation Coverage

| Area | Business Req | Tech Doc | Flow | Diagram | Edge Cases | Tests |
|------|-------------|---------|------|---------|------------|-------|
| Upload Pipeline | ✅ | ✅ | ✅ | ✅ | ✅ 30+ | ✅ 100+ |
| Authentication | ✅ | ✅ | ✅ | ✅ | ✅ 18+ | ✅ |

---

## 7. deploy/ Folder

**Location:** `deploy/`

Contains deployment configurations and scripts. Exact content not enumerated here — typically includes:
- Docker / docker-compose files
- IIS deployment scripts
- Environment-specific config overlays
- CI/CD pipeline definitions

---

## 8. Key Config Files

### angular.json — Workspace Summary

| Project | Type | Prefix | Output | Default Config |
|---------|------|--------|--------|---------------|
| `npa-app` | application | `app` | `dist/npa-app` | production |
| `shared-lib` | library | `lib` | `dist/shared-lib` | production |

**Notable angular.json settings:**
- `packageManager: "pnpm"`
- `stylePreprocessorOptions.includePaths` — shared SCSS
- Assets include `shared-public/` (shared static files)
- CommonJS dependencies allowed: `crypto-js`, `inputmask`, `file-saver`
- SSR configuration available (`@angular/ssr`)

### tsconfig.json (Frontend)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "strict": true,
    "paths": { "shared-lib": ["./dist/shared-lib"] }
  }
}
```

### global.json (Root)

```json
{
  "sdk": {
    "version": "9.0.111",
    "rollForward": "latestFeature",
    "allowPrerelease": false
  }
}
```

---

## 9. Technology Stack Summary

### Frontend

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | Angular (standalone components) | 18.2.14 |
| UI Library | PrimeNG + PrimeFlex | 18.0.2 / 3.3.1 |
| State | Angular Signals | 18.x |
| HTTP | @angular/common/http + interceptors | 18.x |
| Styling | SCSS, PrimeFlex, PrimeIcons | — |
| Charts | Chart.js | — |
| Rich Text | Quill | — |
| Encryption | crypto-js (AES-256-CBC) | — |
| Auth | JWT decode (JwtHelper) | — |
| Build | Angular CLI, ng-packagr | 18.2.14 |
| Testing | Jasmine + Karma | — |
| Lint | ESLint + @angular-eslint + Prettier | 18.4.2 |

### Backend

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | ASP.NET Core Web API | .NET 9.0 |
| ORM | Dapper | 2.1.66 |
| Database | SQL Server | — |
| Auth | JWT Bearer (HMAC-SHA256) | — |
| Password | BCrypt.Net-Next | 4.0.3 |
| Excel | EPPlus + ExcelDataReader | 8.2.1 / 3.8.0 |
| Logging | Serilog (structured, JSON) | 3.1.1 |
| AD Integration | System.DirectoryServices | 10.0.0 |
| API Docs | Swagger/OpenAPI (Swashbuckle) | 6.6.2 |
| Analysis | SonarAnalyzer.CSharp | 10.15.0 |

### Infrastructure / Tooling

| Tool | Version | Role |
|------|---------|------|
| pnpm | 9.15.0 | JS package manager |
| Node.js | 20.19.5 | JS runtime |
| Volta | — | Node/pnpm version pinning |
| .NET SDK | 9.0.111 | Backend build |
| Husky | 9.0.11 | Git hooks |
| Commitlint | 18.4.3 | Commit message enforcement |
| Lint-staged | 15.2.0 | Pre-commit linting |

---

## 10. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        BROWSER (Port 4200)                       │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                       npa-app (Angular 18)                   │ │
│  │                                                               │ │
│  │  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌───────────┐  │ │
│  │  │  login   │  │dashboard │  │  master-  │  │  report-  │  │ │
│  │  │  pages   │  │          │  │  uploads  │  │  access   │  │ │
│  │  └──────────┘  └──────────┘  └───────────┘  └───────────┘  │ │
│  │                                                               │ │
│  │  ┌─────────────────────────────────────────────────────────┐ │ │
│  │  │                    shared-lib                            │ │ │
│  │  │  AuthStateService │ ConfigService │ CryptoService        │ │ │
│  │  │  GridComponent    │ AuthGuard     │ MenuGuard            │ │ │
│  │  │  ErrorInterceptor │ AuthInterceptor│EncryptionInterceptor│ │ │
│  │  │  SessionTimer     │ IdleDetection │ LocalStorage         │ │ │
│  │  └─────────────────────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │ HTTPS (JWT in header)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   ASP.NET Core API (Port 7294)                   │
│                                                                   │
│  Middleware:  CORS → CorrelationId → ReqResLog → UserEnrich      │
│                                                                   │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌────────────────────┐ │
│  │ Login    │ │ Chunk    │ │ Resources │ │   Users / Logs     │ │
│  │ Controller│ │ Upload   │ │ Controller│ │   Controllers      │ │
│  └────┬─────┘ └────┬─────┘ └─────┬─────┘ └────────────────────┘ │
│       │             │              │                               │
│  ┌────▼─────┐ ┌─────▼────┐ ┌──────▼────┐                        │
│  │  Auth    │ │ Upload   │ │   User    │                        │
│  │ Service  │ │ Service  │ │  Service  │                        │
│  │  + JWT   │ │ + Excel  │ │           │                        │
│  └────┬─────┘ └────┬─────┘ └──────┬────┘                        │
│       │             │              │                               │
│  ┌────▼─────────────▼──────────────▼────┐                        │
│  │           Dapper + SQL Server         │                        │
│  │    (Stored Procedures, Bulk Insert)   │                        │
│  └───────────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       SQL Server                                  │
│    SAMPLEDB (main)     │    SAMPLE_DAILY (daily data)           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Data Flows

### Authentication Flow

```
1.  User enters credentials (username + password)
2.  Frontend: CryptoService encrypts password (AES-256-CBC) if enabled
3.  POST /api/Login/CrisMAc/SelectLoginDetails
4.  Backend: LoginRepository validates against DB or AD
5.  Backend: If OTP enabled → generate 6-digit OTP → NotificationService sends email
6.  Backend: Generate JWT (HS256, 600s expiry) + RefreshToken (64-byte random, 2h TTL)
7.  Backend: Store RefreshToken in DB with userId, expiry, IP
8.  Frontend: AuthStateService.loginUser(user) → stores in localStorage
9.  Frontend: LocalStorageService encrypts storage if STORAGE_ENCRYPTION_ENABLED

Token Refresh (every 5 min):
10. AuthStateService.shouldRefreshToken(300) returns true
11. POST /api/Login/CrisMAc/retrieveToken (with refreshToken)
12. Backend: Validate refreshToken in DB → issue new JWT + new RefreshToken
13. AuthStateService.updateToken(newJWT, newRefreshToken)

Logout:
14. beforeunload event → navigator.sendBeacon (GET /api/Login/logout/{userId})
15. Backend: InvalidateRefreshToken in DB
16. Frontend: AuthStateService.logoutUser() → clear localStorage
```

### File Upload (Maker-Checker) Flow

```
MAKER SIDE:
1.  Select Excel file (.xlsx / .xls, max 100 MB)
2.  Frontend validates: size, extension, MIME type
3.  POST /api/ChunkUpload/upload (multipart/form-data)
4.  Backend: FileValidationService
    ├── Extension whitelist check
    ├── MIME type validation
    ├── Magic bytes (ZIP header) verification
    ├── Filename sanitization (no path traversal)
    └── File size check
5.  Backend: ExcelService parses workbook
6.  Backend: ChunkUploadRepository → SqlBulkCopy into staging table
7.  Backend: Execute validation stored procedure
8.  If validation fails → generate error Excel → return download link
9.  If validation passes → Execute insert stored procedure
10. Backend: Create UploadMasterEntity record (status: PENDING)
11. Frontend: Grid refreshes → shows pending record

CHECKER SIDE:
12. GET /api/ChunkUpload/list/PENDING → grid shows pending uploads
13. Checker reviews → POST /api/ChunkUpload/authorize/{id} or /reject/{id}
14. Backend: AuthorizationService validates Checker ≠ Maker (separation of duty)
15. Backend: Update UploadMasterEntity status → AUTHORIZED / REJECTED
16. Frontend: Grid refreshes with updated status
```

---

## 12. Security Layers

### Authentication & Authorization

| Layer | Mechanism |
|-------|-----------|
| Password | BCrypt hashing (server-side) + AES-256-CBC (transit) |
| Token | JWT HMAC-SHA256, 600s expiry |
| Refresh | 64-byte random, DB-stored, 2h TTL, one-time use |
| OTP | 6-digit, 3-min window, email delivery |
| SSO | Azure AD / Okta compatible |
| Guard | AuthGuard (JWT presence) + MenuGuard (MenuId) |
| Separation | Maker ≠ Checker enforced server-side |

### Request Security

| Layer | Mechanism |
|-------|-----------|
| Transport | TLS (HTTPS, Kestrel) |
| CORS | Allowlist (localhost:4200 in dev; configurable) |
| Request body | Optional AES-256-CBC encryption (encryptionInterceptor) |
| Storage | Optional AES-256-CBC localStorage encryption |
| SQL | Dapper parameterized queries (no raw string SQL) |
| Rate limiting | 60 req/min for upload endpoint |

### File Security

| Check | Implementation |
|-------|---------------|
| Extension | Whitelist: `.xlsx`, `.xls` only |
| MIME type | `application/vnd.ms-excel`, `application/vnd.openxmlformats-...` |
| Magic bytes | ZIP header check (PK\x03\x04 — Excel is ZIP) |
| Filename | Sanitized, no `../`, alphanumeric + safe chars only |
| Size | Max 100 MB configurable |
| Rows | Max 1,100,000 rows |
| Timeout | Validation: 600s, Insert: 900s, BulkCopy: 900s |

### Observability

| Feature | Tool |
|---------|------|
| Structured logs | Serilog (JSON to file + text to console) |
| Correlation ID | Per-request via CorrelationHeadersMiddleware |
| User enrichment | CurrentUserEnrichmentMiddleware adds user context |
| Action logging | ActionLoggingFilter logs all controller calls |
| Request/Response | Optional full body logging (RequestResponseLoggingMiddleware) |
| Log levels | Debug → Info → Warning → Error → Fatal |
| Log paths | `Logs/ApiLogs/`, `Logs/AppLogs/` |

---

## 13. Performance Configuration

| Setting | Value | Location |
|---------|-------|---------|
| Max file upload | 100 MB | `UploadSettings.MaxFileSizeMB` |
| Max request body | 100 MB | Kestrel limits in `Program.cs` |
| Max rows per upload | 1,100,000 | `UploadSettings.MaxRowsPerUpload` |
| JWT expiry | 600 seconds (10 min) | `JWTSettings.TokenExpiryInSeconds` |
| Refresh token TTL | 2 hours | `AppSettings.RefreshTokenTTL` |
| Session timeout | 30 minutes | `AppSettings.SessionTimeout` |
| Token refresh threshold | 300 seconds | `shouldRefreshToken(300)` in shared-lib |
| Upload rate limit | 60 req/min | `UploadSettings.RateLimitPerMinute` |
| BulkCopy timeout | 900 seconds | `UploadSettings.BulkCopyTimeoutSeconds` |
| Validation timeout | 600 seconds | `UploadSettings.ValidationTimeoutSeconds` |
| Request header timeout | 5 minutes | Kestrel `RequestHeadersTimeout` |
| Keep-alive timeout | 10 minutes | Kestrel `KeepAliveTimeout` |

### Performance Targets (from `doc/README.md`)

| Operation | Target |
|-----------|--------|
| Standard login | < 5 seconds |
| OTP delivery | < 2 seconds |
| Token refresh | < 200 ms |
| File upload (100 MB) | < 30 seconds |
| Grid load (1,000 rows) | < 5 seconds |
| File download (100K rows) | < 10 seconds |
| JWT generation | < 100 ms |

---

## 14. Expansion Planning Notes

### Current Monorepo Model

This is a **simple co-located monorepo**: one git repo, two independent workspaces (Angular + .NET), with only git-hook tooling (Husky) at root. There is:
- No build orchestrator (Nx, Turborepo, Lerna)
- No pnpm workspace linking between packages
- One Angular app + one Angular library (already an Angular multi-project workspace)

### Angular Workspace Capacity

The `angular.json` already supports multiple projects. Adding new apps or libs requires:
```bash
ng generate application new-app --project-root=projects/new-app
ng generate library new-lib --project-root=projects/new-lib
```

The `tsconfig.json` path aliases must be updated for any new library:
```json
"paths": {
  "shared-lib": ["./dist/shared-lib"],
  "new-lib": ["./dist/new-lib"]
}
```

### If AREAS/ Is to Be Added

A suggested structure for an `AREAS/` top-level folder (for separate deployable backend areas/micro-services):

```
AREAS/
├── area-lab/                 # Current LAB area (rename from backend/)
│   └── src/CrismacAxisNpaApi/
├── area-retail/              # New area (example)
│   └── src/RetailNpaApi/
└── area-corporate/           # New area (example)
    └── src/CorporateNpaApi/
```

Each area backend would be an independent .NET solution, differentiated by `AreaName` in appsettings.

### If Nx Is Desired

To add Nx to the existing Angular workspace:
```bash
cd frontend
npx nx@latest init
```

Nx would provide:
- Affected builds (only rebuild changed projects)
- Dependency graph visualization
- Caching (local + optional Nx Cloud)
- Task orchestration replacing the manual `concurrently`/`wait-on` scripts

### Shared-lib Expansion Consideration

Currently `shared-lib` exports auth, guards, grid, session timer, interceptors, and services — all tightly coupled to NPA app patterns. If expanding to multiple apps:
1. Split `shared-lib` into domain-agnostic (`ui-lib`) and NPA-specific (`npa-shared-lib`)
2. Or scope exports carefully and version the library

---

## Quick Reference

| Need | Go To |
|------|-------|
| Add a new Angular feature module | `frontend/projects/npa-app/src/app/pages/` |
| Add a shared component | `frontend/projects/shared-lib/src/lib/components/` |
| Export from shared-lib | `frontend/projects/shared-lib/src/public-api.ts` |
| Add a new API endpoint | `backend/src/CrismacAxisNpaApi/Controllers/` |
| Add a new repository | `backend/src/CrismacAxisNpaApi/Repositories/` |
| Change upload limits | `backend/src/CrismacAxisNpaApi/appsettings.json` → `UploadSettings` |
| Change JWT expiry | `backend/src/CrismacAxisNpaApi/appsettings.json` → `JWTSettings` |
| Change theme color | `frontend/projects/npa-app/src/app/app.config.ts` → `providePrimeNG` |
| Change allowed origins | `backend/src/CrismacAxisNpaApi/appsettings.json` → `CorsSettings` |
| Enable OTP login | `backend/src/CrismacAxisNpaApi/appsettings.json` → `LoginOtpEnabled: true` |
| Enable request encryption | `appsettings.json` → `RequestEncryptionEnabled: true` |

---

*Last updated: 2026-04-04 — generated from direct codebase analysis*
