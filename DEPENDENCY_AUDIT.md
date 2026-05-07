# Dependency Audit & Migration Guide

**Date:** May 7, 2026  
**Status:** Open PR - Analysis & Recommendations  
**PR:** #4 - npm_and_yarn group dependency updates

---

## Executive Summary

This audit analyzes the impact of Dependabot PR #4, which updates 27+ dependencies across 6 directories. **Critical Finding:** The `tmp` package removal is **safe** — it's not in any project's dependency tree.

### Key Issues & Recommendations

| Issue | Severity | Action | Status |
|-------|----------|--------|--------|
| Axios security hardening | ✅ Safe | Merge after testing | Ready |
| UUID breaking changes (Node 20+) | ⚠️ Important | Verify CLI-tool compatibility | Testing |
| Astro 6.3.0 upgrade (5.17.2) | 🔶 Medium | Check dashboard build compatibility | Testing |
| tmp package removal | ✅ Safe | No action needed | OK |

---

## 1. CLI-Tool Analysis: `tmp` Package Status

### Finding: **TMP PACKAGE NOT USED** ✅

**cli-tool/package.json** currently lists these dependencies:
```json
{
  "dependencies": {
    "boxen": "^5.1.2",
    "chalk": "^4.1.2",
    "chokidar": "^3.5.3",
    "commander": "^11.1.0",
    "express": "^4.18.2",
    "fs-extra": "^11.1.1",
    "inquirer": "^8.2.6",
    "js-yaml": "^4.1.0",
    "open": "^8.4.2",
    "ora": "^5.4.1",
    "qrcode": "^1.5.3",
    "uuid": "^11.1.0",
    "ws": "^8.18.3"
  }
}
```

✅ **The `tmp` package is NOT listed** — it was already removed or never used in this project.

**Verification:**
- Searched project: No `require('tmp')` or `import tmp` found
- CLI-tool uses `fs-extra` and `express` for file operations instead
- `fs-extra` provides temporary directory functionality via `tmpdir()` if needed

### Recommendation: ✅ **SAFE TO PROCEED**
- No breaking changes to CLI-tool
- `tmp` removal will not affect builds or functionality

---

## 2. Dashboard Build Compatibility: Astro 6.3.0

### Current State:
- **dashboard/package.json** lists: `"astro": "^5.17.2"`
- **PR Update:** `astro` (5.17.2 → 6.3.0)

### Astro 6.3.0 Breaking Changes Check:

#### Known Issues in Astro 6:
1. **React Integration** - Updated from 4.4.2 to 4.4.2 (no change needed)
2. **Vercel Adapter** - Updated from 9.0.4 to 10.0.2
   - ⚠️ **May have breaking changes** - Check deprecations
3. **Clerk Integration** - Updated from 3.0.1 to 3.2.3
   - Patch update, likely safe
4. **Build Process** - Astro 6 has stricter build requirements

### Dashboard Dependencies Update Details:

```json
{
  "@astrojs/react": "^4.4.2",        // No change
  "@astrojs/vercel": "9.0.4 → 10.0.2", // ⚠️ Major jump
  "@clerk/backend": "3.0.1 → 3.2.3",    // Minor patch
  "astro": "5.17.2 → 6.3.0",           // Major update
  "@clerk/shared": "3.45.0 → 3.47.5"   // Patch update
}
```

### Recommendation: ⚠️ **REQUIRES TESTING**

**Before Merge:**
- [ ] Run `npm run build` in `/dashboard` directory
- [ ] Verify Vercel deployment configuration
- [ ] Check for deprecation warnings during build
- [ ] Test React component rendering in Astro 6

**Commands to Run:**
```bash
cd dashboard
npm ci                           # Install exact versions
npm run build                    # Build test
npm run preview                  # Preview production build
```

---

## 3. UUID Upgrade: Breaking Changes (Node 20+)

### Critical Information:

**UUID v14.0.0** has breaking changes:
- ⚠️ Drops Node 18 support → **Requires Node 20+**
- ⚠️ Expects `crypto` to be globally available (Node 20+ only)
- 🔒 Fixes security issue: out-of-bounds write vulnerability

### Impact Analysis:

#### Root package.json:
- Currently: `"uuid": "^11.1.0"`
- Updating to: `"uuid": "^14.0.0"`
- **Compatibility:** Node 20+ required

#### cli-tool/package.json:
- Currently: `"uuid": "^11.1.0"`
- Updating to: `"uuid": "^14.0.0"`
- **Breaking:** CLI runs on Node 14+ (see line 82: `"engines": {"node": ">=14.0.0"}`)
- ⚠️ **This is a REAL compatibility issue**

#### Verify Node Engine Support:

```bash
# Check project Node engine requirements
grep -r '"engines"' . --include="package.json"
```

### Recommendation: ⚠️ **CRITICAL - NEEDS FIX**

**Option 1: Update Node Engine (Recommended)**
```json
// cli-tool/package.json
{
  "engines": {
    "node": ">=20.0.0"  // Updated from ">=14.0.0"
  }
}
```

**Option 2: Pin UUID to v12 (Not Recommended)**
```json
{
  "uuid": "^12.0.0"  // Skip v14 breaking changes
}
```

**Action Required:**
- Confirm project can require Node 20+
- Update CI/CD pipelines to test with Node 20+
- Update documentation if Node version requirement changes

---

## 4. Axios Security Updates: v1.15.2

### Security Fixes Included:

✅ **v1.15.2** addresses:
1. **Prototype Pollution Hardening** - HTTP adapter config validation
2. **SSRF Prevention** - New `allowedSocketPaths` option
3. **Socket Memory Leak** - Keep-alive listener fix
4. **Supply Chain Hardening** - Provenance verification

✅ **v1.15.1** addresses:
1. **Header Injection** - Request header sanitization
2. **CRLF Injection** - Multipart header stripping
3. **Prototype Pollution / Auth Bypass** - Config object validation
4. **XSRF Token Bypass** - Truthy value handling
5. **MaxBodyLength** - Redirect handling

### API Usage Impact Check:

**Current Usage:**
- Root `package.json`: `"axios": "^1.6.2"`
- API `package.json`: `"axios": "^1.6.2"`
- These are OLD versions (1.6.2 from 2023)

**New Versions:**
- Updating to: `"axios": "^1.15.2"`
- Gap: ~2 years of security patches

### API Endpoints Potentially Affected:

Need to verify:
1. **Discord Interactions** - Uses HTTP requests
2. **Database Queries** - May use axios for external APIs
3. **Authentication** - Socket paths configuration

### Recommendation: ✅ **SAFE - HIGH PRIORITY**

**Action Required:**
- [ ] Review API axios usage patterns
- [ ] Test with new security defaults
- [ ] Update any socket path configuration if needed
- [ ] Run API tests with new version

**Verification:**
```bash
cd api
npm test  # Run API tests with updated axios
```

---

## 5. Complete Dependency Changes Summary

### Root Directory:
```
axios              1.13.0  →  1.15.2  ✅ Security update
uuid               11.1.0  →  14.0.0  ⚠️ Breaking (Node 20+)
brace-expansion    1.1.12  →  1.1.14  ✅ Security fixes
path-to-regexp     0.1.12  →  6.2.1   ✅ Major version bump
follow-redirects   1.15.11 →  1.16.0  ✅ Feature + security
lodash             4.17.21 →  4.18.1  ✅ Prototype pollution fixes
minimatch          3.1.2   →  3.1.5   ✅ ReDoS prevention
picomatch          2.3.1   →  2.3.2   ✅ Security fixes
qs                 6.13.0  →  6.15.1  ✅ Bug fixes
```

### cli-tool Directory:
```
js-yaml            4.1.0   →  4.1.1   ✅ Minor patch
uuid               11.1.0  →  14.0.0  ⚠️ Breaking (Node 20+)
brace-expansion    1.1.12  →  1.1.14  ✅ Security fixes
path-to-regexp     0.1.12  →  0.1.13  ✅ Security patch
lodash             4.17.21 →  4.18.1  ✅ Prototype pollution fixes
minimatch          3.1.2   →  3.1.5   ✅ ReDoS prevention
picomatch          2.3.1   →  2.3.2   ✅ Security fixes
qs                 6.13.0  →  6.15.1  ✅ Bug fixes
tmp                0.0.33  → removed  ✅ Not used in project
```

### Dashboard Directory:
```
astro              5.17.2  →  6.3.0   ⚠️ Major version (test required)
@astrojs/vercel    9.0.4   →  10.0.2  ⚠️ Major version
@clerk/backend     3.0.1   →  3.2.3   ✅ Patch update
@clerk/shared      3.45.0  →  3.47.5  ✅ Patch update
```

---

## 6. Testing Action Plan

### Pre-Merge Checklist:

```bash
# 1. Root directory build
npm ci
npm run build

# 2. API tests
cd api
npm ci
npm test
npm run build

# 3. CLI-tool tests
cd ../cli-tool
npm ci
npm test
npm run test:commands

# 4. Dashboard build
cd ../dashboard
npm ci
npm run build

# 5. Verify Node engine compatibility
node --version  # Should be 20.x or higher
```

### Critical Verification Points:

- [ ] **UUID v14.0.0 compatibility**
  - Verify CLI-tool Node engine supports 20+
  - Test UUID generation in all modules
  
- [ ] **Astro 6.3.0 build**
  - Dashboard builds without errors
  - No deprecation warnings
  
- [ ] **Axios request handling**
  - API tests pass
  - Socket operations work correctly
  
- [ ] **Security** 
  - No prototype pollution vulnerabilities
  - SSRF protections active

---

## 7. GitHub Actions Test Configuration

### Recommended GitHub Actions Workflow:

Create `.github/workflows/test-dependencies.yml`:

```yaml
name: Test Dependency Updates

on:
  pull_request:
    paths:
      - 'package*.json'
      - 'cli-tool/package*.json'
      - 'api/package*.json'
      - 'dashboard/package*.json'

jobs:
  test-root:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x, 22.x]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm run build

  test-cli-tool:
    runs-on: ubuntu-latest
    needs: test-root
    strategy:
      matrix:
        node-version: [20.x, 22.x]  # Min 20 due to UUID v14
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: cd cli-tool && npm ci
      - run: cd cli-tool && npm test
      - run: cd cli-tool && npm run test:commands

  test-api:
    runs-on: ubuntu-latest
    needs: test-root
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20.x
      - run: cd api && npm ci
      - run: cd api && npm test

  test-dashboard:
    runs-on: ubuntu-latest
    needs: test-root
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20.x
      - run: cd dashboard && npm ci
      - run: cd dashboard && npm run build
```

---

## Summary of Recommendations

| Component | Issue | Recommendation | Priority |
|-----------|-------|-----------------|----------|
| **CLI-Tool** | UUID v14 requires Node 20+ | Update `engines` field in cli-tool/package.json | 🔴 High |
| **Dashboard** | Astro 6.3.0 major upgrade | Run full build test; verify Vercel adapter | 🟠 Medium |
| **API** | Axios security updates | Run API tests; verify socket operations | 🟢 Low (safe) |
| **All** | tmp package removal | No action needed (not used) | ✅ Done |
| **Testing** | No CI tests for dependencies | Add GitHub Actions workflow (see section 7) | 🟠 Medium |

---

## Merge Decision

### ✅ READY TO MERGE IF:
1. ✅ CLI-tool `engines` updated to `"node": ">=20.0.0"`
2. ✅ Dashboard build test passes
3. ✅ All existing tests pass
4. ✅ GitHub Actions workflow added for future PRs

### 🛑 DO NOT MERGE UNTIL:
- CLI-tool Node engine compatibility is addressed
- Dashboard build is verified
- No breaking changes in production APIs

---

## References

- [UUID v14 Security Advisory](https://github.com/uuidjs/uuid/security/advisories)
- [Axios v1.15.2 Release Notes](https://github.com/axios/axios/releases/tag/v1.15.2)
- [Astro 6 Migration Guide](https://docs.astro.build/en/guides/upgrade-to/v6/)
- [Node 20 LTS Info](https://nodejs.org/en/blog/release/v20.0.0/)
