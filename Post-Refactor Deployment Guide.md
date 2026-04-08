# EWS FRM Monorepo - Shared Environment Library Architecture Post-Refactor Deployment Guide

**Status:** How Deployment Works AFTER Refactoring
**Date:** 2026-02-06
**Applies To:** Shared Environment Library Architecture

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [File Structure After Refactoring](#file-structure-after-refactoring)
3. [What Each File Contains](#what-each-file-contains)
4. [Development Workflow](#development-workflow)
5. [Build Process Explained](#build-process-explained)
6. [Deployment Scenarios](#deployment-scenarios)
7. [EWS vs FRMS Comparison](#ews-vs-frms-comparison)
8. [Command Reference](#command-reference)
9. [Troubleshooting](#troubleshooting)

---

## Overview

### Key Changes After Refactoring

**BEFORE (Current):**
```
❌ 6+ environment files with 85% duplication
❌ Build script replaces 3 files for FRMS
❌ Unclear which files are actually used
❌ Maintenance nightmare (update 6 files)
```

**AFTER (Refactored):**
```
✅ 3 shared config files (base + 2 overrides)
✅ Build script replaces 1 file only
✅ Crystal clear what's used when
✅ Update once, affects both apps
```

### Architecture Summary

```
Shared Configuration Library:
└── libs/environment-config/
    ├── base-environment.ts        # 85% shared config (API, keys, etc.)
    ├── ews-overrides.ts           # 15% EWS-specific (visualization, reports)
    └── frms-overrides.ts          # 15% FRMS-specific (SSO client ID)

Apps Import From Library:
├── EWS:  imports base + ews-overrides
└── FRMS: imports base + frms-overrides
```

---

## File Structure After Refactoring

### Complete Directory Structure

```
frontend/
│
├── libs/
│   └── environment-config/                          # ⭐ NEW: Shared configuration library
│       ├── src/
│       │   ├── lib/
│       │   │   ├── base-environment.interface.ts    # TypeScript interfaces
│       │   │   ├── base-environment.ts              # 85% shared config
│       │   │   ├── ews-overrides.ts                 # EWS-specific config
│       │   │   ├── frms-overrides.ts                # FRMS-specific config
│       │   │   └── index.ts                         # Barrel exports
│       │   └── index.ts                             # Public API
│       ├── ng-package.json
│       ├── package.json
│       ├── tsconfig.lib.json
│       └── README.md
│
├── src/                                             # EWS-APP
│   └── environments/
│       ├── environment.ts                           # ⚙️ Used by: ng serve (default)
│       ├── environment.dev.ts                       # ⚙️ Used by: ng serve --configuration=development
│       └── environment.prod.ts                      # ⚙️ Used by: ng build --configuration=production
│
├── projects/frms-app/                               # FRMS-APP
│   └── src/
│       └── environments/
│           ├── environment.ts                       # ⚙️ Used by: ng serve (default)
│           ├── environment.dev.ts                   # ⚙️ Used by: ng serve --configuration=development
│           └── environment.prod.ts                  # ⚙️ Used by: ng build --configuration=production
│
├── env-examples/                                    # Example deployment configs
│   ├── ews-internal.env.ts                         # Example: Internal EWS deployment
│   ├── frms-internal.env.ts                        # Example: Internal FRMS deployment
│   └── README.md
│
└── scripts/
    ├── build-app.js                                 # 🔧 Custom build script (simplified)
    └── README.md
```

### File Roles Summary

| File Location | Role | Used When | Can Commit? |
|--------------|------|-----------|-------------|
| **libs/environment-config/base-environment.ts** | Shared config (85%) | Always (imported by apps) | ✅ Yes |
| **libs/environment-config/ews-overrides.ts** | EWS-specific (15%) | When building EWS | ✅ Yes |
| **libs/environment-config/frms-overrides.ts** | FRMS-specific (15%) | When building FRMS | ✅ Yes |
| **src/environments/environment.dev.ts** | EWS development | Local EWS dev | ✅ Yes |
| **src/environments/environment.prod.ts** | EWS production target | EWS production build | ✅ Yes |
| **projects/frms-app/src/environments/environment.dev.ts** | FRMS development | Local FRMS dev | ✅ Yes |
| **projects/frms-app/src/environments/environment.prod.ts** | FRMS production target | FRMS production build | ✅ Yes |
| **External env files** (e.g., `c:\environments\*.ts`) | Client-specific overrides | Custom deployments | ❌ No (secure location) |

---

## What Each File Contains

### 1. Base Environment (Shared Configuration)

**File:** `libs/environment-config/src/lib/base-environment.ts`

**Contains:** 85% of configuration (shared by both apps)

```typescript
export const baseEnvironment = {
  // Backend API (SHARED)
  production: false,
  host: 'https://ews-enterprise.internal.d2ktech.dev/api/',

  // Security (SHARED)
  requestEncryptionEnabled: false,
  SiteKey: '6LcvoUgUAAAAAJJbhcXvLn3KgG-pyULLusaU4mL1',
  detailKey: '8080808080808080',

  // Authentication (SHARED)
  PasswordResetType: 'DefaultPassword',
  loginType: 'HFC',
  ssoEnabled: false,
};
```

**When to Edit:**
- ✅ Changing backend API URL
- ✅ Updating encryption keys
- ✅ Changing authentication settings
- ✅ Any config shared by both EWS and FRMS

**Impact:** Affects BOTH EWS and FRMS

---

### 2. EWS Overrides (EWS-Specific Configuration)

**File:** `libs/environment-config/src/lib/ews-overrides.ts`

**Contains:** 15% EWS-specific configuration

```typescript
export const ewsOverrides = {
  // EWS SSO Client
  ssoClientId: 'EWS-APP',

  // EWS-Specific Services (NOT used by FRMS)
  visualisationAssociatesUrl: 'http://localhost:5050/',
  visualisationBLFLLinkageUrl: 'http://localhost:5051/',
  reportUrl: 'http://192.168.1.10/EWS_MIG_REPORTS/ViewReport.aspx',

  // Optional: EWS login URL (for cross-app navigation)
  ewsAppLoginUrl: 'https://ews-enterprise.internal.d2ktech.dev/ews-app/',
};
```

**When to Edit:**
- ✅ Changing visualization service URLs
- ✅ Updating report server URL
- ✅ Changing EWS SSO client ID

**Impact:** Affects ONLY EWS

---

### 3. FRMS Overrides (FRMS-Specific Configuration)

**File:** `libs/environment-config/src/lib/frms-overrides.ts`

**Contains:** 15% FRMS-specific configuration

```typescript
export const frmsOverrides = {
  // FRMS SSO Client
  ssoClientId: 'FRMS-APP',

  // EWS Login URL (for SSO redirect)
  ewsAppLoginUrl: 'https://ews-enterprise.internal.d2ktech.dev/ews-app/',
};
```

**When to Edit:**
- ✅ Changing FRMS SSO client ID
- ✅ Updating EWS app URL for SSO redirect

**Impact:** Affects ONLY FRMS

---

### 4. App Environment Files (Consume Shared Config)

#### **EWS Environment Files**

**File:** `src/environments/environment.dev.ts`

```typescript
import { ewsEnvironmentDefaults, EWSEnvironment } from '@environment-config';

export const environment: EWSEnvironment = {
  ...ewsEnvironmentDefaults,  // ← Imports base + ews-overrides
  production: false,
};
```

**Used When:** Local EWS development (`ng serve ews-app --configuration=development`)

**File:** `src/environments/environment.prod.ts`

```typescript
import { ewsEnvironmentDefaults, EWSEnvironment } from '@environment-config';

export const environment: EWSEnvironment = {
  ...ewsEnvironmentDefaults,  // ← Imports base + ews-overrides
  production: true,
};
```

**Used When:**
- Production build (`ng build ews-app --configuration=production`)
- ⚠️ Gets REPLACED by build script if using custom environment file

---

#### **FRMS Environment Files**

**File:** `projects/frms-app/src/environments/environment.dev.ts`

```typescript
import { frmsEnvironmentDefaults, FRMSEnvironment } from '@environment-config';

export const environment: FRMSEnvironment = {
  ...frmsEnvironmentDefaults,  // ← Imports base + frms-overrides
  production: false,
};
```

**Used When:** Local FRMS development (`ng serve frms-app --configuration=development`)

**File:** `projects/frms-app/src/environments/environment.prod.ts`

```typescript
import { frmsEnvironmentDefaults, FRMSEnvironment } from '@environment-config';

export const environment: FRMSEnvironment = {
  ...frmsEnvironmentDefaults,  // ← Imports base + frms-overrides
  production: true,
};
```

**Used When:**
- Production build (`ng build frms-app --configuration=production`)
- ⚠️ Gets REPLACED by build script if using custom environment file

---

### 5. External Environment Files (Client-Specific)

**Location:** Outside repository (e.g., `c:\environments\` or secure storage)

**File:** `c:\environments\client-a-ews.env.ts`

```typescript
import { ewsEnvironmentDefaults, EWSEnvironment } from '@environment-config';

export const environment: EWSEnvironment = {
  ...ewsEnvironmentDefaults,      // ← Start with defaults

  // Override for Client A
  production: true,
  host: 'https://client-a.com/api/',
  requestEncryptionEnabled: true,
  SiteKey: 'CLIENT_A_SPECIFIC_KEY',
  detailKey: 'CLIENT_A_DETAIL_KEY',
  ssoEnabled: true,
  reportUrl: 'https://client-a.com/reports/',
};
```

**Used When:** Building for specific client with custom configuration

**Key Point:** Only need to override what's different from defaults!

---

## Development Workflow

### Scenario 1: Local Development

**Goal:** Run apps locally for development

#### **EWS Development:**

```bash
# 1. Start EWS app
pnpm start:ews-app:dev

# Or explicitly:
ng serve ews-app --configuration=development
```

**What Happens:**
```
1. Angular CLI reads: angular.json
2. Finds configuration: "development"
3. Replaces: environment.ts → environment.dev.ts
4. environment.dev.ts imports: base-environment + ews-overrides
5. App runs with: shared config + EWS-specific config
```

**Environment Used:**
```typescript
{
  ...baseEnvironment,      // From libs/environment-config/base-environment.ts
  ...ewsOverrides,         // From libs/environment-config/ews-overrides.ts
  production: false,       // From src/environments/environment.dev.ts
}
```

**API Calls Go To:** `https://ews-enterprise.internal.d2ktech.dev/api/`

---

#### **FRMS Development:**

```bash
# 1. Start FRMS app
pnpm start:frms-app:dev

# Or explicitly:
ng serve frms-app --configuration=development
```

**What Happens:**
```
1. Angular CLI reads: angular.json
2. Finds configuration: "development"
3. Replaces: environment.ts → environment.dev.ts
4. environment.dev.ts imports: base-environment + frms-overrides
5. App runs with: shared config + FRMS-specific config
```

**Environment Used:**
```typescript
{
  ...baseEnvironment,      // From libs/environment-config/base-environment.ts
  ...frmsOverrides,        // From libs/environment-config/frms-overrides.ts
  production: false,       // From projects/frms-app/src/environments/environment.dev.ts
}
```

**API Calls Go To:** `https://ews-enterprise.internal.d2ktech.dev/api/` (SAME as EWS!)

---

## Build Process Explained

### Scenario 2: Internal/QA Deployment

**Goal:** Build for internal testing/QA environment

#### **EWS Internal Build:**

```bash
# Command
pnpm build:ews-app:internal
```

**What Happens:**
```
Step 1: Build shared library
  └─ ng build environment-config
  └─ Output: dist/environment-config/

Step 2: Build EWS app with internal config
  └─ Reads: env-examples/ews-internal.env.ts
  └─ Temporarily replaces: src/environments/environment.prod.ts
  └─ Runs: ng build ews-app --configuration=production --base-href=/ews-app/
  └─ Angular replaces: environment.ts → environment.prod.ts
  └─ Restores: original environment.prod.ts

Step 3: Output
  └─ dist/ews-app/ (ready for deployment)
```

**Environment in Final Build:**
```typescript
// Content from env-examples/ews-internal.env.ts
{
  ...ewsEnvironmentDefaults,
  production: true,
  // Internal-specific overrides if any
}
```

**Files Replaced:** 1 file only (`src/environments/environment.prod.ts`)

**Output:** `dist/ews-app/`

---

#### **FRMS Internal Build:**

```bash
# Command
pnpm build:frms-app:internal
```

**What Happens:**
```
Step 1: Build shared library
  └─ ng build environment-config
  └─ Output: dist/environment-config/

Step 2: Build FRMS app with internal config
  └─ Reads: env-examples/frms-internal.env.ts
  └─ Temporarily replaces: projects/frms-app/src/environments/environment.prod.ts
  └─ Runs: ng build frms-app --configuration=production --base-href=/frms-app/
  └─ Angular replaces: environment.ts → environment.prod.ts
  └─ Restores: original environment.prod.ts

Step 3: Output
  └─ dist/frms-app/ (ready for deployment)
```

**Environment in Final Build:**
```typescript
// Content from env-examples/frms-internal.env.ts
{
  ...frmsEnvironmentDefaults,
  production: true,
  // Internal-specific overrides if any
}
```

**Files Replaced:** 1 file only (`projects/frms-app/src/environments/environment.prod.ts`)

**Output:** `dist/frms-app/`

**Key Difference from BEFORE:** ✅ No longer replaces EWS root environment files!

---

### Scenario 3: Client-Specific Production Deployment

**Goal:** Build for specific client with custom configuration

#### **EWS Client Production Build:**

```bash
# Command
node scripts/build-app.js \
  -a ews-app \
  -e c:/environments/client-a-ews.env.ts \
  -b /ews-app/ \
  -c production \
  -o c:/deploy/client-a/ews-app
```

**What Happens:**
```
Step 1: Validate inputs
  └─ App: ews-app ✓
  └─ Environment file: c:/environments/client-a-ews.env.ts ✓
  └─ Base href: /ews-app/ ✓
  └─ Configuration: production ✓

Step 2: Build shared library
  └─ pnpm build:environment-config

Step 3: Backup original environment
  └─ Backup: src/environments/environment.prod.ts → .backup

Step 4: Replace with custom environment
  └─ Copy: c:/environments/client-a-ews.env.ts
      → src/environments/environment.prod.ts

Step 5: Build application
  └─ pnpm build:ews-app --configuration=production --base-href=/ews-app/
  └─ Angular replaces: environment.ts → environment.prod.ts
  └─ Build output: dist/ews-app/

Step 6: Restore original environment
  └─ Restore: .backup → src/environments/environment.prod.ts
  └─ Delete: .backup

Step 7: Copy to custom output (if specified)
  └─ Copy: dist/ews-app/ → c:/deploy/client-a/ews-app/
```

**Environment in Final Build:**
```typescript
// Content from c:/environments/client-a-ews.env.ts
{
  ...ewsEnvironmentDefaults,  // Base + EWS overrides
  production: true,
  host: 'https://client-a.com/api/',           // Client-specific
  SiteKey: 'CLIENT_A_SPECIFIC_KEY',           // Client-specific
  detailKey: 'CLIENT_A_DETAIL_KEY',           // Client-specific
  ssoEnabled: true,                           // Client-specific
  reportUrl: 'https://client-a.com/reports/', // Client-specific
}
```

**Files Replaced:** 1 file (`src/environments/environment.prod.ts`)

**Output:** `c:/deploy/client-a/ews-app/`

---

#### **FRMS Client Production Build:**

```bash
# Command
node scripts/build-app.js \
  -a frms-app \
  -e c:/environments/client-a-frms.env.ts \
  -b /frms-app/ \
  -c production \
  -o c:/deploy/client-a/frms-app
```

**What Happens:**
```
Step 1-2: [Same validation and shared-lib build]

Step 3: Backup original environment
  └─ Backup: projects/frms-app/src/environments/environment.prod.ts → .backup

Step 4: Replace with custom environment
  └─ Copy: c:/environments/client-a-frms.env.ts
      → projects/frms-app/src/environments/environment.prod.ts

  ⭐ KEY CHANGE: No longer replaces EWS root environment files!

Step 5: Build application
  └─ pnpm build:frms-app --configuration=production --base-href=/frms-app/
  └─ Angular replaces: environment.ts → environment.prod.ts
  └─ Build output: dist/frms-app/

Step 6: Restore original environment
  └─ Restore: .backup → projects/frms-app/src/environments/environment.prod.ts
  └─ Delete: .backup

Step 7: Copy to custom output
  └─ Copy: dist/frms-app/ → c:/deploy/client-a/frms-app/
```

**Environment in Final Build:**
```typescript
// Content from c:/environments/client-a-frms.env.ts
{
  ...frmsEnvironmentDefaults,  // Base + FRMS overrides
  production: true,
  host: 'https://client-a.com/api/',          // Client-specific
  SiteKey: 'CLIENT_A_SPECIFIC_KEY',          // Client-specific
  ssoEnabled: true,                          // Client-specific
  ewsAppLoginUrl: 'https://client-a.com/ews-app/', // Client-specific
}
```

**Files Replaced:** 1 file (`projects/frms-app/src/environments/environment.prod.ts`)

**Output:** `c:/deploy/client-a/frms-app/`

**Key Improvement:** ✅ Only touches FRMS files, not EWS files!

---

## Deployment Scenarios

### Complete Deployment Matrix

| Scenario | Command | Environment File Used | Output Location | Files Replaced |
|----------|---------|----------------------|-----------------|----------------|
| **EWS Local Dev** | `pnpm start:ews-app:dev` | `environment.dev.ts` (imports base+ews) | N/A (dev server) | None (Angular replaces) |
| **FRMS Local Dev** | `pnpm start:frms-app:dev` | `environment.dev.ts` (imports base+frms) | N/A (dev server) | None (Angular replaces) |
| **EWS Internal** | `pnpm build:ews-app:internal` | `env-examples/ews-internal.env.ts` | `dist/ews-app/` | 1 (EWS prod.ts) |
| **FRMS Internal** | `pnpm build:frms-app:internal` | `env-examples/frms-internal.env.ts` | `dist/frms-app/` | 1 (FRMS prod.ts) |
| **EWS Client A** | `build-app.js -a ews-app -e client-a.env.ts` | External file | Custom output | 1 (EWS prod.ts) |
| **FRMS Client A** | `build-app.js -a frms-app -e client-a.env.ts` | External file | Custom output | 1 (FRMS prod.ts) |

---

## EWS vs FRMS Comparison

### Side-by-Side Build Flow

#### **EWS Build Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│                    EWS PRODUCTION BUILD                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Shared Library                                           │
│     └─ Build: libs/environment-config/                       │
│        └─ Output: dist/environment-config/                   │
│                                                              │
│  2. Environment Preparation                                  │
│     └─ Backup: src/environments/environment.prod.ts          │
│     └─ Replace with: External env file OR keep default       │
│                                                              │
│  3. Angular Build                                            │
│     └─ Command: ng build ews-app --configuration=production  │
│     └─ Angular replaces: environment.ts → environment.prod.ts│
│     └─ environment.prod.ts contains:                         │
│        ├─ baseEnvironment (from library)                     │
│        ├─ ewsOverrides (from library)                        │
│        └─ production: true                                   │
│                                                              │
│  4. Restore Original                                         │
│     └─ Restore: src/environments/environment.prod.ts         │
│                                                              │
│  5. Output                                                   │
│     └─ dist/ews-app/                                         │
│        ├─ index.html                                         │
│        ├─ main-[hash].js  ← Contains compiled config        │
│        ├─ styles-[hash].css                                  │
│        └─ assets/                                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

#### **FRMS Build Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│                   FRMS PRODUCTION BUILD                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Shared Library                                           │
│     └─ Build: libs/environment-config/                       │
│        └─ Output: dist/environment-config/                   │
│                                                              │
│  2. Environment Preparation                                  │
│     └─ Backup: projects/frms-app/src/environments/          │
│                environment.prod.ts                           │
│     └─ Replace with: External env file OR keep default       │
│     ⭐ KEY: No longer touches EWS root environment!          │
│                                                              │
│  3. Angular Build                                            │
│     └─ Command: ng build frms-app --configuration=production │
│     └─ Angular replaces: environment.ts → environment.prod.ts│
│     └─ environment.prod.ts contains:                         │
│        ├─ baseEnvironment (from library)                     │
│        ├─ frmsOverrides (from library)                       │
│        └─ production: true                                   │
│                                                              │
│  4. Restore Original                                         │
│     └─ Restore: projects/frms-app/src/environments/         │
│                 environment.prod.ts                          │
│                                                              │
│  5. Output                                                   │
│     └─ dist/frms-app/                                        │
│        ├─ index.html                                         │
│        ├─ main-[hash].js  ← Contains compiled config        │
│        ├─ styles-[hash].css                                  │
│        └─ assets/                                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Key Differences

| Aspect | EWS Build | FRMS Build |
|--------|-----------|------------|
| **Environment File Location** | `src/environments/` | `projects/frms-app/src/environments/` |
| **Config Imported From** | `base + ewsOverrides` | `base + frmsOverrides` |
| **Files Replaced (BEFORE refactor)** | 1 file (EWS env) | ❌ 3 files (FRMS + EWS + template) |
| **Files Replaced (AFTER refactor)** | 1 file (EWS env) | ✅ 1 file (FRMS env only) |
| **Touches Other App Files?** | No | ✅ No (AFTER refactor) |
| **Build Command** | `ng build ews-app` | `ng build frms-app` |
| **Output Directory** | `dist/ews-app/` | `dist/frms-app/` |

---

## Command Reference

### Quick Command Lookup

#### **Development Commands**

```bash
# EWS Development
pnpm start:ews-app:dev
# or
ng serve ews-app --configuration=development
# Environment: base + ews-overrides + dev settings
# API: https://ews-enterprise.internal.d2ktech.dev/api/

# FRMS Development
pnpm start:frms-app:dev
# or
ng serve frms-app --configuration=development
# Environment: base + frms-overrides + dev settings
# API: https://ews-enterprise.internal.d2ktech.dev/api/ (SAME!)
```

#### **Internal Build Commands**

```bash
# Build Shared Library (Required before app builds)
pnpm build:environment-config
# or
ng build environment-config
# Output: dist/environment-config/

# EWS Internal Build
pnpm build:ews-app:internal
# Environment: env-examples/ews-internal.env.ts
# Output: dist/ews-app/

# FRMS Internal Build
pnpm build:frms-app:internal
# Environment: env-examples/frms-internal.env.ts
# Output: dist/frms-app/
```

#### **Production Build Commands**

```bash
# EWS Production (with default config)
ng build ews-app --configuration=production --base-href=/ews-app/
# Environment: src/environments/environment.prod.ts (imports base+ews)
# Output: dist/ews-app/

# FRMS Production (with default config)
ng build frms-app --configuration=production --base-href=/frms-app/
# Environment: projects/frms-app/src/environments/environment.prod.ts
# Output: dist/frms-app/
```

#### **Custom Client Build Commands**

```bash
# EWS for Client A
node scripts/build-app.js \
  -a ews-app \
  -e c:/environments/client-a-ews.env.ts \
  -b /ews-app/ \
  -c production \
  -o c:/deploy/client-a/ews-app

# FRMS for Client A
node scripts/build-app.js \
  -a frms-app \
  -e c:/environments/client-a-frms.env.ts \
  -b /frms-app/ \
  -c production \
  -o c:/deploy/client-a/frms-app

# Both apps for Client A (sequential)
node scripts/build-app.js -a ews-app -e c:/environments/client-a-ews.env.ts -b /ews-app/ -c production -o c:/deploy/client-a/ews && \
node scripts/build-app.js -a frms-app -e c:/environments/client-a-frms.env.ts -b /frms-app/ -c production -o c:/deploy/client-a/frms
```

### Build Script Parameters

| Parameter | Description | Example | Required |
|-----------|-------------|---------|----------|
| `-a, --app` | App to build (`ews-app` or `frms-app`) | `-a ews-app` | ✅ Yes |
| `-e, --env-file` | Path to environment file | `-e c:/envs/prod.env.ts` | ✅ Yes |
| `-b, --base-path` | Base href for deployment | `-b /ews-app/` | ✅ Yes |
| `-c, --config` | Angular configuration | `-c production` | ⚠️ Optional (default: production) |
| `-o, --output` | Custom output directory | `-o c:/deploy/ews` | ⚠️ Optional |

---

## Troubleshooting

### Common Issues After Refactoring

#### **Issue 1: Build fails with "Cannot find module '@environment-config'"**

**Cause:** Shared library not built

**Solution:**
```bash
# Build the shared library first
pnpm build:environment-config

# Then build your app
pnpm build:ews-app
```

**Prevention:** Always build shared-lib before apps

---

#### **Issue 2: Wrong environment values in development**

**Symptom:** Local dev using wrong API URL

**Diagnosis:**
```bash
# Check which environment Angular is using
ng serve ews-app --configuration=development --verbose
```

**Solution:**
```bash
# Ensure using development configuration
pnpm start:ews-app:dev

# Or explicitly
ng serve ews-app --configuration=development
```

**Check:** `environment.dev.ts` imports correct overrides

---

#### **Issue 3: FRMS build still replaces 3 files**

**Cause:** Using old build script

**Solution:**
```bash
# Check build script version
head -20 scripts/build-app.js
# Should NOT have lines 117-158 (EWS root replacement)

# If old script, pull latest from git
git pull origin <branch>
```

**Verify:** FRMS build only replaces `projects/frms-app/src/environments/environment.prod.ts`

---

#### **Issue 4: Deployed app uses wrong configuration**

**Symptom:** Production app points to wrong API

**Diagnosis:**
```bash
# Check compiled environment in dist/
grep -r "https://.*api" dist/ews-app/main-*.js

# Should show the API URL from your environment file
```

**Solution:**
1. Verify external environment file has correct values
2. Rebuild with correct environment file
3. Check build script output for confirmation

---

#### **Issue 5: Type errors when importing from @environment-config**

**Symptom:** TypeScript errors about environment types

**Solution:**
```bash
# Rebuild shared library
pnpm build:environment-config

# Update tsconfig paths
# tsconfig.json should have:
{
  "compilerOptions": {
    "paths": {
      "@environment-config": ["dist/environment-config"]
    }
  }
}
```

---

### Verification Checklist

After any build, verify:

```bash
# ✅ Check 1: Build succeeded
[ -d "dist/ews-app" ] && echo "✅ Build output exists"

# ✅ Check 2: Environment file restored
git status src/environments/environment.prod.ts
# Should show: nothing to commit (if unchanged)

# ✅ Check 3: Correct API URL in build
grep -o "host:.*" dist/ews-app/main-*.js | head -1
# Should show expected API URL

# ✅ Check 4: Production flag set
grep "production.*true" dist/ews-app/main-*.js
# Should find production: true

# ✅ Check 5: App-specific config present
# For EWS: Check for visualization URLs
grep "visualisation" dist/ews-app/main-*.js
# For FRMS: Check for SSO client ID
grep "FRMS-APP" dist/frms-app/main-*.js
```

---

## Visual Summary

### Environment File Flow (After Refactoring)

```
┌──────────────────────────────────────────────────────────────────┐
│                    SHARED CONFIGURATION                          │
│                                                                  │
│  libs/environment-config/                                        │
│  ├── base-environment.ts          ← 85% shared config           │
│  │   (API, keys, auth, etc.)                                    │
│  │                                                               │
│  ├── ews-overrides.ts             ← 15% EWS-specific            │
│  │   (visualization, reports)                                   │
│  │                                                               │
│  └── frms-overrides.ts            ← 15% FRMS-specific           │
│      (SSO client ID, EWS URL)                                   │
│                                                                  │
└───────────────────┬──────────────────────────────────────────────┘
                    │
        ┌───────────┴────────────┐
        │                        │
        ▼                        ▼
┌───────────────┐        ┌───────────────┐
│   EWS  APP    │        │  FRMS  APP    │
├───────────────┤        ├───────────────┤
│ environment.  │        │ environment.  │
│ dev.ts        │        │ dev.ts        │
│   imports:    │        │   imports:    │
│   base +      │        │   base +      │
│   ews-over    │        │   frms-over   │
│               │        │               │
│ environment.  │        │ environment.  │
│ prod.ts       │        │ prod.ts       │
│   imports:    │        │   imports:    │
│   base +      │        │   base +      │
│   ews-over    │        │   frms-over   │
└───────────────┘        └───────────────┘
        │                        │
        │ Build Script           │ Build Script
        │ (replaces 1 file)      │ (replaces 1 file)
        ▼                        ▼
┌───────────────┐        ┌───────────────┐
│ dist/ews-app/ │        │ dist/frms-app/│
│               │        │               │
│ Compiled with │        │ Compiled with │
│ base +        │        │ base +        │
│ ews-overrides │        │ frms-overrides│
└───────────────┘        └───────────────┘
```

---

## Summary

### Key Takeaways After Refactoring

1. **✅ Single Source of Truth**
   - Edit `base-environment.ts` → affects both apps
   - No more updating 6 files for one change

2. **✅ Clear Separation**
   - `base-environment.ts` = shared (85%)
   - `ews-overrides.ts` = EWS-specific (15%)
   - `frms-overrides.ts` = FRMS-specific (15%)

3. **✅ Simplified Build**
   - FRMS build replaces 1 file (not 3!)
   - No cross-app file replacement
   - Clear, predictable behavior

4. **✅ Development Same**
   - Commands unchanged
   - Workflow unchanged
   - Just cleaner internally

5. **✅ Deployment Clearer**
   - External env files can extend defaults
   - Only override what's different
   - Type-safe, documented

---

**Questions?** Refer to:
- [PROPOSAL-ENVIRONMENT-ARCHITECTURE-REFACTOR.md](PROPOSAL-ENVIRONMENT-ARCHITECTURE-REFACTOR.md) - Complete implementation plan
- [ARCHITECTURE-ANALYSIS.md](ARCHITECTURE-ANALYSIS.md) - Technical details
- [EXECUTIVE-SUMMARY.md](EXECUTIVE-SUMMARY.md) - Quick reference

---

**END OF POST-REFACTOR DEPLOYMENT GUIDE**
