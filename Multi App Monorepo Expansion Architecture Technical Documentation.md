# CrisMAC AXIS NPA — Multi App Monorepo Plan Expansion Comprehensive Overview

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

## 14. Expansion Planning — Multi-Application Architecture

> **Objective:** Grow this co-located monorepo into a multi-application platform with zero disruption to `npa-app`/`npa-api`. Every step below is **additive**. Existing code builds and deploys identically throughout the migration.

---

### 14.A Current State & What Changes

**What exists today (unchanged):**
- Simple co-located monorepo — no build orchestrator (no Nx, Lerna, Turborepo)
- Angular workspace with 2 projects: `npa-app` (app) + `shared-lib` (library)
- One ASP.NET Core API at `backend/src/CrismacAxisNpaApi/`
- Root tooling: Husky + Commitlint only

**What the expansion adds:**
- `frontend/projects/adf-app/` — second Angular app sharing the same `shared-lib`
- `backend/src/Common.Core/` — .NET 9 class library (extracted shared infrastructure)
- `backend/src/AREAS/npa-api/` — current NPA API moved here (folder only, no code change)
- `backend/src/AREAS/adf-api/` — new ADF area API referencing Common.Core
- `backend/CrismacAxis.sln` — solution file tying all .NET projects together

---

### 14.B Target Repository Structure

```
crismac-npa-axis/
├── frontend/
│   ├── projects/
│   │   ├── shared-lib/            # existing — 2 new injection tokens added
│   │   ├── npa-app/               # existing — ZERO changes
│   │   └── adf-app/               # NEW Angular app
│   ├── angular.json               # updated: adf-app project added, port 4201
│   ├── package.json               # updated: new build/serve scripts
│   └── tsconfig.json              # unchanged (shared-lib path alias inherited)
│
├── backend/
│   ├── CrismacAxis.sln            # NEW .NET solution file
│   └── src/
│       ├── Common.Core/           # NEW shared .NET class library
│       │   └── Crismac.Axis.Common.Core.csproj
│       └── AREAS/
│           ├── npa-api/           # MOVED from src/CrismacAxisNpaApi/ (folder rename only)
│           │   └── Crismac.Axis.Npa.Api.csproj  ← references Common.Core
│           └── adf-api/           # NEW area API
│               └── Crismac.Axis.Adf.Api.csproj  ← references Common.Core
│
├── doc/
└── deploy/
```

---

### 14.C Frontend Expansion — Adding adf-app

#### C.1 Angular Workspace Scaffold

```bash
cd frontend
ng generate application adf-app --style scss --routing false --standalone --prefix app
```

This appends `adf-app` build/serve/test targets into `angular.json`. The `--routing false` flag is intentional — routes are defined manually in `app.routes.ts` following the npa-app pattern.

Updated `angular.json` project list:

| Project | Type | Prefix | Output | Dev Port |
|---------|------|--------|--------|---------|
| `npa-app` | application | `app` | `dist/npa-app` | 4200 |
| `adf-app` | application | `app` | `dist/adf-app` | 4201 |
| `shared-lib` | library | `lib` | `dist/shared-lib` | — |

Add port to `adf-app`'s serve target in `angular.json`:
```json
"serve": {
  "builder": "@angular-devkit/build-angular:dev-server",
  "options": { "port": 4201 },
  "configurations": {
    "production":   { "buildTarget": "adf-app:build:production" },
    "development":  { "buildTarget": "adf-app:build:development" }
  },
  "defaultConfiguration": "development"
}
```

Assets and styles for `adf-app` build target:
```json
"assets": [
  { "glob": "**/*", "input": "projects/adf-app/public" },
  { "glob": "**/*", "input": "shared-public" }
],
"styles": [
  "shared-public/assets/layout/css/layout-light.css",
  "projects/adf-app/src/styles.scss"
]
```

#### C.2 shared-lib Changes (Minimal)

Two hardcoded strings in shared-lib must become injectable to support multiple apps:

**Problem 1:** `ConfigService.getActivityLogApiUrl()` has `'CrisMAc/AppLogsUpdate'` hardcoded. ADF uses a different endpoint prefix.

**Problem 2:** No way for shared services to know which app they are running inside.

**Fix — add two injection tokens to `shared-lib/src/lib/auth/auth.token.ts`:**

```typescript
// Path prefix for the activity log endpoint — app-specific
export const ACTIVITY_LOG_PATH = new InjectionToken<string>(
  'ACTIVITY_LOG_PATH',
  { providedIn: 'root', factory: () => 'CrisMAc/AppLogsUpdate' }
);

// Identifies the host application for logging and telemetry
export const APP_NAME = new InjectionToken<string>(
  'APP_NAME',
  { providedIn: 'root', factory: () => 'unknown-app' }
);
```

Update `ConfigService.getActivityLogApiUrl()` to inject and use `ACTIVITY_LOG_PATH` instead of the hardcoded string.

Export both tokens from `public-api.ts`:
```typescript
export { ACTIVITY_LOG_PATH, APP_NAME } from './lib/auth/auth.token';
```

**npa-app requires zero changes** — the default factory value `'CrisMAc/AppLogsUpdate'` matches exactly what npa-app uses today.

**No other shared-lib changes are needed.** All interceptors, guards, services, components already operate generically via injection tokens.

#### C.3 Per-App Mandatory File Set

Every app in the workspace is self-contained in its own `projects/<app-name>/src/`. The following is the **required file set** for any new app:

```
projects/adf-app/src/
├── index.html                          # App title, favicon
├── main.ts                             # bootstrapApplication(AppComponent, appConfig)
├── styles.scss                         # App-level SCSS overrides
├── environments/
│   ├── environment.ts                  # Dev defaults
│   ├── environment.dev.ts              # Dev server overrides (ssoEnabled, host URL)
│   └── environment.prod.ts             # Production values
└── app/
    ├── app.component.html
    ├── app.component.ts                # Tab-close handler, ActivityLogService wiring
    ├── app.config.ts                   # DI providers: theme, router, interceptors, tokens
    ├── app.routes.ts                   # ADF-specific routes + MenuIds
    ├── core/
    │   ├── services/
    │   │   ├── auth.service.ts         # Calls api/Login/ADF/... endpoints
    │   │   └── core.service.ts         # Token refresh interval, logout orchestration
    │   └── menu/
    │       ├── menu.service.ts         # Signal-based menu state
    │       ├── menu-api.service.ts     # Fetches menus from adf-api
    │       ├── menu.model.ts
    │       └── menu-list-response-model.ts
    ├── layout/                         # Topbar, sidebar, footer — rebranded for ADF
    │   ├── app.main.component.ts / .html
    │   ├── app.menu.component.ts
    │   ├── app.menuitem.component.ts
    │   ├── app.topbar.component.ts
    │   ├── app.footer.component.ts
    │   ├── app.menu.service.ts
    │   └── app.rightpanel.component.html
    ├── pages/
    │   └── auth/
    │       ├── login/                  # ADF-branded login (logo, color, title)
    │       │   ├── login.component.ts
    │       │   ├── login.component.html
    │       │   └── login.component.scss
    │       ├── reset-password/
    │       └── change-password/
    └── utilities/
        └── common-module.ts
```

**File action summary for each new app:**

| File | Action | What changes |
|------|--------|-------------|
| `app.config.ts` | New | PrimeNG theme color, `API_BASE_URL`, `APP_NAME`, `ACTIVITY_LOG_PATH` |
| `environments/environment*.ts` | New | Host URL, `ssoEnabled`, feature flags |
| `core/services/auth.service.ts` | New | Login endpoint prefix (e.g. `api/Login/ADF/`) |
| `core/services/core.service.ts` | Copy from npa-app | No changes needed |
| `pages/auth/login/` | Copy + rebrand | Logo `<img>`, heading text, SCSS color variable |
| `layout/` | Copy + rebrand | App name/logo in topbar template |
| `app.routes.ts` | New | App-specific routes + MenuIds |
| `core/menu/` | Copy from npa-app | No changes needed |
| `main.ts` | Generated | Identical pattern to npa-app |

#### C.4 Theme and Branding Per App

Branding is entirely controlled by two per-app files — no shared-lib involvement:

**PrimeNG theme (color palette)** — defined in `app.config.ts`:
```typescript
// npa-app: crimson-pink
const npaThemePreset = definePreset(Aura, {
  semantic: { primary: { 500: '#AE285D', /* ... full palette */ } }
});

// adf-app: example blue
const adfThemePreset = definePreset(Aura, {
  semantic: { primary: { 500: '#1A6B8A', /* ... full palette */ } }
});
```

Each app's `app.config.ts` calls `providePrimeNG({ theme: { preset: <appThemePreset> } })` — they never conflict.

**Login page branding** — the login component template contains the logo `<img>` and app title heading. These are local to each app's `pages/auth/login/`. The SCSS file overrides the color variable for that app's login screen.

**Login URL** — configured in each app's `auth.service.ts`. The endpoint prefix (`CrisMAc` vs `ADF`) is a one-line string constant per app.

#### C.5 Build and Serve Scripts

Add to `frontend/package.json`:

```json
{
  "scripts": {
    "start:dev":          "concurrently -k \"pnpm run watch-lib\" \"pnpm run serve-npa-app:dev\"",
    "serve-npa-app:dev":  "wait-on dist/shared-lib/fesm2022/shared-lib.mjs && ng serve npa-app --hmr --configuration development --port 4200",

    "start:adf-dev":      "concurrently -k \"pnpm run watch-lib\" \"pnpm run serve-adf-app:dev\"",
    "serve-adf-app:dev":  "wait-on dist/shared-lib/fesm2022/shared-lib.mjs && ng serve adf-app --hmr --configuration development --port 4201",

    "build:shared-lib":   "pnpm run clean-lib && ng build shared-lib",
    "build:npa-app":      "ng build npa-app",
    "build:adf-app":      "ng build adf-app",
    "build:all":          "pnpm run build:shared-lib && pnpm run build:npa-app && pnpm run build:adf-app",

    "watch-lib":          "ng build shared-lib --watch",
    "clean-lib":          "rimraf dist/shared-lib"
  }
}
```

> **Pattern for adding app N:** Copy the `start:adf-dev` / `serve-adf-app:dev` / `build:adf-app` triplet, substitute the app name and pick an unused port (4202, 4203, …). Add the app to `build:all`.

---

### 14.D Backend — Common.Core Shared Library

#### D.1 Project Setup

```bash
cd backend/src
mkdir Common.Core && cd Common.Core
dotnet new classlib -n Crismac.Axis.Common.Core --framework net9.0
dotnet sln ../../CrismacAxis.sln add Crismac.Axis.Common.Core.csproj
```

`Common.Core` is a `Microsoft.NET.Sdk` class library (not Web) — no controllers, no `Program.cs`, no HTTP pipeline.

**Required NuGet packages in `Crismac.Axis.Common.Core.csproj`:**
```xml
<PackageReference Include="BCrypt.Net-Next"                            Version="4.0.3" />
<PackageReference Include="Dapper"                                     Version="2.1.66" />
<PackageReference Include="EPPlus"                                     Version="8.2.1" />
<PackageReference Include="ExcelDataReader"                            Version="3.8.0" />
<PackageReference Include="Microsoft.AspNetCore.Http.Abstractions"     Version="2.3.0" />
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.4" />
<PackageReference Include="Microsoft.Data.SqlClient"                   Version="5.2.0" />
<PackageReference Include="Microsoft.Extensions.Options"               Version="9.0.0" />
<PackageReference Include="Microsoft.Extensions.Caching.Memory"        Version="9.0.0" />
<PackageReference Include="Newtonsoft.Json"                            Version="13.0.4" />
<PackageReference Include="Serilog"                                    Version="3.1.1" />
<PackageReference Include="System.DirectoryServices"                   Version="10.0.0" />
```

#### D.2 What Moves to Common.Core

**Services → `Common.Core/Services/`**

| File | From npa-api | Changes needed |
|------|-------------|----------------|
| `Crypto/ICryptoService.cs` | Move | Namespace only |
| `Crypto/CryptoAes256CbcService.cs` | Move | Namespace only |
| `PasswordHasher/IPasswordHasher.cs` | Move | Namespace only |
| `PasswordHasher/BcryptPasswordHasher.cs` | Move | Namespace only |
| `ExcelService/IExcelService.cs` | Move | Namespace only |
| `ExcelService/ExcelService.cs` | Move | Namespace only |
| `NotificationService.cs` | Move | Namespace only |
| `FileLogService.cs` | Move | Namespace only |
| `ExceptionErrorFileLogService.cs` | Move | Namespace only |
| `FileValidationService.cs` | Move | Namespace only |
| `JwtService.cs` | Move + modify | Parameterize claim names (see D.5) |

**Middlewares → `Common.Core/Middlewares/`**

| File | Changes needed |
|------|----------------|
| `CorrelationHeadersMiddleware.cs` | Namespace only |
| `EncryptionMiddleware.cs` | Namespace only |
| `ExceptionHandlerMiddleware.cs` | Namespace only |
| `RequestResponseLoggingMiddleware.cs` | Namespace only |
| `SqlTraceMiddleware.cs` | Namespace only |
| `CurrentUserEnrichmentMiddleware.cs` | Depend on `IUserRepository` interface (see E.2) |

**Data → `Common.Core/Data/`**

| File | Changes needed |
|------|----------------|
| `IDapperContext.cs` | Add `CreateNamedConnection(string name)` overload |
| `DapperContext.cs` | Implement new overload; namespace only otherwise |

**Entities → `Common.Core/Entities/`**

| File | Notes |
|------|-------|
| `ApiResponseDto.cs` | Generic `ApiResponse<T>` wrapper |
| `RefreshToken.cs` | Generic token entity |
| `TokenApiModel.cs` | Generic token DTO |
| `RequestLogEntry.cs` | Generic request log |
| `AppLogDto.cs` | Generic activity log DTO |

**Helpers → `Common.Core/Helpers/`**

| File | Notes |
|------|-------|
| `AppSettings.cs` (all classes) | Move entire file: `ConnectionStrings`, `JWTSettings`, `AppSettings`, `SmsOptions`, `EmailOptions`, `UploadSettings`, `CorsSettings`, `CustomLogging`, `CustomUserSettings` |
| `JwtClaimsMap.cs` | NEW — see D.5 |
| `CorsSettings.cs` | Move |
| `AppException.cs` | Move |

**Utilities → `Common.Core/Utilities/`**

| File | Notes |
|------|-------|
| `OtpGenerator.cs` | Pure static, zero deps |
| `LoggingUtil.cs` | Move |
| `UrlUtils.cs` | Move |
| `fileUtil.cs` | Move |
| `MenuGuardConstants.cs` | Move |

**Constants → `Common.Core/Constants/`**

| File | Notes |
|------|-------|
| `HttpStatusCodes.cs` | Move |

**Filters → `Common.Core/Filters/`**

| File | Notes |
|------|-------|
| `ActionLoggingFilter.cs` | Move |
| `MenuGuardAttribute.cs` | Move |
| `RateLimitAttribute.cs` | Move |
| `RequestLimitAttribute.cs` | Move |

**Repository Interfaces → `Common.Core/Repositories/`**

These are minimal contracts that `CurrentUserEnrichmentMiddleware` and `JwtService` depend on. Each area-api provides its own implementation:

```csharp
// Common.Core/Repositories/IUserRepository.cs
public interface IUserRepository
{
    Task<UserDetails?> GetByUserIdAsync(string userId);
}

// Common.Core/Repositories/IRefreshTokenRepository.cs
public interface IRefreshTokenRepository
{
    bool IsTokenUnique(string token);
    Task UpdateRefreshTokenAsync(UpdateRefreshTokenRequest request);
}
```

**Extension Methods → `Common.Core/Extensions/ServiceCollectionExtensions.cs`**

```csharp
// Registers all shared settings POCOs (ConnectionStrings, JWTSettings, AppSettings, etc.)
public static void AddCommonAppSettings(this IServiceCollection services, IConfiguration configuration)

// Registers all Common.Core services (ICryptoService, IPasswordHasher, JwtService,
// NotificationService, FileValidationService, FileLogService, ExcelService, etc.)
public static IServiceCollection AddCommonModelServices(this IServiceCollection services)

// Registers IDapperContext / DapperContext
public static IServiceCollection AddDapperContext(this IServiceCollection services)

// Configures JWT Bearer authentication from JWTSettings
public static IServiceCollection AddCommonJwtAuthentication(
    this IServiceCollection services, IConfiguration configuration)

// Registers middleware types for DI (CorrelationHeaders, CurrentUserEnrichment, etc.)
public static IServiceCollection AddCommonMiddleware(this IServiceCollection services)
```

#### D.3 What Stays in AREAS/npa-api

| Category | Files / Folders | Reason |
|----------|----------------|--------|
| All Repositories | `Repositories/*` | Hardcoded NPA stored procedure names |
| Upload pipeline | `Services/ChunkUploadService/` | NPA domain workflow |
| Auth/role logic | `Services/AuthorizationService.cs` | NPA Maker-Checker operation flags (1, 16, 17) |
| User management | `Services/UserService.cs` | NPA AD lookup + role classification |
| All Controllers | `Controllers/*` | NPA domain endpoints |
| NPA Entities | `Entities/User.cs`, `UploadMasterEntity.cs`, `AuthenticateResponse.cs`, etc. | NPA-specific DTOs |
| App bootstrap | `Program.cs` | Area-specific middleware pipeline |
| NPA DI wiring | `Extensions/ServiceCollectionExtensions.cs` → `AddNpaRepositoryServices()`, `AddNpaModelServices()` | NPA-specific registrations |

#### D.4 IDapperContext — Multi-DB Strategy

The expanded `IDapperContext` in Common.Core supports all three database topologies via configuration only:

```csharp
// Common.Core/Data/IDapperContext.cs
public interface IDapperContext
{
    /// <summary>Primary or daily DB — existing pattern, unchanged</summary>
    IDbConnection CreateConnection(bool isDailyDBCn = false);

    /// <summary>Named connection for areas requiring a third DB</summary>
    IDbConnection CreateNamedConnection(string connectionStringName);
}
```

**Topology 1 — Separate DB per area:** Each area-api's `appsettings.json` points `ConnectionStrings:ConnectionString` to its own database. `DapperContext.CreateConnection()` reads that value. Zero code difference — only config differs.

```json
// AREAS/npa-api/appsettings.json
"ConnectionStrings": { "ConnectionString": "Server=...;Database=NPA_DB;..." }

// AREAS/adf-api/appsettings.json
"ConnectionStrings": { "ConnectionString": "Server=...;Database=ADF_DB;..." }
```

**Topology 2 — Separate schema within same DB:** Same connection string, stored procedure names carry schema prefix. No `IDapperContext` changes needed.

```csharp
// npa-api repository
_context.CreateConnection().ExecuteAsync("[npa].[sp_SelectLoginDetails]", params);

// adf-api repository
_context.CreateConnection().ExecuteAsync("[adf].[sp_SelectLoginDetails]", params);
```

**Topology 3 — Secondary daily DB:** Existing `CreateConnection(isDailyDBCn: true)` pattern — already implemented, carried forward unchanged.

#### D.5 JwtService — Claim Name Parameterization

Current `JwtService.cs` has hardcoded claim name strings: `"Username"`, `"UserLoginId"`, `"UserType"`, `"Timekey"`, `"Menus"`, `"IsChecker"`, `"RoleAltKey"`. ADF may use different claim schemas.

**Add `JwtClaimsMap` to `Common.Core/Helpers/AppSettings.cs`:**

```csharp
// Common.Core/Helpers/JwtClaimsMap.cs
public class JwtClaimsMap
{
    public string Username    { get; set; } = "Username";
    public string UserLoginId { get; set; } = "UserLoginId";
    public string UserType    { get; set; } = "UserType";
    public string TimeKey     { get; set; } = "Timekey";
    public string Menus       { get; set; } = "Menus";
    public string IsChecker   { get; set; } = "IsChecker";
    public string RoleAltKey  { get; set; } = "RoleAltKey";
}
```

Add to `AppSettings.cs`:
```csharp
public class AppSettings
{
    // ... existing fields ...
    public JwtClaimsMap ClaimsMap { get; set; } = new();
}
```

`JwtService` reads `_appSettings.ClaimsMap.Menus` instead of `"Menus"` etc. `CurrentUserEnrichmentMiddleware` and `AuthorizationService` do the same.

**npa-api impact:** Zero. Default values match the existing hardcoded strings exactly. No `appsettings.json` change needed. ADF overrides only the keys that differ.

Also fix the hardcoded `AddMinutes(30)` in `GenerateRefreshToken()`: move to `JWTSettings.RefreshTokenTTLMinutes` (default: `30`).

#### D.6 Program.cs Pattern for Each Area-API

**`AREAS/npa-api/Program.cs` (after migration — same runtime behavior):**

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Host.UseSerilog(...);                            // unchanged

var services = builder.Services;
services.AddCommonAppSettings(builder.Configuration);    // ← Common.Core
services.AddCommonJwtAuthentication(builder.Configuration); // ← Common.Core
services.AddCommonMiddleware();                          // ← Common.Core
services.AddCommonModelServices();                       // ← Common.Core
services.AddDapperContext();                             // ← Common.Core
services.AddNpaRepositoryServices();                     // ← stays in npa-api
services.AddNpaModelServices();                          // ← stays in npa-api
services.AddControllers(opts => opts.Filters.Add<ActionLoggingFilter>());
// ... swagger, cors, formOptions — unchanged ...

var app = builder.Build();
app.UseMiddleware<CorrelationHeadersMiddleware>();
app.UseMiddleware<RequestResponseLoggingMiddleware>();
app.UseMiddleware<CurrentUserEnrichmentMiddleware>();
app.UseAuthentication();
app.UseAuthorization();
app.UseMiddleware<ExceptionHandlerMiddleware>();
app.MapControllers();
await app.RunAsync();
```

**`AREAS/adf-api/Program.cs` (new area — identical structure):**

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Host.UseSerilog(...);

var services = builder.Services;
services.AddCommonAppSettings(builder.Configuration);    // ← Common.Core
services.AddCommonJwtAuthentication(builder.Configuration);
services.AddCommonMiddleware();
services.AddCommonModelServices();
services.AddDapperContext();
services.AddAdfRepositoryServices();                     // ← stays in adf-api
services.AddAdfModelServices();                          // ← stays in adf-api
services.AddControllers(opts => opts.Filters.Add<ActionLoggingFilter>());

var app = builder.Build();
// Identical middleware pipeline
await app.RunAsync();
```

#### D.7 .NET Solution Structure

```
backend/CrismacAxis.sln
  ├── src/Common.Core/Crismac.Axis.Common.Core.csproj
  ├── src/AREAS/npa-api/Crismac.Axis.Npa.Api.csproj   → ProjectRef: Common.Core
  └── src/AREAS/adf-api/Crismac.Axis.Adf.Api.csproj   → ProjectRef: Common.Core
```

Build all: `dotnet build backend/CrismacAxis.sln` — builds in dependency order automatically.

---

### 14.E AREAS Pattern — Contract Every Area-API Must Fulfill

An area-api is any `Microsoft.NET.Sdk.Web` project under `backend/src/AREAS/`. To be a valid area-api:

| Requirement | Detail |
|-------------|--------|
| Reference `Common.Core` | Via `<ProjectReference>` in `.csproj` |
| Call `AddCommonAppSettings()` | Registers all shared config sections |
| Call `AddCommonJwtAuthentication()` | Sets up JWT Bearer middleware |
| Call `AddCommonMiddleware()` + `AddCommonModelServices()` | Registers shared services |
| Call `AddDapperContext()` | Registers `IDapperContext` |
| Provide `AppSettings:AreaName` in appsettings | Identifies area in logs + Swagger title |
| Implement `IUserRepository` | Area-specific user table schema |
| Implement `IRefreshTokenRepository` | Area-specific refresh token table |
| Expose `LoginController` at `api/Login/{AreaPrefix}/...` | URL prefix discriminates areas for frontend |
| Own all domain repositories, services, controllers | No cross-area code sharing at runtime |

**How `AreaName` is used:**

```csharp
// Swagger title — in Program.cs
swagger.SwaggerDoc("v1", new OpenApiInfo {
    Title = $"CRisMac AXIS {appSettings.AreaName} API"
});

// Serilog enrichment — in Program.cs
.Enrich.WithProperty("AreaName", appSettings.AreaName)

// ActionLoggingFilter — already reads AreaName for structured log context
```

**Login endpoint URL prefix** — the `LoginController` route attribute in each area:
```csharp
// npa-api LoginController.cs
[HttpPost("CrisMAc/SelectLoginDetails")]

// adf-api LoginController.cs
[HttpPost("ADF/SelectLoginDetails")]
```

The Angular `auth.service.ts` in each frontend app targets its own prefix. These never conflict because area-apis run on different ports/domains.

---

### 14.F Migration Steps — Non-Breaking, Ordered

#### Phase 1: Reorganise Backend (Zero Code Changes)

**Step 1** — Create .NET solution file:
```bash
cd backend
dotnet new sln -n CrismacAxis
dotnet sln add src/CrismacAxisNpaApi/Crismac.Axis.Npa.Api.csproj
dotnet build CrismacAxis.sln   # must pass before continuing
```

**Step 2** — Move npa-api to AREAS folder (folder rename only):
```bash
mkdir -p src/AREAS/npa-api
# Move all files from src/CrismacAxisNpaApi/ → src/AREAS/npa-api/
dotnet sln remove src/CrismacAxisNpaApi/Crismac.Axis.Npa.Api.csproj
dotnet sln add src/AREAS/npa-api/Crismac.Axis.Npa.Api.csproj
dotnet build CrismacAxis.sln   # must still pass
```
Update any `deploy/` scripts referencing the old path.

**Step 3** — Scaffold `Common.Core` project:
```bash
cd src/Common.Core
dotnet new classlib -n Crismac.Axis.Common.Core --framework net9.0
dotnet sln ../../CrismacAxis.sln add Crismac.Axis.Common.Core.csproj
```
Add `<ProjectReference>` to Common.Core in `npa-api.csproj`. Build passes (Common.Core is empty).

#### Phase 2: Extract to Common.Core — Incrementally (Safest Order)

Extract in this exact order to minimise cascading compile errors. After each step: **build must pass**.

**Step 4** — Pure utilities first (no project dependencies):
Move `Constants/HttpStatusCodes.cs`, `Utilities/OtpGenerator.cs`, `Utilities/LoggingUtil.cs`, `Utilities/UrlUtils.cs`, `Utilities/fileUtil.cs`, `Utilities/MenuGuardConstants.cs`. Update namespaces. Add `using Crismac.Axis.Common.Core.*` in npa-api files that reference them.

**Step 5** — Helpers (POCO settings classes):
Move entire `Helpers/AppSettings.cs` + `Helpers/CorsSettings.cs` + `Helpers/AppException.cs` to `Common.Core/Helpers/`. Create `JwtClaimsMap.cs`, add `ClaimsMap` property to `AppSettings`. Update namespaces and using directives.

**Step 6** — Crypto:
Move `Services/Crypto/ICryptoService.cs` + `CryptoAes256CbcService.cs`.

**Step 7** — Password, Excel, Notification, File services:
Move `PasswordHasher/`, `ExcelService/`, `NotificationService.cs`, `FileLogService.cs`, `ExceptionErrorFileLogService.cs`, `FileValidationService.cs`. One at a time — build after each.

**Step 8** — Data layer:
Move `Data/IDapperContext.cs` + `Data/DapperContext.cs`. Add `CreateNamedConnection` overload.

**Step 9** — Repository interfaces:
Create `Common.Core/Repositories/IUserRepository.cs` + `IRefreshTokenRepository.cs`. Update `CurrentUserEnrichmentMiddleware` (next step) to depend on `IUserRepository` interface. In npa-api, register `UserService` (or a thin adapter) as `IUserRepository`.

**Step 10** — Middlewares:
Move all six middleware files. `CurrentUserEnrichmentMiddleware` now takes `IUserRepository` (from Common.Core) instead of the concrete `UserService` — update its constructor.

**Step 11** — JwtService parameterization + move:
Update `JwtService.cs` to read claim names from `_appSettings.ClaimsMap.*`. Fix hardcoded `AddMinutes(30)` — use `_jwtSettings.RefreshTokenTTLMinutes`. Move to Common.Core. Move `IRefreshTokenRepository` interface to Common.Core; keep `NpaRefreshTokenRepository` in npa-api.

**Step 12** — Generic Entities:
Move `ApiResponseDto.cs`, `RefreshToken.cs`, `TokenApiModel.cs`, `RequestLogEntry.cs`, `AppLogDto.cs` to `Common.Core/Entities/`.

**Step 13** — Filters:
Move `ActionLoggingFilter.cs`, `MenuGuardAttribute.cs`, `RateLimitAttribute.cs`, `RequestLimitAttribute.cs`.

**Step 14** — Extension methods:
Create `Common.Core/Extensions/ServiceCollectionExtensions.cs` with `AddCommonAppSettings()`, `AddCommonModelServices()`, `AddDapperContext()`, `AddCommonJwtAuthentication()`, `AddCommonMiddleware()`. Update npa-api's `ServiceCollectionExtensions.cs` to call these + keep only `AddNpaRepositoryServices()` and `AddNpaModelServices()`. Update `Program.cs` to use the new method names.

#### Phase 3: Regression Validation

**Step 15** — Full npa-api regression:
- `dotnet build CrismacAxis.sln` — zero warnings in Common.Core, zero new errors in npa-api
- Start npa-api: confirm Swagger UI loads, `AreaName` shows in title
- Test: login, OTP flow, token refresh, logout
- Test: file upload (end-to-end), authorize, reject, download
- Test: report access, app logs
- Verify Serilog structured output — no new `[WRN]` or `[ERR]` entries

#### Phase 4: Add adf-api

**Step 16** — Scaffold:
```bash
dotnet new webapi -n Crismac.Axis.Adf.Api --framework net9.0 \
  -o backend/src/AREAS/adf-api
dotnet sln backend/CrismacAxis.sln add src/AREAS/adf-api/Crismac.Axis.Adf.Api.csproj
```
Add `<ProjectReference>` to Common.Core.

**Step 17** — Wire adf-api following the contract (Section 14.E):
- `appsettings.json` with ADF DB connection, `AreaName: "ADF"`
- `Repositories/` for ADF stored procedures (implement `IUserRepository`, `IRefreshTokenRepository`)
- `Services/UserService.cs` — ADF auth + AD integration
- `Controllers/LoginController.cs` — route prefix `ADF`
- `Program.cs` — calls Common.Core extension methods + `AddAdfRepositoryServices()` + `AddAdfModelServices()`

**Step 18** — Verify adf-api builds and starts independently:
```bash
dotnet run --project src/AREAS/adf-api/Crismac.Axis.Adf.Api.csproj
# GET / → { "message": "CRisMac AXIS ADF API is hosted successfully!" }
```

#### Phase 5: Add adf-app Frontend + Root Tooling

**Step 19** — Follow frontend onboarding checklist (Section 14.G.1).

**Step 20** — Update root-level Husky hooks if pre-commit lint paths need updating to cover `AREAS/npa-api/` and `AREAS/adf-api/`.

---

### 14.G New App Onboarding Checklists

These checklists are the repeatable pattern for any future app (`xyz-app` / `xyz-api`).

#### G.1 Frontend: Adding a New Angular App

| Step | Action |
|------|--------|
| 1 | `ng generate application xyz-app --style scss --routing false --standalone --prefix app` |
| 2 | In `angular.json` → xyz-app serve target: set `"port": 42XX` (pick unused) |
| 3 | In `angular.json` → xyz-app build target: add `shared-public` assets + layout CSS style |
| 4 | Create `projects/xyz-app/src/environments/environment*.ts` — set `host`, `ssoEnabled`, encryption flags |
| 5 | Create `app.config.ts` — set PrimeNG theme color, provide `API_BASE_URL`, `ENCRYPTION_KEY`, `APP_NAME`, `ACTIVITY_LOG_PATH`, `LOGOUT_HANDLER` |
| 6 | Create `app.routes.ts` — xyz-app-specific routes + MenuIds |
| 7 | Copy `core/services/auth.service.ts` from npa-app — change login endpoint prefix to xyz area name |
| 8 | Copy `core/services/core.service.ts` from npa-app — no changes needed |
| 9 | Copy `core/menu/` from npa-app — no changes needed |
| 10 | Copy `layout/` from npa-app — update topbar template: app name + logo |
| 11 | Copy `pages/auth/login/` from npa-app — update logo `<img>`, heading text, SCSS color variable |
| 12 | Add `start:xyz-dev`, `serve-xyz-app:dev`, `build:xyz-app` scripts to `frontend/package.json` |
| 13 | Add `build:xyz-app` to `build:all` script |
| 14 | Verify: `pnpm run build:shared-lib && pnpm run build:xyz-app` → zero type errors |
| 15 | Verify: `pnpm run start:xyz-dev` → serves at `http://localhost:42XX`, branding correct |

#### G.2 Backend: Adding a New Area API

| Step | Action |
|------|--------|
| 1 | `dotnet new webapi -n Crismac.Axis.Xyz.Api --framework net9.0 -o backend/src/AREAS/xyz-api` |
| 2 | `dotnet sln backend/CrismacAxis.sln add src/AREAS/xyz-api/Crismac.Axis.Xyz.Api.csproj` |
| 3 | Add `<ProjectReference>` to `Common.Core` in xyz-api's `.csproj` |
| 4 | Create `appsettings.json`: `ConnectionStrings`, `JWTSettings`, `AppSettings.AreaName: "XYZ"`, `CorsSettings` (allow xyz-app origin) |
| 5 | Create `Repositories/LoginRepository/` + `UserRepository/` + `RefreshTokenRepository/` — implement `ILoginRepository`, `IUserRepository`, `IRefreshTokenRepository` against XYZ DB schema |
| 6 | Create `Entities/User.cs` — XYZ-specific user models |
| 7 | Create `Services/UserService.cs` — XYZ auth + AD lookup logic |
| 8 | Create `Controllers/LoginController.cs` — copy from npa-api, change route prefix to `XYZ` |
| 9 | Create `Extensions/ServiceCollectionExtensions.cs` → `AddXyzRepositoryServices()`, `AddXyzModelServices()` |
| 10 | Create `Program.cs` — call `AddCommonAppSettings()`, `AddCommonJwtAuthentication()`, `AddCommonMiddleware()`, `AddCommonModelServices()`, `AddDapperContext()`, `AddXyzModelServices()`, `AddXyzRepositoryServices()` |
| 11 | `dotnet build backend/CrismacAxis.sln` — must pass |
| 12 | Start xyz-api: Swagger UI loads, title shows "CRisMac AXIS XYZ API" |
| 13 | End-to-end: login from xyz-app → JWT issued → protected endpoint returns 200 |

---

### 14.H Key Design Decisions & Rationale

| Decision | Rationale |
|----------|-----------|
| `UserService` stays in area-api, not Common.Core | AD lookup, `CheckerRoles` classification, and `ILoginRepository` dependency are domain-specific per area. Forcing them into Common.Core would require either abstract classes (tight coupling) or too many area-specific interfaces leaking into the shared layer. |
| `AuthorizationService` stays in npa-api | Maker-Checker operation flags (1, 16, 17) and `NP`/`A`/`R` status codes are NPA domain concepts. ADF defines its own authorization logic. |
| Login endpoint prefix hardcoded in controller, not from config | It is a one-line route attribute in one file per area. Making it configurable adds indirection with no benefit — route attributes are compile-time contracts, not runtime config. |
| No `pnpm-workspace.yaml` — shared-lib consumed from `dist/` | Existing Angular library pattern. Library must be built first (`build:shared-lib`) before the app can compile. This is standard ng-packagr behavior. |
| Database isolation via config, not code | Each area-api reads its own `appsettings.json`. A misconfigured connection string is the only failure mode. Acceptable for the team size and governance model of this project. |
| No Nx/Turborepo | Build orchestration using `concurrently` + `wait-on` already handles topological ordering for 2–4 apps. Adopting Nx is a separate, orthogonal future decision. |

---

### 14.I Branch Management & PR Process

#### I.1 Branching Model — Environment-Based

There is **no `main` branch**. Every branch maps directly to a deployment environment. The flow is strictly linear per app:

```
develop
   │  ← all feature work lands here first
   │
   ▼  (promote when ready for staging QA)
release-npa   /   release-adf        ← staging environments (per app)
   │
   ▼  (promote after staging QA sign-off)
uat-npa       /   uat-adf            ← UAT environments (per app)
   │
   ▼  (promote after UAT business sign-off)
prod-npa      /   prod-adf           ← production environments (per app)
```

**Permanent branches — never deleted, never committed to directly:**

| Branch | Environment | Purpose |
|--------|-------------|---------|
| `develop` | Development | Integration — all feature/fix PRs target here |
| `release-npa` | Staging — NPA | QA testing for NPA before UAT |
| `release-adf` | Staging — ADF | QA testing for ADF before UAT |
| `uat-npa` | UAT — NPA | Business acceptance testing for NPA |
| `uat-adf` | UAT — ADF | Business acceptance testing for ADF |
| `prod-npa` | Production — NPA | Live NPA environment |
| `prod-adf` | Production — ADF | Live ADF environment |

**Short-lived working branches** (deleted after PR merges):

```
feature/<scope>/<description>     ← new functionality
fix/<scope>/<description>         ← bug fix (found in staging or UAT)
hotfix/<scope>/<description>      ← production emergency
```

**Short-lived branch naming convention** (enforced by pre-push hook — see Section 14.L):

| Type | When | Branch From |
|------|------|-------------|
| `feature` | New functionality | `develop` |
| `fix` | Bug found in staging or UAT | The environment branch where bug was found |
| `hotfix` | Production emergency | `prod-npa` or `prod-adf` |

Valid scopes:

| Scope | Covers | Rebuild impact |
|-------|--------|----------------|
| `npa` | `npa-app` + `npa-api` | NPA only |
| `adf` | `adf-app` + `adf-api` | ADF only |
| `common` | `shared-lib` + `Common.Core` | **ALL apps** |
| `root` | Root config, Husky, CI | **ALL apps** |
| `ci` | CI/CD workflow files | Infrastructure |
| `docs` | `doc/` only | None |
| `deps` | Dependency upgrades | Varies |
| `scripts` | `scripts/` folder | Dev tooling |

#### I.2 Normal Development Flow (Feature)

All new work — regardless of which app — goes through `develop` first. Promotions between environment branches are direct merges, not PRs from working branches.

```
1. Branch from develop:
   git checkout develop && git checkout -b feature/npa/upload-progress

2. Develop + commit locally
   Commits follow: feat(npa): add upload progress bar

3. PR → develop  (1 reviewer for npa/adf; 2 for common/root)
   CI: build + lint for affected scope only

4. Promote to staging when sprint is ready:
   git checkout release-npa && git merge --no-ff develop
   git push origin release-npa
   (Deploy release-npa to staging server → QA begins)

5. After staging QA sign-off → promote to UAT:
   git checkout uat-npa && git merge --no-ff release-npa
   git push origin uat-npa
   (Deploy uat-npa → business UAT begins)

6. After UAT sign-off → promote to production:
   git checkout prod-npa && git merge --no-ff uat-npa
   git push origin prod-npa
   (Deploy prod-npa → go live)

7. Tag the production state:
   git tag npa/v1.2.0 -m "NPA v1.2.0 — upload progress"
   git push origin npa/v1.2.0
```

**Common-scope features** follow the same flow but must promote to ALL staging branches when `develop` is ready, since shared code impacts both apps:
```
develop → release-npa  AND  develop → release-adf   (both promoted together)
release-npa → uat-npa  AND  release-adf → uat-adf
uat-npa → prod-npa     AND  uat-adf → prod-adf
```

#### I.3 Bugfix Scenarios

**Key rule:** A bugfix branches from the environment where the bug was found. Back-merge always flows downward toward `develop` — never upward toward prod.

---

**Scenario A — Bug found in Staging (`release-npa`)**

```
Bug discovered during QA on release-npa / staging-npa environment.

1. Branch from the staging branch (NOT from develop):
   git checkout release-npa
   git checkout -b fix/npa/upload-validation-error

2. Fix the bug, commit:
   fix(npa): handle empty row in Excel upload validation

3. PR → release-npa  (1 reviewer)
   CI: npa-app + npa-api build

4. MANDATORY back-merge to develop (fixes must not be lost):
   git checkout develop
   git merge --no-ff release-npa -m "chore(npa): back-merge fix from release-npa"
   git push origin develop

5. Continue normal promotion:
   release-npa → uat-npa → prod-npa
   (The fix travels up with the rest of the staging content)

6. Delete working branch: git branch -d fix/npa/upload-validation-error
```

**Important:** `develop` must receive the fix via back-merge before the next feature cycle starts. Otherwise the next promotion from `develop` → `release-npa` will overwrite the fix.

---

**Scenario B — Bug found in UAT (`uat-npa`)**

```
Bug discovered during UAT business testing on uat-npa.

1. Branch from the UAT branch:
   git checkout uat-npa
   git checkout -b fix/npa/grid-pagination-wrong-count

2. Fix + commit:
   fix(npa): correct grid row count calculation in paginated response

3. PR → uat-npa  (1 reviewer)
   CI: npa-app + npa-api build

4. MANDATORY back-merge cascade (all the way to develop):
   git checkout release-npa && git merge --no-ff uat-npa && git push origin release-npa
   git checkout develop    && git merge --no-ff release-npa && git push origin develop

5. UAT re-validation of the fixed behaviour → promote to prod:
   uat-npa → prod-npa (merge + deploy)

6. Tag: git tag npa/v1.2.1 → git push origin npa/v1.2.1
```

**Why cascade to develop?** If only `uat-npa` gets the fix, the next `develop` → `release-npa` promotion will lose it. The invariant is: **develop always contains everything that is in any environment branch.**

#### I.4 Hotfix Scenario (Production Emergency)

A hotfix is reserved for critical production issues that cannot wait for the normal promotion cycle.

**Scope: single app (e.g., NPA production down)**

```
Critical bug found live on prod-npa.

1. Branch from prod-npa (the source of truth for what is live):
   git checkout prod-npa
   git checkout -b hotfix/npa/jwt-token-not-refreshing

2. Minimal, targeted fix only — no refactoring, no unrelated changes:
   fix(npa): correct refresh token expiry timestamp comparison

3. PR → prod-npa  (1 reviewer minimum — expedited review)
   CI: smoke test only (fastest possible validation)

4. Merge + immediate production deploy:
   Notify team → deploy → verify prod is stable

5. MANDATORY back-merge cascade — fix must reach all lower environments:
   git checkout uat-npa    && git merge --no-ff prod-npa  && git push origin uat-npa
   git checkout release-npa && git merge --no-ff uat-npa && git push origin release-npa
   git checkout develop     && git merge --no-ff release-npa && git push origin develop

6. Tag the hotfix:
   git tag npa/v1.1.1 -m "hotfix: jwt refresh token fix"
   git push origin npa/v1.1.1

7. Delete working branch: git branch -d hotfix/npa/jwt-token-not-refreshing
```

**Scope: common code (shared-lib or Common.Core — both apps affected)**

When a bug in `Common.Core` or `shared-lib` causes a production issue in both apps, the fastest safe path is to fix through `develop` and fast-track through all environments for both apps simultaneously:

```
1. Branch from develop (common fixes always go through develop):
   git checkout develop
   git checkout -b hotfix/common/crypto-decryption-failure

2. Fix Common.Core or shared-lib:
   fix(common-core): handle null IV in AES decryption

3. PR → develop  (2 reviewers mandatory — common scope)
   CI: ALL apps build + ALL area-apis compile

4. Fast-track promote both apps through all environments together:
   develop → release-npa + release-adf  (deploy both to staging — quick smoke test)
   release-npa + release-adf → uat-npa + uat-adf  (expedited UAT sign-off)
   uat-npa + uat-adf → prod-npa + prod-adf  (coordinated dual deploy)

5. Tag both apps:
   git tag npa/v1.1.1 && git tag adf/v1.0.1
   git push origin npa/v1.1.1 adf/v1.0.1
```

#### I.5 PR Rules Summary

| Scenario | Branch From | PR Targets | Reviewers | CI Required | Back-merge |
|---|---|---|---|---|---|
| Feature (npa/adf) | `develop` | `develop` | 1 | Affected app only | — |
| Feature (common/root) | `develop` | `develop` | **2** | ALL apps | — |
| Bugfix in staging | `release-<app>` | `release-<app>` | 1 | Affected app | `release` → `develop` |
| Bugfix in UAT | `uat-<app>` | `uat-<app>` | 1 | Affected app | `uat` → `release` → `develop` |
| Hotfix (prod, app-specific) | `prod-<app>` | `prod-<app>` | 1 | Smoke test | `prod` → `uat` → `release` → `develop` |
| Hotfix (prod, common) | `develop` | `develop` | **2** | ALL apps | Fast-track promote both apps |

#### I.6 PR Checklist for `common` Scope

Any PR touching `shared-lib` or `Common.Core` must verify:

- [ ] `shared-lib/src/public-api.ts` backward compatible? (no removed exports without major version bump)
- [ ] `AddCommonModelServices()` still registers the same services?
- [ ] Both `npa-app` and `adf-app` build: `pnpm run build:all`
- [ ] Both `npa-api` and `adf-api` compile: `dotnet build CrismacAxis.sln`
- [ ] No app-specific hardcoded strings in shared code (`CrisMAc`, `NPA_`, `ADF_` literals)
- [ ] `JwtClaimsMap` defaults unchanged (npa-api needs no `appsettings.json` change)
- [ ] Doc updated in `doc/` if area-apis need action (e.g., add a new appsettings key)

#### I.7 Version Tagging

Tags are per-app so each app can release independently:

```bash
# Tag after each production deploy
git tag npa/v1.2.0 -m "NPA v1.2.0 — chunked upload redesign"
git tag adf/v1.0.0 -m "ADF v1.0.0 — initial release"
git push origin --tags
```

This lets NPA ship a hotfix (`npa/v1.1.1`) while ADF is still on `adf/v1.0.0` — no coupling.

---

### 14.J Build Strategy & Isolation

#### J.1 Dependency Graph — Rebuild Rules

```
Common.Core ──────────────► npa-api  (ProjectReference)
            └─────────────► adf-api  (ProjectReference)

shared-lib (dist/) ────────► npa-app  (tsconfig path alias)
                   └────────► adf-app  (tsconfig path alias)
```

**Only rebuild what is downstream of the changed layer:**

| What changed | Minimum rebuild required |
|---|---|
| `Common.Core` only | Common.Core + **ALL** area-apis |
| `shared-lib` only | shared-lib + **ALL** Angular apps |
| `npa-api` only | npa-api only |
| `adf-api` only | adf-api only |
| `npa-app` only | npa-app only (shared-lib already built) |
| `adf-app` only | adf-app only (shared-lib already built) |
| `Common.Core` + `npa-app` | Common.Core → all area-apis + shared-lib → npa-app |
| `shared-lib` + `Common.Core` | shared-lib → all apps + Common.Core → all area-apis |

#### J.2 Build Artifacts — Never Cross-Pollinate

Each deployable produces an isolated output directory. No app reads another app's output:

```
Frontend:
  dist/shared-lib/   ← consumed only via tsconfig path alias at build time
  dist/npa-app/      ← deployed to NPA environment only
  dist/adf-app/      ← deployed to ADF environment only

Backend:
  publish/npa-api/   ← dotnet publish output for NPA server
  publish/adf-api/   ← dotnet publish output for ADF server
```

`Common.Core` has **no publish output** — it is compiled into each area-api's output at `dotnet publish` time via `<ProjectReference>`. Deploying `npa-api` includes Common.Core's DLLs automatically.

#### J.3 CI/CD Scope-Gated Build Matrix

The CI pipeline detects changed file paths and runs only the relevant targets. This keeps CI fast for single-app PRs while running full validation for common changes:

| Files changed in PR | Frontend CI | Backend CI |
|---|---|---|
| `frontend/projects/shared-lib/**` | `pnpm run build:all` | — |
| `backend/src/Common.Core/**` | — | `dotnet build CrismacAxis.sln` |
| `frontend/projects/npa-app/**` | `pnpm run build:npa-app` | — |
| `frontend/projects/adf-app/**` | `pnpm run build:adf-app` | — |
| `backend/src/AREAS/npa-api/**` | — | `dotnet publish AREAS/npa-api` |
| `backend/src/AREAS/adf-api/**` | — | `dotnet publish AREAS/adf-api` |
| `frontend/package.json` | `pnpm run build:all` | — |
| Root `package.json` or `.husky/` | Full monorepo build | Full solution build |

#### J.4 Build Configuration Reference

| Scenario | Frontend command | Backend command |
|---|---|---|
| Local dev — NPA | `pnpm run start:dev` | `dotnet run --project AREAS/npa-api` |
| Local dev — ADF | `pnpm run start:adf-dev` | `dotnet run --project AREAS/adf-api` |
| Build NPA only | `pnpm run build:npa` | `dotnet publish AREAS/npa-api -c Release -o publish/npa-api` |
| Build ADF only | `pnpm run build:adf` | `dotnet publish AREAS/adf-api -c Release -o publish/adf-api` |
| Build shared only | `pnpm run build:shared-lib` | `dotnet build Common.Core` |
| Build everything | `pnpm run build:all` | `dotnet build CrismacAxis.sln -c Release` |

#### J.5 Parallel App Builds

When all apps are in a stable state (no shared-lib changes pending), app builds can run in parallel since they only share the pre-built `dist/shared-lib/` artifact:

```bash
# Step 1 — sequential (required dependency)
pnpm run build:shared-lib

# Step 2 — parallel (independent outputs)
pnpm run build:npa-app & pnpm run build:adf-app & wait
echo "All apps built"
```

For backend, `dotnet build CrismacAxis.sln` already parallelises by default (MSBuild determines safe parallel order from `<ProjectReference>` graph).

---

### 14.K Package & Dependency Management

#### K.1 Frontend — One `package.json`, All Apps Share It

The Angular multi-project workspace has a **single** `frontend/package.json`. There is no per-app `package.json`. Every Angular app in the workspace uses the same version of every dependency.

**Core implication:** You cannot have `npa-app` on `primeng@18` and `adf-app` on `primeng@19` simultaneously. The workspace moves as a unit on major framework versions. New apps joining must accept the current locked versions.

**Version conflict prevention — rules for the team:**

**Rule 1 — Lock major versions per Angular release cycle.**  
All `@angular/*` packages stay on the same major. When Angular 19 is adopted, all apps in the workspace upgrade together in one coordinated PR (`scope: deps`).

**Rule 2 — Use caret (`^`) ranges, not tilde (`~`).**  
Caret allows minor/patch upgrades; the `pnpm-lock.yaml` is the real version pin. Always commit lockfile changes alongside `package.json` changes.

**Rule 3 — Pin the pnpm version in Volta.**  
Currently only Node is pinned. Add pnpm too:
```json
// frontend/package.json
"volta": { "node": "20.19.5", "pnpm": "9.15.0" }
```

**Rule 4 — Strict peer dependency mode in `.npmrc`:**
```
strict-peer-dependencies=true
auto-install-peers=false
shamefully-hoist=false
```
This surfaces version conflicts at `pnpm install` time rather than silently resolving them wrong. The CI `pnpm install --frozen-lockfile` flag then catches any lockfile drift.

**Rule 5 — Version upgrade process:**
1. One developer upgrades in `package.json`, runs `pnpm install`
2. Verifies all apps build: `pnpm run build:all`
3. Runs all tests: `pnpm run test`
4. Raises PR with `scope: deps` — reviewed by maintainer of every affected app
5. Lockfile diff is reviewed carefully in the PR (no surprise transitive upgrades)

**Escape hatch — `pnpm overrides` (last resort only):**  
If two apps genuinely need incompatible versions of the same package (rare), use pnpm overrides with a mandatory expiry comment:
```json
// frontend/package.json
"pnpm": {
  "overrides": {
    "some-lib": "2.x"
  },
  "// NOTE": "adf-app requires 2.x; resolve by 2026-Q3 when npa-app upgrades — owner: @developer"
}
```
This must be treated as technical debt. Set a calendar reminder, assign an owner, and resolve it at the next major Angular upgrade cycle.

#### K.2 Backend — Per-Project NuGet (More Flexible)

Unlike the frontend, each `.csproj` has its own `<PackageReference>`. Different area-apis **can** reference different versions of the same NuGet package — .NET resolves to the highest referenced version at build time through MSBuild's standard dependency resolution.

**Common.Core version strategy:**  
Common.Core specifies **minimum viable versions** — the lowest version that provides the feature the shared code uses. Area-apis inherit these versions via `<ProjectReference>` and can reference higher versions in their own `.csproj` without conflict.

```xml
<!-- Common.Core: minimum version -->
<PackageReference Include="Dapper" Version="2.1.66" />

<!-- adf-api: uses newer Dapper feature — no conflict -->
<PackageReference Include="Dapper" Version="2.2.0" />
```

**NuGet lockfile per project** (prevent silent upgrades on CI restore):
```xml
<!-- Add to each .csproj -->
<PropertyGroup>
  <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
</PropertyGroup>
```

Commit the generated `packages.lock.json` alongside the `.csproj`. CI must run with `dotnet restore --locked-mode` to catch drift.

**NuGet update process:**
```bash
# Check what is outdated across the solution
dotnet list backend/CrismacAxis.sln package --outdated

# Update a package in Common.Core
dotnet add backend/src/Common.Core package Dapper --version 2.2.0

# Update a package in a specific area-api only
dotnet add backend/src/AREAS/npa-api package EPPlus --version 8.3.0

# Rebuild to verify no conflicts
dotnet build backend/CrismacAxis.sln

# Restore in locked mode (used in CI)
dotnet restore backend/CrismacAxis.sln --locked-mode
```

#### K.3 Version Compatibility Matrix

Maintain this table in `doc/` — update it whenever a package version changes. It is the first reference point when a compatibility issue arises:

| Package | Common.Core | npa-api | adf-api | Frontend (all apps) | Notes |
|---------|-------------|---------|---------|---------------------|-------|
| Dapper | 2.1.66 | inherits | inherits | — | — |
| EPPlus | 8.2.1 | inherits | 8.3.0 | — | adf needs newer Excel feature |
| Serilog.AspNetCore | 5.0.0 | inherits | inherits | — | — |
| BCrypt.Net-Next | 4.0.3 | inherits | inherits | — | — |
| Angular | — | — | — | 18.2.14 | Move together on major upgrades |
| PrimeNG | — | — | — | 18.0.2 | Move together |
| Node.js | — | — | — | 20.19.5 (Volta) | — |
| pnpm | — | — | — | 9.15.0 (Volta) | — |
| .NET SDK | — | 9.0.111 | 9.0.111 | — | `global.json` at root |

*"inherits"* means the area-api uses Common.Core's version via `<ProjectReference>` — no explicit `<PackageReference>` in the area-api's `.csproj`.

---

### 14.L Automation Scripts & Developer Commands

#### L.1 Root-Level `scripts/` Folder

Create a `scripts/` folder at the monorepo root. All scripts are plain **Node.js** — no shell interpreter required. Run with `node scripts/<file>.js`. Node is already required (Volta-pinned), so there are zero extra tools to install.

```
crismac-npa-axis/
└── scripts/
    ├── build-npa.js          # Build npa-app + npa-api (full NPA stack)
    ├── build-adf.js          # Build adf-app + adf-api (full ADF stack)
    ├── build-common.js       # Rebuild shared-lib + Common.Core only
    ├── build-all.js          # Build entire monorepo
    ├── new-api.js            # Scaffold a new area API interactively
    ├── new-app.js            # Scaffold a new Angular app (guidance)
    └── validate-branch.js    # Validate branch naming convention (used by Husky)
```

**`scripts/build-npa.js`** — full NPA stack build:
```js
// scripts/build-npa.js
const { execSync } = require('child_process');
const path = require('path');

const ROOT = path.resolve(__dirname, '..');
const FE   = path.join(ROOT, 'frontend');
const BE   = path.join(ROOT, 'backend');

function run(cmd, cwd = ROOT) {
  console.log(`\n  > ${cmd}`);
  execSync(cmd, { stdio: 'inherit', cwd });
}

console.log('\n[1/3] Building shared-lib...');
run('pnpm run build:shared-lib', FE);

console.log('\n[2/3] Building npa-app...');
run('pnpm run build:npa-app', FE);

console.log('\n[3/3] Publishing npa-api...');
run(`dotnet publish src/AREAS/npa-api -c Release -o "${path.join(ROOT, 'publish', 'npa-api')}" --nologo -q`, BE);

console.log('\n✓ NPA build complete');
console.log('  Frontend → dist/npa-app/');
console.log('  Backend  → publish/npa-api/');
```

**`scripts/build-common.js`** — run when `shared-lib` or `Common.Core` changes:
```js
// scripts/build-common.js
const { execSync } = require('child_process');
const path = require('path');

const ROOT = path.resolve(__dirname, '..');

function run(cmd, cwd = ROOT) {
  console.log(`\n  > ${cmd}`);
  execSync(cmd, { stdio: 'inherit', cwd });
}

console.log('\n[1/2] Rebuilding shared-lib...');
run('pnpm run build:shared-lib', path.join(ROOT, 'frontend'));

console.log('\n[2/2] Rebuilding Common.Core + all area-apis...');
run('dotnet build CrismacAxis.sln -c Release --nologo -q', path.join(ROOT, 'backend'));

console.log('\n✓ Common rebuild complete — all area-apis recompiled against updated Common.Core');
```

**`scripts/new-api.js`** — scaffold a new area API:
```js
// scripts/new-api.js
const { execSync } = require('child_process');
const path = require('path');

const areaName = process.argv[2];
if (!areaName) {
  console.error('Usage: node scripts/new-api.js <AreaName>');
  console.error('Example: node scripts/new-api.js Xyz');
  process.exit(1);
}

const areaLower  = areaName.toLowerCase();
const ROOT       = path.resolve(__dirname, '..');
const apiDir     = path.join(ROOT, 'backend', 'src', 'AREAS', `${areaLower}-api`);
const projName   = `Crismac.Axis.${areaName}.Api`;
const projFile   = path.join(apiDir, `${projName}.csproj`);
const slnFile    = path.join(ROOT, 'backend', 'CrismacAxis.sln');
const coreFile   = path.join(ROOT, 'backend', 'src', 'Common.Core', 'Crismac.Axis.Common.Core.csproj');

function run(cmd, cwd = ROOT) {
  console.log(`\n  > ${cmd}`);
  execSync(cmd, { stdio: 'inherit', cwd });
}

console.log(`\n==> Scaffolding ${projName}...`);
run(`dotnet new webapi -n "${projName}" --framework net9.0 -o "${apiDir}" --no-restore`);

console.log('\n==> Adding to solution...');
run(`dotnet sln "${slnFile}" add "${projFile}"`);

console.log('\n==> Adding Common.Core project reference...');
run(`dotnet add "${projFile}" reference "${coreFile}"`);

console.log('\n==> Restoring packages...');
run(`dotnet restore "${slnFile}" -q`);

console.log(`\n✓ ${projName} created at backend/src/AREAS/${areaLower}-api/`);
console.log('\nNext steps (follow Section 14.G.2 checklist):');
console.log(`  1. Create appsettings.json → AppSettings:AreaName = '${areaName}'`);
console.log('  2. Implement IUserRepository and IRefreshTokenRepository');
console.log(`  3. Create Controllers/LoginController.cs — route prefix '${areaName}'`);
console.log(`  4. Wire Program.cs: AddCommonAppSettings() + Add${areaName}RepositoryServices()`);
console.log(`  5. Scaffold frontend: cd frontend && ng generate application ${areaLower}-app --style scss --standalone`);
```

**`scripts/validate-branch.js`** — validates branch naming convention (used by Husky `pre-push`):
```js
// scripts/validate-branch.js
const { execSync } = require('child_process');

const branch = execSync('git rev-parse --abbrev-ref HEAD').toString().trim();

// Permanent environment branches — always allowed
const PERMANENT = [
  'develop',
  'release-npa', 'release-adf',
  'uat-npa',     'uat-adf',
  'prod-npa',    'prod-adf',
];
if (PERMANENT.includes(branch)) process.exit(0);

// Short-lived working branch pattern: <type>/<scope>/<description>
const PATTERN = /^(feature|fix|hotfix)\/(npa|adf|common|root|ci|docs|deps|scripts)\/.+/;

if (!PATTERN.test(branch)) {
  console.error('');
  console.error(`  ERROR: Branch '${branch}' violates the naming convention.`);
  console.error('');
  console.error('  Required format:  <type>/<scope>/<description>');
  console.error('  Types:   feature | fix | hotfix');
  console.error('  Scopes:  npa | adf | common | root | ci | docs | deps | scripts');
  console.error('');
  console.error('  Examples:');
  console.error('    feature/npa/upload-progress-bar');
  console.error('    fix/common/cors-header-missing');
  console.error('    hotfix/npa/jwt-token-not-refreshing');
  console.error('');
  process.exit(1);
}

process.exit(0);
```

Add convenience script entries to the **root** `package.json`:
```json
{
  "scripts": {
    "build:npa":       "node scripts/build-npa.js",
    "build:adf":       "node scripts/build-adf.js",
    "build:common":    "node scripts/build-common.js",
    "build:all":       "node scripts/build-all.js",
    "new:api":         "node scripts/new-api.js",
    "new:app":         "node scripts/new-app.js"
  }
}
```

Developers then run `pnpm run build:npa` or `pnpm run new:api Xyz` from the repo root.

#### L.2 Frontend Package Scripts (Complete Reference)

The complete `scripts` block for `frontend/package.json`. This supersedes the current partial set:

```json
{
  "scripts": {
    "ng": "ng",

    "start:dev":        "concurrently --names \"LIB,NPA\" -c \"blue,green\" -k \"pnpm run watch-lib\" \"pnpm run serve-npa:dev\"",
    "start:adf-dev":    "concurrently --names \"LIB,ADF\" -c \"blue,cyan\"  -k \"pnpm run watch-lib\" \"pnpm run serve-adf:dev\"",

    "watch-lib":        "set NODE_OPTIONS=--max-old-space-size=4096 && pnpm run clean-lib && ng build shared-lib --watch --poll=2000 --configuration development",
    "serve-npa:dev":    "wait-on dist/shared-lib/fesm2022/shared-lib.mjs && ng serve npa-app --hmr --configuration development --port 4200",
    "serve-adf:dev":    "wait-on dist/shared-lib/fesm2022/shared-lib.mjs && ng serve adf-app --hmr --configuration development --port 4201",

    "build:shared-lib": "pnpm run clean-lib && ng build shared-lib",
    "build:npa-app":    "pnpm run clean-npa && ng build npa-app",
    "build:adf-app":    "pnpm run clean-adf && ng build adf-app",

    "build:npa":        "pnpm run build:shared-lib && pnpm run build:npa-app",
    "build:adf":        "pnpm run build:shared-lib && pnpm run build:adf-app",
    "build:all":        "pnpm run build:shared-lib && pnpm run build:npa-app && pnpm run build:adf-app",

    "clean-lib":        "rimraf dist/shared-lib",
    "clean-npa":        "rimraf dist/npa-app",
    "clean-adf":        "rimraf dist/adf-app",
    "clean":            "rimraf dist",

    "test":             "ng test",
    "lint":             "ng lint",
    "lint:fix":         "ng lint --fix",
    "format":           "prettier --write \"projects/**/*.{ts,html,css,scss}\"",
    "format:check":     "prettier --check \"projects/**/*.{ts,html,css,scss}\""
  }
}
```

**Pattern for each new app (`xyz-app`) — add to `frontend/package.json` scripts:**
```json
"start:xyz-dev":    "concurrently --names \"LIB,XYZ\" -k \"pnpm run watch-lib\" \"pnpm run serve-xyz:dev\"",
"serve-xyz:dev":    "wait-on dist/shared-lib/fesm2022/shared-lib.mjs && ng serve xyz-app --hmr --configuration development --port 42XX",
"build:xyz-app":    "pnpm run clean-xyz && ng build xyz-app",
"build:xyz":        "pnpm run build:shared-lib && pnpm run build:xyz-app",
"clean-xyz":        "rimraf dist/xyz-app"
```
Also append `&& pnpm run build:xyz-app` to `build:all`.

#### L.3 Backend Build Command Reference

```bash
# ── Development ───────────────────────────────────────────────────────
dotnet run   --project backend/src/AREAS/npa-api    # https://localhost:7294
dotnet run   --project backend/src/AREAS/adf-api    # https://localhost:7295
dotnet watch run --project backend/src/AREAS/npa-api  # Hot-reload dev

# ── Build ─────────────────────────────────────────────────────────────
dotnet build backend/CrismacAxis.sln                # Build all (dependency order auto-resolved)
dotnet build backend/src/AREAS/npa-api              # NPA only
dotnet build backend/src/Common.Core                # Common.Core only

# ── Publish ───────────────────────────────────────────────────────────
dotnet publish backend/src/AREAS/npa-api -c Release -o publish/npa-api --nologo
dotnet publish backend/src/AREAS/adf-api -c Release -o publish/adf-api --nologo

# ── Packages ──────────────────────────────────────────────────────────
dotnet list backend/CrismacAxis.sln package --outdated
dotnet add  backend/src/Common.Core package Dapper --version 2.2.0
dotnet restore backend/CrismacAxis.sln --locked-mode   # CI: fail on lockfile drift

# ── Code Quality ──────────────────────────────────────────────────────
dotnet format backend/CrismacAxis.sln --verify-no-changes  # CI check
dotnet format backend/CrismacAxis.sln                      # Fix formatting
```

#### L.4 Commitlint Scope Enforcement

Every commit must declare which part of the monorepo it touches. CI reads the scope from the commit message and activates only the relevant build targets — keeping pipelines fast for single-app PRs.

```js
// .commitlintrc.js (at repo root — update existing file)
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'scope-enum': [2, 'always', [
      'npa',          // npa-app or npa-api
      'adf',          // adf-app or adf-api
      'shared-lib',   // frontend shared library
      'common-core',  // backend Common.Core class library
      'root',         // root package.json, Husky, CI config
      'ci',           // CI/CD workflow files only
      'docs',         // documentation only
      'deps',         // dependency upgrades
      'scripts'       // scripts/ folder
    ]],
    'scope-empty': [2, 'never']  // scope is MANDATORY on every commit
  }
};
```

Valid commit message examples:
```
feat(npa): add chunked upload progress bar
fix(common-core): parameterize JWT claim names in JwtService
fix(npa): back-merge staging fix — upload validation empty row
hotfix(npa): correct refresh token expiry comparison
chore(deps): upgrade primeng to 18.1.0
docs(docs): add Section 14.I environment branch model
ci(ci): add scope-gated build matrix
```

#### L.5 Husky Hooks — Pre-Push Branch Validation

The `pre-push` hook calls the Node.js validator. The hook file itself must be a shell file (Git hook requirement), but it simply delegates to the JS script:

**`.husky/pre-push`:**
```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

node scripts/validate-branch.js
```

**Existing `.lintstagedrc.js`** — extend to cover backend C# if `dotnet-format` is installed globally:
```js
// .lintstagedrc.js
module.exports = {
  // Frontend — existing rules
  'frontend/projects/**/*.{ts,html}': ['ng lint --fix', 'prettier --write'],
  'frontend/projects/**/*.scss':      ['prettier --write'],

  // Backend — optional (enable after: dotnet tool install -g dotnet-format)
  // 'backend/**/*.cs': ['dotnet format --include'],
};
```

---

## Quick Reference (Updated for Multi-App)

**Frontend — npa-app:**

| Need | Go To |
|------|-------|
| Add a feature to npa-app | `frontend/projects/npa-app/src/app/pages/` |
| Add a shared UI component | `frontend/projects/shared-lib/src/lib/components/` |
| Export from shared-lib | `frontend/projects/shared-lib/src/public-api.ts` |
| Change npa-app theme color | `frontend/projects/npa-app/src/app/app.config.ts` → `providePrimeNG` |
| Add a new app to workspace | Follow Section 14.G.1 checklist |

**Backend — npa-api:**

| Need | Go To |
|------|-------|
| Add a new API endpoint (NPA) | `backend/src/AREAS/npa-api/Controllers/` |
| Add a new NPA repository | `backend/src/AREAS/npa-api/Repositories/` |
| Change upload limits | `backend/src/AREAS/npa-api/appsettings.json` → `UploadSettings` |
| Change JWT expiry | `backend/src/AREAS/npa-api/appsettings.json` → `JWTSettings` |
| Change CORS origins | `backend/src/AREAS/npa-api/appsettings.json` → `CorsSettings` |
| Enable OTP login | `backend/src/AREAS/npa-api/appsettings.json` → `LoginOtpEnabled: true` |
| Enable request encryption | `backend/src/AREAS/npa-api/appsettings.json` → `RequestEncryptionEnabled: true` |
| Add a shared backend service | `backend/src/Common.Core/Services/` → export via `AddCommonModelServices()` |
| Add a new area API | Follow Section 14.G.2 checklist |

---

---

## 15. Post-Refactor Repository Hierarchy

> **What this shows:** The complete folder and file layout of the repository after the Common.Core extraction, AREAS reorganisation, and full addition of both `npa-app`/`npa-api` and `adf-app`/`adf-api`. Every path is grounded in the actual current codebase — no invented names.

```
crismac-npa-axis/                                          ← Git root
│
│  ── Root tooling ───────────────────────────────────────
├── package.json                                           ← Meta-only. Husky, Commitlint, Volta pins
├── pnpm-lock.yaml                                         ← Lockfile (JS root deps only)
├── global.json                                            ← .NET SDK pin → 9.0.111
├── .commitlintrc.js                                       ← Scope enum: npa|adf|common|root|ci|docs|deps|scripts
├── .lintstagedrc.js                                       ← Pre-commit lint for frontend files
├── .npmrc                                                 ← strict-peer-dependencies=true
├── .prettierrc / .prettierignore
├── .editorconfig
├── .gitignore / .gitattributes
├── crismac-npa-axis.code-workspace                        ← VS Code multi-root workspace
├── .husky/
│   ├── _/husky.sh
│   ├── commit-msg                                         ← Runs commitlint
│   ├── pre-commit                                         ← Runs lint-staged
│   └── pre-push                                           ← node scripts/validate-branch.js
├── .vscode/
│   └── settings.json
│
│  ── Automation scripts ─────────────────────────────────
├── scripts/                                               ← Node.js scripts, no shell required
│   ├── build-npa.js                                       ← shared-lib + npa-app + npa-api publish
│   ├── build-adf.js                                       ← shared-lib + adf-app + adf-api publish
│   ├── build-common.js                                    ← shared-lib + dotnet build solution
│   ├── build-all.js                                       ← Full monorepo build
│   ├── new-api.js                                         ← Scaffold area API + wire solution + Common.Core ref
│   ├── new-app.js                                         ← Guided Angular app scaffold instructions
│   └── validate-branch.js                                 ← Validates branch naming convention
│
│  ── Frontend Angular workspace ─────────────────────────
├── frontend/
│   ├── angular.json                                       ← 3 projects: shared-lib, npa-app, adf-app
│   ├── package.json                                       ← Single dep file for ALL apps (pnpm, Volta)
│   ├── pnpm-lock.yaml                                     ← Frontend lockfile
│   ├── tsconfig.json                                      ← paths: { "shared-lib": ["./dist/shared-lib"] }
│   ├── tsconfig.app.json
│   ├── tsconfig.spec.json
│   ├── .eslintrc.json
│   ├── karma.conf.js
│   │
│   ├── shared-public/                                     ← Static assets shared by all apps
│   │   └── assets/
│   │       └── layout/
│   │           └── css/
│   │               └── layout-light.css
│   │
│   ├── dist/                                              ← Build outputs (gitignored)
│   │   ├── shared-lib/                                    ← ng-packagr output; consumed via tsconfig alias
│   │   ├── npa-app/                                       ← NPA deployable (standalone)
│   │   └── adf-app/                                       ← ADF deployable (standalone)
│   │
│   └── projects/
│       │
│       │  ── Shared Library ────────────────────────────
│       ├── shared-lib/
│       │   ├── ng-package.json
│       │   └── src/
│       │       ├── public-api.ts                          ← Single export surface for all consumers
│       │       └── lib/
│       │           ├── auth/
│       │           │   ├── auth.interceptor.ts            ← Attaches JWT Bearer token to every request
│       │           │   ├── auth-state.service.ts          ← Angular Signals auth state (login/logout/refresh)
│       │           │   ├── auth.token.ts                  ← InjectionTokens: API_BASE_URL, APP_NAME,
│       │           │   │                                     ACTIVITY_LOG_PATH, ENCRYPTION_KEY, etc.
│       │           │   └── provide-auth-interceptor.ts
│       │           ├── components/
│       │           │   ├── grid/
│       │           │   │   ├── grid.component.ts          ← Reusable PrimeNG data grid
│       │           │   │   └── model/
│       │           │   │       ├── grid-column-definition.ts
│       │           │   │       └── table-parameter.ts
│       │           │   ├── remark-modal/
│       │           │   │   └── remark-modal.component.ts  ← Authorize / Reject dialog
│       │           │   └── session-timer/
│       │           │       ├── session-timer.component.ts ← Idle countdown UI
│       │           │       └── session-timer.service.ts
│       │           ├── directives/
│       │           │   ├── lowerCaseInput.directive.ts
│       │           │   ├── restrict-decimal-input.directive.ts
│       │           │   ├── restrictInput.directive.ts
│       │           │   └── upperCaseText.directive.ts
│       │           ├── error-pages/
│       │           │   ├── app.accessdenied.component.ts
│       │           │   ├── app.notfound.component.ts
│       │           │   ├── comming-soon.component.ts
│       │           │   ├── session-expired.component.ts
│       │           │   └── error-pages.routes.ts          ← Catch-all route definitions
│       │           ├── guards/
│       │           │   ├── auth.guard.ts                  ← Redirects unauthenticated users → /login
│       │           │   └── menu.guard.ts                  ← MenuId-based feature access control
│       │           ├── helpers/
│       │           │   └── jwt.helper.ts
│       │           ├── interceptors/
│       │           │   ├── encryption.interceptor.ts      ← AES-256-CBC request body encryption
│       │           │   └── error.interceptor.ts           ← Global HTTP error → user-facing message
│       │           ├── models/
│       │           │   ├── icomplaint.ts
│       │           │   └── iuser.ts
│       │           ├── services/
│       │           │   ├── activity-log.service.ts        ← Page view + action logging (uses ACTIVITY_LOG_PATH)
│       │           │   ├── common.service.ts
│       │           │   ├── config.service.ts              ← Reads injection tokens (API_BASE_URL, etc.)
│       │           │   ├── crypto.service.ts              ← AES-256-CBC client-side encrypt/decrypt
│       │           │   ├── idle-detection.service.ts      ← User inactivity tracker → triggers auto-logout
│       │           │   ├── local-storage.service.ts       ← Encrypted localStorage wrapper
│       │           │   └── utils.service.ts               ← General utilities (largest: 19.7 KB)
│       │           └── utilities/
│       │               └── common-module.ts               ← COMMON_MODULE: shared Angular imports barrel
│       │
│       │  ── NPA Application ─────────────────────────────
│       ├── npa-app/
│       │   ├── tsconfig.app.json
│       │   └── src/
│       │       ├── index.html                             ← "CrisMAC NPA" title + favicon
│       │       ├── main.ts                                ← bootstrapApplication(AppComponent, appConfig)
│       │       ├── styles.scss
│       │       └── app/
│       │           ├── app.component.ts / .html           ← Tab-close beacon, ActivityLog wiring, ripple init
│       │           ├── app.config.ts                      ← Providers: Aura theme (#AE285D), router, interceptors,
│       │           │                                         API_BASE_URL, APP_NAME='npa', ACTIVITY_LOG_PATH='CrisMAc/AppLogsUpdate'
│       │           ├── app.routes.ts                      ← /login /home /dashboard /master-uploads /report-access
│       │           ├── core/
│       │           │   ├── interfaces/
│       │           │   │   ├── icomplaint.ts
│       │           │   │   ├── ilog.ts
│       │           │   │   └── iuser.ts
│       │           │   ├── menu/
│       │           │   │   ├── menu-api.service.ts        ← GET api/Login/CrisMAc/GetMenus
│       │           │   │   ├── menu.model.ts
│       │           │   │   ├── menu.service.ts            ← Signal-based menu state
│       │           │   │   └── menu-list-response-model.ts
│       │           │   └── services/
│       │           │       ├── auth.service.ts            ← Calls api/Login/CrisMAc/* endpoints
│       │           │       ├── complaint-details.service.ts
│       │           │       ├── core.service.ts            ← Token refresh interval, logout orchestration
│       │           │       └── (other NPA-specific services)
│       │           ├── layout/
│       │           │   ├── app.main.component.ts / .html  ← Shell: sidebar + topbar + router-outlet
│       │           │   ├── app.topbar.component.ts        ← NPA logo, user menu, session timer
│       │           │   ├── app.menu.component.ts
│       │           │   ├── app.menuitem.component.ts
│       │           │   ├── app.menu.service.ts
│       │           │   ├── app.footer.component.ts
│       │           │   └── app.rightpanel.component.ts / .html
│       │           ├── pages/
│       │           │   ├── auth/
│       │           │   │   ├── login/                     ← NPA-branded: crimson-pink (#AE285D), NPA logo
│       │           │   │   │   ├── login.component.ts     ← 14.4 KB — SSO + OTP + standard login flows
│       │           │   │   │   ├── login.component.html
│       │           │   │   │   └── login.component.scss
│       │           │   │   ├── reset-password/            ← 17 KB — token-based reset flow
│       │           │   │   └── change-password/           ← 15.7 KB — authenticated password change
│       │           │   ├── main-dashboard/
│       │           │   │   └── dashboard/
│       │           │   │       └── dashboard.component.ts ← Stub (charts TBD)
│       │           │   ├── master-uploads/                ← PRIMARY FEATURE (27.4 KB)
│       │           │   │   ├── master-uploads.component.ts  ← Maker-Checker grid + upload + authorize/reject
│       │           │   │   ├── master-uploads.component.html
│       │           │   │   ├── master-uploads.component.scss
│       │           │   │   └── master-uploads.service.ts
│       │           │   └── report-access/
│       │           │       ├── report-access.component.ts
│       │           │       └── report-access.service.ts
│       │           ├── utilities/
│       │           │   ├── base-href.service.ts
│       │           │   ├── common-module.ts
│       │           │   └── constant.ts
│       │           └── environments/
│       │               ├── environment.ts
│       │               ├── environment.dev.ts
│       │               └── environment.prod.ts
│       │
│       │  ── ADF Application ─────────────────────────────
│       └── adf-app/
│           ├── tsconfig.app.json
│           └── src/
│               ├── index.html                             ← "CrisMAC ADF" title + favicon
│               ├── main.ts                                ← bootstrapApplication(AppComponent, appConfig)
│               ├── styles.scss
│               └── app/
│                   ├── app.component.ts / .html           ← Same shell pattern as npa-app
│                   ├── app.config.ts                      ← Providers: Aura theme (ADF brand colour),
│                   │                                         APP_NAME='adf', ACTIVITY_LOG_PATH='ADF/AppLogsUpdate'
│                   ├── app.routes.ts                      ← ADF-specific routes + MenuIds
│                   ├── core/
│                   │   ├── menu/                          ← Identical structure to npa-app/core/menu
│                   │   └── services/
│                   │       ├── auth.service.ts            ← Calls api/Login/ADF/* endpoints
│                   │       └── core.service.ts
│                   ├── layout/                            ← ADF logo + brand colour in topbar
│                   │   ├── app.main.component.ts / .html
│                   │   ├── app.topbar.component.ts
│                   │   ├── app.menu.component.ts
│                   │   ├── app.menuitem.component.ts
│                   │   ├── app.menu.service.ts
│                   │   └── app.footer.component.ts
│                   ├── pages/
│                   │   ├── auth/
│                   │   │   ├── login/                     ← ADF-branded login (ADF colour + logo)
│                   │   │   ├── reset-password/
│                   │   │   └── change-password/
│                   │   └── (ADF domain pages — data-flow, reports, etc.)
│                   ├── utilities/
│                   └── environments/
│                       ├── environment.ts
│                       ├── environment.dev.ts
│                       └── environment.prod.ts
│
│  ── Backend .NET solution ───────────────────────────────
└── backend/
    ├── CrismacAxis.sln                                    ← Solution: Common.Core + npa-api + adf-api
    └── src/
        │
        │  ── Shared Class Library ───────────────────────
        ├── Common.Core/
        │   ├── Crismac.Axis.Common.Core.csproj            ← Microsoft.NET.Sdk (no HTTP pipeline)
        │   ├── Constants/
        │   │   └── HttpStatusCodes.cs
        │   ├── Data/
        │   │   ├── IDapperContext.cs                      ← CreateConnection() + CreateNamedConnection()
        │   │   └── DapperContext.cs
        │   ├── Entities/                                  ← Generic, app-agnostic DTOs
        │   │   ├── ApiResponseDto.cs
        │   │   ├── AppLogDto.cs
        │   │   ├── RefreshToken.cs
        │   │   ├── RequestLogEntry.cs
        │   │   └── TokenApiModel.cs
        │   ├── Extensions/
        │   │   └── ServiceCollectionExtensions.cs         ← AddCommonAppSettings()
        │   │                                                 AddCommonModelServices()
        │   │                                                 AddDapperContext()
        │   │                                                 AddCommonJwtAuthentication()
        │   │                                                 AddCommonMiddleware()
        │   ├── Filters/
        │   │   ├── ActionLoggingFilter.cs
        │   │   ├── MenuGuardAttribute.cs
        │   │   ├── RateLimitAttribute.cs
        │   │   └── RequestLimitAttribute.cs
        │   ├── Helpers/
        │   │   ├── AppSettings.cs                         ← JWTSettings, AppSettings, SmsOptions,
        │   │   │                                             EmailOptions, CustomLogging, CustomUserSettings
        │   │   ├── CorsSettings.cs
        │   │   ├── DbErrorHandler.cs
        │   │   ├── JwtClaimsMap.cs                        ← NEW — configurable claim name strings
        │   │   └── UploadSettings.cs
        │   ├── Middlewares/
        │   │   ├── CorrelationHeadersMiddleware.cs        ← Injects X-Correlation-Id per request
        │   │   ├── CurrentUserEnrichmentMiddleware.cs     ← Adds user context to Serilog scope
        │   │   ├── EncryptionMiddleware.cs
        │   │   ├── ExceptionHandlerMiddleware.cs
        │   │   ├── RequestResponseLoggingMiddleware.cs
        │   │   └── SqlDebugMiddleware.cs
        │   ├── Repositories/                              ← Interfaces ONLY (contracts for area-apis)
        │   │   ├── IUserRepository.cs                     ← GetByUserIdAsync(string userId)
        │   │   └── IRefreshTokenRepository.cs             ← IsTokenUnique(), UpdateRefreshTokenAsync()
        │   ├── Services/
        │   │   ├── Crypto/
        │   │   │   ├── ICryptoService.cs
        │   │   │   └── CryptoAes256CbcService.cs
        │   │   ├── ExcelService/
        │   │   │   ├── IExcelService.cs
        │   │   │   └── ExcelService.cs                    ← EPPlus-based Excel parse + export
        │   │   ├── PasswordHasher/
        │   │   │   ├── IPasswordHasher.cs
        │   │   │   └── BcryptPasswordHasher.cs
        │   │   ├── ExceptionErrorFileLogService.cs
        │   │   ├── FileLogService.cs
        │   │   ├── FileValidationService.cs               ← MIME, magic bytes, extension, size (18.8 KB)
        │   │   ├── JwtService.cs                          ← JWT gen/validation; claim names from JwtClaimsMap
        │   │   └── NotificationService.cs                 ← Email + SMS dispatch
        │   └── Utilities/
        │       ├── fileUtil.cs
        │       ├── LoggingUtil.cs
        │       ├── MenuGuardConstants.cs
        │       ├── OtpGenerator.cs
        │       └── UrlUtils.cs
        │
        └── AREAS/
            │
            │  ── NPA Area API ─────────────────────────────────
            ├── npa-api/
            │   ├── Crismac.Axis.Npa.Api.csproj            ← <ProjectReference> → Common.Core
            │   ├── Program.cs                             ← Kestrel limits, Serilog, CORS, Auth,
            │   │                                             AddCommon*() + AddNpa*() extensions
            │   ├── appsettings.json                       ← AppSettings:AreaName="LAB"
            │   │                                             ConnectionString → NPA_DB
            │   │                                             JWTSettings, UploadSettings, CorsSettings
            │   ├── appsettings.Development.json
            │   ├── Controllers/
            │   │   ├── AppLogsController.cs               ← GET activity logs
            │   │   ├── ChunkUploadController.cs           ← POST upload / GET meta / authorize / reject / download (26 KB)
            │   │   ├── LoginController.cs                 ← Route: api/Login/CrisMAc/* (22.9 KB)
            │   │   ├── ReportAcessController.cs
            │   │   ├── ResourcesController.cs
            │   │   └── UsersController.cs
            │   ├── Entities/                              ← NPA-specific DTOs
            │   │   ├── AutheticateRequestwithOtp.cs
            │   │   ├── AutheticateResponse.cs
            │   │   ├── AuthorizationResult.cs
            │   │   ├── HeaderDefinition.cs
            │   │   ├── UploadMasterEntity.cs              ← Maker-Checker upload record (4.2 KB)
            │   │   └── User.cs                            ← NPA user model with roles/menus (7.9 KB)
            │   ├── Extensions/
            │   │   ├── AppSettingsExtension.cs
            │   │   ├── JwtSecurityTokenHandlerExtension.cs
            │   │   └── ServiceCollectionExtensions.cs     ← AddNpaRepositoryServices()
            │   │                                             AddNpaModelServices()
            │   ├── Repositories/
            │   │   ├── ChunkUploadRepository/
            │   │   │   ├── IChunkUploadRepository.cs
            │   │   │   └── ChunkUploadRepository.cs       ← SqlBulkCopy + staging SP calls
            │   │   ├── IRefreshTokenRepository/           ← Implements Common.Core.IRefreshTokenRepository
            │   │   │   ├── IRefreshTokenRepository.cs
            │   │   │   └── RefreshTokenRepository.cs
            │   │   ├── LoginRepository/
            │   │   │   ├── ILoginRepository.cs
            │   │   │   └── LoginRepository.cs             ← sp_SelectLoginDetails, OTP, AD lookup
            │   │   ├── ReportAccessRepository/
            │   │   │   ├── IReportAcessRepository.cs
            │   │   │   └── ReportAcessRepository.cs
            │   │   ├── UserRpository/                     ← note: existing typo kept
            │   │   │   ├── IUserRepository.cs             ← Implements Common.Core.IUserRepository
            │   │   │   └── UserRepository.cs
            │   │   └── DatabaseLogService.cs
            │   ├── Services/
            │   │   ├── AuthorizationService.cs            ← Maker-Checker: opFlags 1/16/17, NP/A/R status (13 KB)
            │   │   ├── ChunkUploadService/                ← 7-step upload pipeline
            │   │   └── UserService.cs                     ← AD lookup, role classification (13.3 KB)
            │   ├── Resources/
            │   │   └── Export/                            ← Excel export templates
            │   ├── Logs/
            │   │   ├── ApiLogs/
            │   │   └── AppLogs/
            │   └── Properties/
            │       └── PublishProfiles/
            │
            │  ── ADF Area API ─────────────────────────────────
            └── adf-api/
                ├── Crismac.Axis.Adf.Api.csproj            ← <ProjectReference> → Common.Core
                ├── Program.cs                             ← Same Common.Core extensions + AddAdf*()
                ├── appsettings.json                       ← AppSettings:AreaName="ADF"
                │                                             ConnectionString → ADF_DB
                ├── appsettings.Development.json
                ├── Controllers/
                │   ├── LoginController.cs                 ← Route: api/Login/ADF/* (mirrors NPA pattern)
                │   └── (ADF domain controllers)
                ├── Entities/                              ← ADF-specific user model, DTOs
                │   └── User.cs
                ├── Extensions/
                │   └── ServiceCollectionExtensions.cs     ← AddAdfRepositoryServices()
                │                                             AddAdfModelServices()
                ├── Repositories/
                │   ├── IRefreshTokenRepository/           ← Implements Common.Core.IRefreshTokenRepository
                │   │   └── RefreshTokenRepository.cs      ← ADF refresh_tokens table
                │   ├── LoginRepository/
                │   │   └── LoginRepository.cs             ← ADF [adf].[sp_SelectLoginDetails]
                │   ├── UserRpository/                     ← Implements Common.Core.IUserRepository
                │   │   └── UserRepository.cs              ← ADF user table schema
                │   └── (ADF domain repositories)
                ├── Services/
                │   └── UserService.cs                     ← ADF AD lookup + role classification
                └── Logs/
                    ├── ApiLogs/
                    └── AppLogs/
```

---

### Key Relationships at a Glance

```
┌─────────────────────────────────────────────────────────────────────┐
│  shared-lib (dist/shared-lib)                                        │
│  AuthStateService · AuthGuard · MenuGuard · GridComponent            │
│  ErrorInterceptor · AuthInterceptor · EncryptionInterceptor          │
│  CryptoService · IdleDetectionService · LocalStorageService          │
│  SessionTimerComponent · ConfigService · UtilsService                │
│  InjectionTokens: API_BASE_URL · APP_NAME · ACTIVITY_LOG_PATH        │
└───────────────────┬─────────────────────────┬───────────────────────┘
                    │ consumed via tsconfig    │ consumed via tsconfig
                    ▼ path alias               ▼ path alias
          ┌─────────────────┐       ┌──────────────────────┐
          │  npa-app :4200  │       │   adf-app :4201       │
          │  theme #AE285D  │       │   theme <ADF colour>  │
          │  APP_NAME='npa' │       │   APP_NAME='adf'      │
          │  /CrisMAc/...   │       │   /ADF/...            │
          └────────┬────────┘       └──────────┬────────────┘
                   │ HTTPS JWT                 │ HTTPS JWT
                   ▼                           ▼
          ┌─────────────────┐       ┌──────────────────────┐
          │  npa-api :7294  │       │   adf-api :7295       │
          │  AreaName=LAB   │       │   AreaName=ADF        │
          │  DB=NPA_DB      │       │   DB=ADF_DB           │
          └────────┬────────┘       └──────────┬────────────┘
                   │ ProjectReference           │ ProjectReference
                   └──────────┬─────────────────┘
                              ▼
              ┌───────────────────────────────┐
              │         Common.Core           │
              │  JwtService · Crypto          │
              │  FileValidation · Excel       │
              │  Notification · PasswordHash  │
              │  All Middlewares · Filters    │
              │  IDapperContext · Entities    │
              │  IUserRepository (interface)  │
              │  IRefreshTokenRepository      │
              └───────────────────────────────┘
```

### What Is Shared vs. What Is App-Owned

| Concern | shared-lib | npa-app | adf-app | Common.Core | npa-api | adf-api |
|---------|:----------:|:-------:|:-------:|:-----------:|:-------:|:-------:|
| Auth state (signals) | ✅ | | | | | |
| JWT interceptor | ✅ | | | | | |
| Route guards | ✅ | | | | | |
| Grid / modal components | ✅ | | | | | |
| Session timer + idle | ✅ | | | | | |
| Theme / branding | | ✅ | ✅ | | | |
| Login page UI | | ✅ | ✅ | | | |
| App routes + MenuIds | | ✅ | ✅ | | | |
| Login endpoint prefix | | ✅ (`CrisMAc`) | ✅ (`ADF`) | | | |
| Domain pages/features | | ✅ | ✅ | | | |
| JWT generation + validation | | | | ✅ | | |
| File validation (MIME/sig) | | | | ✅ | | |
| Email / SMS notification | | | | ✅ | | |
| Excel parse + export | | | | ✅ | | |
| All middleware pipeline | | | | ✅ | | |
| DB connection factory | | | | ✅ | | |
| IUserRepository (contract) | | | | ✅ | | |
| User model + AD lookup | | | | | ✅ | ✅ |
| Domain controllers | | | | | ✅ | ✅ |
| Domain repositories + SPs | | | | | ✅ | ✅ |
| Maker-Checker logic | | | | | ✅ | |
| Upload pipeline (7 steps) | | | | | ✅ | |
| App bootstrap (Program.cs) | | | | | ✅ | ✅ |

---

*Last updated: 2026-04-09 — post-refactor hierarchy based on direct codebase analysis*
