# Axis Monorepo - Setup Guide

**Enterprise-grade monorepo setup with git commit-based workflow**

---

## Prerequisites

- Git
- Volta (for Node.js version management)
- .NET 9 SDK
- VS Code (recommended)

---

## Setup Process

Follow these steps in order. Each step represents a git commit.

---

## Step 1: Initialize Repository and Volta

```bash
# Create project directory
mkdir crismac-npa-axis
cd crismac-npa-axis

# Initialize git
git init
git checkout -b main

# Initialize volta for node version management
volta pin node@20.11.0
volta pin pnpm@8.15.0
```

**Create: `package.json`**
```json
{
  "name": "crismac-npa-axis",
  "version": "1.0.0",
  "private": true,
  "description": "Crismac NPA Axis - Enterprise Monorepo",
  "engines": {
    "node": ">=20.11.0",
    "pnpm": ">=8.15.0"
  },
  "volta": {
    "node": "20.11.0",
    "pnpm": "8.15.0"
  },
  "packageManager": "pnpm@8.15.0"
}
```

**Commit:**
```bash
git add .
git commit -m "chore: initialize repository with volta"
```

---

## Step 2: Setup Workspace Configuration

**Create: `.npmrc`**
```ini
# Force pnpm usage
package-manager=pnpm

# Workspace configuration
shamefully-hoist=true
strict-peer-dependencies=false
auto-install-peers=true

# Lockfile
lockfile=true
```

**Create: `pnpm-workspace.yaml`**
```yaml
# Define workspace packages
# This file tells pnpm which directories contain packages

packages:
  # Frontend workspace
  - 'apps/frontend'
  - 'libs/*'
```

**Create: `.gitignore`**
```gitignore
# Dependencies
node_modules/
.pnpm-store/

# Build outputs
dist/
build/
out/
*.js.map

# Environment
.env
.env.local
.env.*.local

# IDE
.vscode/*
!.vscode/settings.json
!.vscode/extensions.json
.idea/

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Cache
.cache/
.angular/
.nx/

# Testing
coverage/
.nyc_output/
```

**Commit:**
```bash
git add .
git commit -m "chore: configure pnpm workspace"
```

---

## Step 3: Create Monorepo Structure

```bash
# Create directory structure
mkdir -p apps/frontend
mkdir -p apps/backend/src
mkdir -p libs
mkdir -p docs
```

**Create: `README.md`**
```markdown
# Crismac NPA Axis

Enterprise monorepo for NPA Axis management system.

## Architecture

- **Frontend**: Angular 18 standalone workspace
- **Backend**: .NET 9 Web API
- **Package Manager**: pnpm with Volta
- **Node Version**: Locked with Volta

## Structure

\`\`\`
crismac-npa-axis/
├── apps/
│   ├── frontend/     # Angular application
│   └── backend/      # .NET API
├── libs/             # Shared libraries
├── docs/             # Documentation
└── package.json      # Root workspace
\`\`\`

## Quick Start

\`\`\`bash
# Install dependencies
pnpm install

# Start development
pnpm run dev
\`\`\`

## Prerequisites

- Volta (installs correct Node.js/pnpm versions automatically)
- .NET 9 SDK
```

**Commit:**
```bash
git add .
git commit -m "chore: create monorepo directory structure"
```

---

## Step 4: Setup Angular Frontend Workspace

```bash
cd apps/frontend

# Create Angular workspace without default application
ng new . --create-application=false --skip-git

# Generate the main application
ng generate application axis-masters-app --routing --style=scss --standalone

# Install PrimeNG and dependencies
pnpm add primeng primeicons
pnpm add -D @angular-eslint/builder @angular-eslint/eslint-plugin @angular-eslint/eslint-plugin-template @angular-eslint/template-parser @angular-eslint/schematics
```

**Note**: The `--create-application=false` flag creates a workspace foundation without a default app, allowing better multi-project organization.

**Update: `apps/frontend/package.json`**

Add these scripts:
```json
{
  "name": "axis-frontend",
  "version": "1.0.0",
  "scripts": {
    "start": "ng serve axis-masters-app",
    "build": "ng build axis-masters-app",
    "build:lib": "ng build shared-lib",
    "watch": "ng build axis-masters-app --watch --configuration development",
    "test": "ng test",
    "lint": "ng lint"
  }
}
```

**Angular workspace structure created:**
```
apps/frontend/
├── projects/
│   └── axis-masters-app/
│       ├── public/              # Static assets (Angular 18+)
│       ├── src/
│       │   ├── app/
│       │   └── main.ts
│       ├── tsconfig.app.json
│       └── tsconfig.spec.json
├── angular.json                 # Workspace configuration
├── package.json
└── tsconfig.json                # Base TypeScript config
```

The `angular.json` is automatically configured by Angular CLI with proper build, serve, and test targets for `axis-masters-app`

**Create: `apps/frontend/.gitignore`**
```gitignore
# Compiled output
/dist
/tmp
/out-tsc
/bazel-out

# Node
/node_modules

# IDEs
.idea/
.vscode/
*.swp
*.swo

# System
.DS_Store
Thumbs.db

# Angular
/.angular
```

**Create: `apps/frontend/README.md`**
```markdown
# Axis Frontend

Angular 18 standalone application.

## Commands

\`\`\`bash
# Development
pnpm run start

# Build
pnpm run build

# Test
pnpm run test

# Lint
pnpm run lint
\`\`\`

## Structure

\`\`\`
src/
├── app/              # Application code
│   ├── core/        # Core module (services, guards, interceptors)
│   ├── shared/      # Shared components, directives, pipes
│   ├── features/    # Feature modules
│   └── app.config.ts
├── assets/          # Static assets
└── environments/    # Environment configs
\`\`\`
```

**Create: `apps/frontend/projects/axis-masters-app/src/environments/environment.ts`**
```typescript
/**
 * Development environment configuration
 * 
 * This file contains environment-specific settings for development.
 * Angular 18+ uses this file for development configuration.
 */

export const environment = {
  production: false,
  apiUrl: 'https://localhost:5001/api',
  appName: 'Axis Masters',
  version: '1.0.0'
};
```

**Create: `apps/frontend/projects/axis-masters-app/src/environments/environment.prod.ts`**
```typescript
/**
 * Production environment configuration
 * 
 * This file is used when building for production.
 * Set environment-specific values here.
 */

export const environment = {
  production: true,
  apiUrl: '/api',  // Relative URL for production
  appName: 'Axis Masters',
  version: '1.0.0'
};
```

**Note**: Angular 18+ uses the `public/` folder for static assets instead of `src/assets/`. The `public/` folder in `projects/axis-masters-app/` is for favicon, images, and other static files that should be copied as-is to the output.

**Update: `apps/frontend/projects/axis-masters-app/src/app/app.config.ts`**
```typescript
/**
 * Application configuration
 * 
 * This file sets up the application-wide providers and configuration.
 * It uses Angular's standalone API for a more modular architecture.
 */

import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideAnimations } from '@angular/platform-browser/animations';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([])),
    provideAnimations()
  ]
};
```

**Back to root:**
```bash
cd ../..
```

**Commit:**
```bash
git add .
git commit -m "feat: setup Angular frontend workspace with standalone architecture"
```

---

## Step 5: Create Shared Library Structure

```bash
cd apps/frontend

# Generate shared library for code reusability
ng generate library shared-lib --prefix=axis
```

**Note**: This creates a buildable library in `projects/shared-lib/` with its own package.json and build configuration.

**Create: `apps/frontend/projects/shared-lib/ng-package.json`**
```json
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/shared-lib",
  "lib": {
    "entryFile": "src/public-api.ts"
  }
}
```

**Update: `apps/frontend/projects/shared-lib/package.json`**
```json
{
  "name": "@axis/shared-lib",
  "version": "0.0.1",
  "peerDependencies": {
    "@angular/common": "^18.0.0",
    "@angular/core": "^18.0.0"
  },
  "dependencies": {
    "tslib": "^2.3.0"
  },
  "sideEffects": false
}
```

**Update: `apps/frontend/projects/shared-lib/src/public-api.ts`**
```typescript
/**
 * Public API Surface of shared-lib
 * 
 * This file defines what is exported from the shared library.
 * Only items exported here are accessible to applications using this library.
 * 
 * Usage in apps:
 *   import { MyComponent } from '@axis/shared-lib';
 */

// Export barrel files for each feature area
export * from './lib/components';
export * from './lib/services';
export * from './lib/models';
export * from './lib/guards';
export * from './lib/interceptors';
export * from './lib/directives';
export * from './lib/pipes';
```

**Create barrel export files:**

**`apps/frontend/projects/shared-lib/src/lib/components/index.ts`**
```typescript
/**
 * Shared Components Barrel Export
 * 
 * Export all reusable UI components from this file.
 * These components should be stateless and highly reusable.
 */

// Example exports (add as you create components):
// export * from './button/button.component';
// export * from './card/card.component';
// export * from './table/table.component';
```

**`apps/frontend/projects/shared-lib/src/lib/services/index.ts`**
```typescript
/**
 * Shared Services Barrel Export
 * 
 * Export all shared services handling cross-cutting concerns.
 */

// Example exports:
// export * from './api/api.service';
// export * from './auth/auth.service';
// export * from './storage/storage.service';
```

**`apps/frontend/projects/shared-lib/src/lib/models/index.ts`**
```typescript
/**
 * Shared Models Barrel Export
 * 
 * Export TypeScript interfaces, types, and enums used across applications.
 */

// Example exports:
// export * from './user.model';
// export * from './api-response.model';
// export * from './pagination.model';
```

**`apps/frontend/projects/shared-lib/src/lib/guards/index.ts`**
```typescript
/**
 * Shared Guards Barrel Export
 * 
 * Export route guards for navigation control.
 */

// Example exports:
// export * from './auth.guard';
// export * from './role.guard';
```

**`apps/frontend/projects/shared-lib/src/lib/interceptors/index.ts`**
```typescript
/**
 * Shared Interceptors Barrel Export
 * 
 * Export HTTP interceptors for request/response handling.
 */

// Example exports:
// export * from './auth.interceptor';
// export * from './error.interceptor';
// export * from './loading.interceptor';
```

**`apps/frontend/projects/shared-lib/src/lib/directives/index.ts`**
```typescript
/**
 * Shared Directives Barrel Export
 * 
 * Export custom attribute and structural directives.
 */

// Example exports:
// export * from './tooltip.directive';
// export * from './auto-focus.directive';
```

**`apps/frontend/projects/shared-lib/src/lib/pipes/index.ts`**
```typescript
/**
 * Shared Pipes Barrel Export
 * 
 * Export custom transformation pipes.
 */

// Example exports:
// export * from './safe-html.pipe';
// export * from './truncate.pipe';
```

**Configure TypeScript Paths for Development**

**Update: `apps/frontend/tsconfig.json`**

Add paths configuration for easy imports and hot-reload during development:

```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "outDir": "./dist/out-tsc",
    "strict": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "declaration": false,
    "experimentalDecorators": true,
    "moduleResolution": "bundler",
    "importHelpers": true,
    "target": "ES2022",
    "module": "ES2022",
    "lib": ["ES2022", "dom"],
    "paths": {
      "@axis/shared-lib": [
        "projects/shared-lib/src/public-api.ts"
      ],
      "@axis/shared-lib/*": [
        "projects/shared-lib/src/*"
      ]
    }
  },
  "angularCompilerOptions": {
    "enableI18nLegacyMessageIdFormat": false,
    "strictInjectionParameters": true,
    "strictInputAccessModifiers": true,
    "strictTemplates": true
  }
}
```

**Important Note on Development vs Production Paths:**

**For Development** (hot-reload, no rebuild needed):
```json
"paths": {
  "@axis/shared-lib": ["projects/shared-lib/src/public-api.ts"]
}
```

**For Production Build** (use built library):
```json
"paths": {
  "@axis/shared-lib": ["dist/shared-lib"]
}
```

You can switch between these as needed, or maintain separate `tsconfig.dev.json` and `tsconfig.prod.json` files.

**Using the Shared Library in Applications:**

In your `axis-masters-app`, you can now import from the shared library:

```typescript
// projects/axis-masters-app/src/app/app.component.ts
import { Component } from '@angular/core';
// Import from shared library using path alias
// import { ButtonComponent } from '@axis/shared-lib';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    // ButtonComponent  // Add shared components here
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'axis-masters-app';
}
```

**Create: `apps/frontend/projects/shared-lib/README.md`**
```markdown
# Shared Library

Reusable components, services, and utilities for the Axis workspace.

## Purpose

This library contains code shared across multiple applications:
- **Components**: Reusable UI components (buttons, cards, tables)
- **Services**: Shared business logic (API, auth, storage)
- **Models**: TypeScript interfaces and types
- **Guards**: Route protection logic
- **Interceptors**: HTTP request/response handlers
- **Directives**: Custom DOM manipulation
- **Pipes**: Data transformation

## Usage

### Importing in Applications

\`\`\`typescript
// Import components
import { ButtonComponent } from '@axis/shared-lib';

// Import services
import { ApiService } from '@axis/shared-lib';

// Import models
import { User } from '@axis/shared-lib';
\`\`\`

### Development Workflow

**During Development** (with hot-reload):
- No need to rebuild library
- Changes reflect immediately in apps
- TypeScript paths point to source files

**For Production Build**:
\`\`\`bash
# Build the library first
ng build shared-lib

# Then build the application
ng build axis-masters-app
\`\`\`

## Structure

\`\`\`
projects/shared-lib/
├── src/
│   ├── lib/
│   │   ├── components/     # Reusable UI components
│   │   ├── services/       # Shared services
│   │   ├── models/         # TypeScript interfaces
│   │   ├── guards/         # Route guards
│   │   ├── interceptors/   # HTTP interceptors
│   │   ├── directives/     # Custom directives
│   │   └── pipes/          # Custom pipes
│   └── public-api.ts       # Public API surface
├── ng-package.json         # Library build config
├── package.json            # Library dependencies
├── tsconfig.lib.json       # Library TypeScript config
└── README.md               # This file
\`\`\`

## Best Practices

1. **Keep it Pure**: Components should be stateless when possible
2. **Barrel Exports**: Use index.ts files for clean imports
3. **Documentation**: Add JSDoc comments for public APIs
4. **Testing**: Write unit tests for all shared code
5. **Breaking Changes**: Avoid breaking changes; use versioning
\`\`\`
```

**Back to root:**
```bash
cd ../..
```

**Commit:**
```bash
git add .
git commit -m "feat: create shared library structure for code reusability"
```

---

## Step 6: Setup .NET Backend API

```bash
cd apps/backend

# Create .NET solution
dotnet new sln -n Crismac.Npa.Axis

# Create Web API project
dotnet new webapi -n Crismac.Npa.Axis.Api -o src/Crismac.Npa.Axis.Api

# Add project to solution
dotnet sln add src/Crismac.Npa.Axis.Api/Crismac.Npa.Axis.Api.csproj

# Add common packages
cd src/Crismac.Npa.Axis.Api
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Swashbuckle.AspNetCore
```

**Update: `apps/backend/src/Crismac.Npa.Axis.Api/Program.cs`**
```csharp
/**
 * Application Entry Point
 * 
 * This file configures and runs the ASP.NET Core web application.
 * It sets up services, middleware pipeline, and application configuration.
 */

using Microsoft.OpenApi.Models;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();

// Configure CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

// Configure Swagger/OpenAPI
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Crismac NPA Axis API",
        Version = "v1",
        Description = "API for NPA Axis Management System"
    });
});

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Enable CORS
app.UseCors("AllowFrontend");

app.UseAuthorization();

app.MapControllers();

app.Run();
```

**Update: `apps/backend/src/Crismac.Npa.Axis.Api/appsettings.json`**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=AxisNPA;Trusted_Connection=true;TrustServerCertificate=true;"
  },
  "Jwt": {
    "SecretKey": "your-secret-key-change-in-production",
    "Issuer": "axis-api",
    "Audience": "axis-app",
    "ExpiryMinutes": 60
  }
}
```

**Update: `apps/backend/src/Crismac.Npa.Axis.Api/appsettings.Development.json`**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=AxisNPA_Dev;Trusted_Connection=true;TrustServerCertificate=true;"
  }
}
```

**Create: `apps/backend/src/Crismac.Npa.Axis.Api/Controllers/HealthController.cs`**
```csharp
/**
 * Health Check Controller
 * 
 * Provides endpoints for monitoring application health and status.
 * Used by monitoring tools and load balancers.
 */

using Microsoft.AspNetCore.Mvc;

namespace Crismac.Npa.Axis.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class HealthController : ControllerBase
{
    private readonly ILogger<HealthController> _logger;

    public HealthController(ILogger<HealthController> logger)
    {
        _logger = logger;
    }

    /// <summary>
    /// Health check endpoint
    /// </summary>
    /// <returns>API status</returns>
    [HttpGet]
    public IActionResult Get()
    {
        _logger.LogInformation("Health check requested");
        
        return Ok(new
        {
            Status = "Healthy",
            Timestamp = DateTime.UtcNow,
            Version = "1.0.0",
            Environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")
        });
    }
}
```

**Create: `apps/backend/.gitignore`**
```gitignore
## Ignore Visual Studio temporary files, build results, and files generated by popular add-ons

# User-specific files
*.rsuser
*.suo
*.user
*.userosscache
*.sln.docstates

# Build results
[Dd]ebug/
[Dd]ebugPublic/
[Rr]elease/
[Rr]eleases/
x64/
x86/
[Ww][Ii][Nn]32/
[Aa][Rr][Mm]/
[Aa][Rr][Mm]64/
bld/
[Bb]in/
[Oo]bj/

# Visual Studio cache/options
.vs/

# .NET Core
project.lock.json
project.fragment.lock.json
artifacts/

# Files built by Visual Studio
*.log
*.vspscc
*.vssscc

# Environment
.env
.env.local

# Database
*.db
*.db-wal
*.db-shm
```

**Create: `apps/backend/README.md`**
```markdown
# Axis Backend API

.NET 9 Web API for NPA Axis management system.

## Commands

\`\`\`bash
# Restore dependencies
dotnet restore

# Build
dotnet build

# Run
dotnet run --project src/Crismac.Npa.Axis.Api

# Watch (auto-reload)
dotnet watch run --project src/Crismac.Npa.Axis.Api

# Test
dotnet test
\`\`\`

## Structure

\`\`\`
src/
└── Crismac.Npa.Axis.Api/
    ├── Controllers/     # API endpoints
    ├── Models/         # Domain models
    ├── Services/       # Business logic
    ├── Data/           # Database context
    └── Program.cs      # Entry point
\`\`\`

## API Documentation

- Swagger UI: https://localhost:5001/swagger
- Health Check: https://localhost:5001/api/health
```

**Back to root:**
```bash
cd ../..
```

**Commit:**
```bash
git add .
git commit -m "feat: setup .NET 9 Web API with CORS and Swagger"
```

---

## Step 7: Configure Root Workspace Scripts

**Update: `package.json`** (at root)
```json
{
  "name": "crismac-npa-axis",
  "version": "1.0.0",
  "private": true,
  "description": "Crismac NPA Axis - Enterprise Monorepo",
  "scripts": {
    "frontend:dev": "cd apps/frontend && pnpm run start",
    "frontend:build": "cd apps/frontend && pnpm run build",
    "frontend:test": "cd apps/frontend && pnpm run test",
    "frontend:lint": "cd apps/frontend && pnpm run lint",
    "backend:dev": "cd apps/backend && dotnet run --project src/Crismac.Npa.Axis.Api",
    "backend:build": "cd apps/backend && dotnet build",
    "backend:test": "cd apps/backend && dotnet test",
    "backend:watch": "cd apps/backend && dotnet watch run --project src/Crismac.Npa.Axis.Api",
    "dev": "echo 'Run frontend:dev and backend:watch in separate terminals'",
    "build": "pnpm run frontend:build && pnpm run backend:build",
    "install:frontend": "cd apps/frontend && pnpm install",
    "clean": "rm -rf apps/frontend/node_modules apps/frontend/dist apps/backend/bin apps/backend/obj"
  },
  "engines": {
    "node": ">=20.11.0",
    "pnpm": ">=8.15.0"
  },
  "volta": {
    "node": "20.11.0",
    "pnpm": "8.15.0"
  },
  "packageManager": "pnpm@8.15.0"
}
```

**Commit:**
```bash
git add .
git commit -m "chore: configure root workspace scripts"
```

---

## Step 8: Setup VS Code Configuration

**Create: `.vscode/settings.json`**
```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": "explicit"
  },
  "files.eol": "\n",
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "[typescript]": {
    "editor.defaultFormatter": "vscode.typescript-language-features"
  },
  "[csharp]": {
    "editor.defaultFormatter": "ms-dotnettools.csharp"
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/bin": true,
    "**/obj": true
  }
}
```

**Create: `.vscode/extensions.json`**
```json
{
  "recommendations": [
    "angular.ng-template",
    "ms-dotnettools.csharp",
    "ms-dotnettools.csdevkit",
    "dbaeumer.vscode-eslint"
  ]
}
```

**Create: `.vscode/launch.json`**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Backend: Launch",
      "type": "coreclr",
      "request": "launch",
      "preLaunchTask": "build-backend",
      "program": "${workspaceFolder}/apps/backend/src/Crismac.Npa.Axis.Api/bin/Debug/net9.0/Crismac.Npa.Axis.Api.dll",
      "args": [],
      "cwd": "${workspaceFolder}/apps/backend/src/Crismac.Npa.Axis.Api",
      "stopAtEntry": false,
      "env": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    {
      "name": "Frontend: Chrome",
      "type": "chrome",
      "request": "launch",
      "preLaunchTask": "serve-frontend",
      "url": "http://localhost:4200",
      "webRoot": "${workspaceFolder}/apps/frontend"
    }
  ]
}
```

**Create: `.vscode/tasks.json`**
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build-backend",
      "command": "dotnet",
      "type": "process",
      "args": [
        "build",
        "${workspaceFolder}/apps/backend/Crismac.Npa.Axis.sln"
      ],
      "problemMatcher": "$msCompile"
    },
    {
      "label": "serve-frontend",
      "type": "shell",
      "command": "pnpm run frontend:dev",
      "isBackground": true,
      "problemMatcher": {
        "pattern": {
          "regexp": "."
        },
        "background": {
          "activeOnStart": true,
          "beginsPattern": ".",
          "endsPattern": "."
        }
      }
    }
  ]
}
```

**Commit:**
```bash
git add .
git commit -m "chore: configure VS Code workspace settings"
```

---

## Step 9: Add EditorConfig for Consistent Formatting

**Create: `.editorconfig`**
```ini
# EditorConfig is awesome: https://EditorConfig.org

# Top-most EditorConfig file
root = true

# All files
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space

# TypeScript/JavaScript
[*.{ts,js}]
indent_size = 2

# C#
[*.cs]
indent_size = 4

# JSON
[*.json]
indent_size = 2

# YAML
[*.{yml,yaml}]
indent_size = 2

# Markdown
[*.md]
trim_trailing_whitespace = false

# C# specific rules
[*.cs]
# Organize usings
dotnet_sort_system_directives_first = true
dotnet_separate_import_directive_groups = false

# this. preferences
dotnet_style_qualification_for_field = false:suggestion
dotnet_style_qualification_for_property = false:suggestion
dotnet_style_qualification_for_method = false:suggestion
dotnet_style_qualification_for_event = false:suggestion

# Language keywords vs BCL types preferences
dotnet_style_predefined_type_for_locals_parameters_members = true:suggestion
dotnet_style_predefined_type_for_member_access = true:suggestion

# New line preferences
csharp_new_line_before_open_brace = all
csharp_new_line_before_else = true
csharp_new_line_before_catch = true
csharp_new_line_before_finally = true
```

**Commit:**
```bash
git add .
git commit -m "chore: add editorconfig for consistent code formatting"
```

---

## Step 10: Create Documentation Structure

**Create: `docs/ARCHITECTURE.md`**
```markdown
# Architecture Overview

## Monorepo Structure

This is a monorepo containing both frontend (Angular) and backend (.NET) applications.

### Directory Structure

\`\`\`
crismac-npa-axis/
├── apps/
│   ├── frontend/          # Angular workspace
│   │   ├── src/          # Main application
│   │   └── projects/     # Shared libraries
│   └── backend/          # .NET solution
│       └── src/          # API projects
├── libs/                 # Future: Shared libraries
├── docs/                 # Documentation
└── package.json          # Workspace root
\`\`\`

## Technology Stack

### Frontend
- **Framework**: Angular 18 (Standalone)
- **UI Library**: PrimeNG
- **State Management**: RxJS
- **Build Tool**: Angular CLI
- **Package Manager**: pnpm

### Backend
- **Framework**: .NET 9
- **API Style**: RESTful Web API
- **ORM**: Entity Framework Core
- **Database**: SQL Server
- **Documentation**: Swagger/OpenAPI

## Development Workflow

### Frontend Development
\`\`\`bash
cd apps/frontend
pnpm run start    # Runs on http://localhost:4200
\`\`\`

### Backend Development
\`\`\`bash
cd apps/backend
dotnet watch run --project src/Crismac.Npa.Axis.Api  # Runs on https://localhost:5001
\`\`\`

## Key Design Decisions

1. **Standalone Components**: Using Angular's standalone API for better tree-shaking
2. **Multi-Project Workspace**: Supporting code reusability through shared libraries
3. **CORS Configuration**: Frontend and backend on different ports during development
4. **Volta**: Node version management for consistent development environment

## API Communication

- Frontend makes HTTP calls to: `https://localhost:5001/api`
- Backend CORS allows: `http://localhost:4200`
- Authentication: JWT Bearer tokens (to be implemented)

## Environment Configuration

### Frontend Environments
- `environment.ts` - Development
- `environment.prod.ts` - Production

### Backend Configuration
- `appsettings.json` - Base settings
- `appsettings.Development.json` - Development overrides
```

**Create: `docs/GETTING_STARTED.md`**
```markdown
# Getting Started

## Prerequisites

1. **Install Volta**
   \`\`\`bash
   curl https://get.volta.sh | bash
   \`\`\`

2. **Install .NET 9 SDK**
   Download from: https://dotnet.microsoft.com/download/dotnet/9.0

3. **Clone Repository**
   \`\`\`bash
   git clone <repository-url>
   cd crismac-npa-axis
   \`\`\`

## Installation

Volta will automatically install the correct Node.js and pnpm versions.

\`\`\`bash
# Install frontend dependencies
pnpm run install:frontend

# Restore backend dependencies
cd apps/backend
dotnet restore
cd ../..
\`\`\`

## Running the Application

### Option 1: Separate Terminals (Recommended)

Terminal 1 - Backend:
\`\`\`bash
pnpm run backend:watch
\`\`\`

Terminal 2 - Frontend:
\`\`\`bash
pnpm run frontend:dev
\`\`\`

### Option 2: Individual Services

Frontend only:
\`\`\`bash
pnpm run frontend:dev
\`\`\`

Backend only:
\`\`\`bash
pnpm run backend:dev
\`\`\`

## Verification

1. **Frontend**: http://localhost:4200
2. **Backend API**: https://localhost:5001/api/health
3. **Swagger**: https://localhost:5001/swagger

## Next Steps

1. Review the architecture documentation: `docs/ARCHITECTURE.md`
2. Start coding in `apps/frontend/src/app`
3. Add API endpoints in `apps/backend/src/Crismac.Npa.Axis.Api/Controllers`
```

**Commit:**
```bash
git add .
git commit -m "docs: add architecture and getting started documentation"
```

---

## Step 11: Final Verification and Create Develop Branch

```bash
# Verify structure
tree -L 3 -I 'node_modules|bin|obj|dist'

# Create develop branch
git checkout -b develop

# Switch back to main
git checkout main

# Verify everything is committed
git status

# Push to remote (if configured)
# git remote add origin <repository-url>
# git push -u origin main develop
```

**Commit:**
```bash
# No changes to commit, just branch creation
git log --oneline --all --graph
```

---

## Final Structure

Your monorepo should now look like this:

```
crismac-npa-axis/
├── .vscode/
│   ├── extensions.json
│   ├── launch.json
│   ├── settings.json
│   └── tasks.json
├── apps/
│   ├── backend/
│   │   ├── src/
│   │   │   └── Crismac.Npa.Axis.Api/
│   │   │       ├── Controllers/
│   │   │       │   └── HealthController.cs
│   │   │       ├── Properties/
│   │   │       ├── appsettings.json
│   │   │       ├── appsettings.Development.json
│   │   │       ├── Crismac.Npa.Axis.Api.csproj
│   │   │       └── Program.cs
│   │   ├── .gitignore
│   │   ├── Crismac.Npa.Axis.sln
│   │   └── README.md
│   └── frontend/
│       ├── .angular/                # Angular cache
│       ├── .vscode/                 # VS Code settings (optional)
│       ├── node_modules/            # Dependencies
│       ├── projects/
│       │   ├── axis-masters-app/   # Main application
│       │   │   ├── public/         # Static assets (Angular 18+)
│       │   │   ├── src/
│       │   │   │   ├── app/
│       │   │   │   │   ├── app.component.ts
│       │   │   │   │   ├── app.component.html
│       │   │   │   │   ├── app.component.scss
│       │   │   │   │   ├── app.config.ts
│       │   │   │   │   └── app.routes.ts
│       │   │   │   ├── environments/
│       │   │   │   │   ├── environment.ts
│       │   │   │   │   └── environment.prod.ts
│       │   │   │   ├── index.html
│       │   │   │   ├── main.ts
│       │   │   │   └── styles.scss
│       │   │   ├── tsconfig.app.json
│       │   │   └── tsconfig.spec.json
│       │   └── shared-lib/         # Shared library
│       │       ├── src/
│       │       │   ├── lib/
│       │       │   │   ├── components/
│       │       │   │   │   └── index.ts
│       │       │   │   ├── services/
│       │       │   │   │   └── index.ts
│       │       │   │   ├── models/
│       │       │   │   │   └── index.ts
│       │       │   │   ├── guards/
│       │       │   │   │   └── index.ts
│       │       │   │   ├── interceptors/
│       │       │   │   │   └── index.ts
│       │       │   │   ├── directives/
│       │       │   │   │   └── index.ts
│       │       │   │   └── pipes/
│       │       │   │       └── index.ts
│       │       │   └── public-api.ts
│       │       ├── ng-package.json
│       │       ├── package.json
│       │       ├── README.md
│       │       ├── tsconfig.lib.json
│       │       ├── tsconfig.lib.prod.json
│       │       └── tsconfig.spec.json
│       ├── .editorconfig
│       ├── .gitignore
│       ├── angular.json             # Angular workspace config
│       ├── package.json
│       ├── README.md
│       └── tsconfig.json            # Base TypeScript config
├── docs/
│   ├── ARCHITECTURE.md
│   └── GETTING_STARTED.md
├── .editorconfig
├── .gitignore
├── .npmrc
├── package.json
├── pnpm-workspace.yaml
└── README.md
```

---

## Quick Reference Commands

| Task | Command |
|------|---------|
| Install dependencies | `pnpm run install:frontend` |
| Start frontend | `pnpm run frontend:dev` |
| Start backend | `pnpm run backend:watch` |
| Build everything | `pnpm run build` |
| Clean | `pnpm run clean` |
| Health check | Visit https://localhost:5001/api/health |

---

## Next Steps

1. **Configure Database**: Update connection strings in `apps/backend/src/Crismac.Npa.Axis.Api/appsettings.json`
2. **Add Authentication**: Implement JWT authentication in both frontend and backend
3. **Create Features**: Start building your application features
4. **Setup CI/CD**: Configure your deployment pipeline
5. **Add Testing**: Set up unit and integration tests

---

## Git Commit History

Your git history should show these commits in order:

1. `chore: initialize repository with volta`
2. `chore: configure pnpm workspace`
3. `chore: create monorepo directory structure`
4. `feat: setup Angular frontend workspace with standalone architecture`
5. `feat: create shared library structure for code reusability`
6. `feat: setup .NET 9 Web API with CORS and Swagger`
7. `chore: configure root workspace scripts`
8. `chore: configure VS Code workspace settings`
9. `chore: add editorconfig for consistent code formatting`
10. `docs: add architecture and getting started documentation`

---

## Support

For issues or questions:
1. Check `docs/ARCHITECTURE.md`
2. Review `docs/GETTING_STARTED.md`
3. Contact your team lead
