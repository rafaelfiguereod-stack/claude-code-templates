# Pre-Merge Checklist for PR #4: npm_and_yarn Dependency Updates

## Status: 🟡 PENDING REVIEW

**Date Created:** May 7, 2026  
**PR Number:** #4  
**Focus:** npm_and_yarn group dependency updates  

---

## Critical Issues Found

### 🔴 **ISSUE 1: UUID v14 Requires Node 20+**

**Current State:**
- `cli-tool/package.json` line 82: `"engines": {"node": ">=14.0.0"}`
- UUID will update to `^14.0.0` (requires Node 20+)

**Action Required:**
```json
// cli-tool/package.json - UPDATE THIS:
{
  "engines": {
    "node": ">=20.0.0"  // Changed from ">=14.0.0"
  }
}
```

**Verification:**
```bash
cd cli-tool
node --version  # Must be 20.x or higher
npm ci
npm test
```

---

### ⚠️ **ISSUE 2: Dashboard Astro 6.3.0 Build**

**Updates:**
- astro: 5.17.2 → 6.3.0 (major)
- @astrojs/vercel: 9.0.4 → 10.0.2 (major)

**Required Test:**
```bash
cd dashboard
npm ci
npm run build
npm run preview  # Visual verification
```

**Success Criteria:**
- ✅ Build completes without errors
- ✅ No deprecation warnings
- ✅ Asset generation successful
- ✅ Preview renders correctly

---

### ✅ **CLEARED: `tmp` Package Removal**

**Status:** SAFE - Already verified as unused
- `tmp` was never listed in cli-tool dependencies
- Project uses `fs-extra` for file operations
- No code imports `tmp` module

---

### ✅ **CLEARED: Axios Security Updates**

**Status:** SAFE - High priority security fixes
- v1.15.2 includes prototype pollution fixes
- SSRF prevention improvements  
- No breaking API changes
- Requires: Run API tests

**Verification:**
```bash
cd api
npm test
```

---

## Pre-Merge Verification Steps

### Step 1: Update Node Engine Requirement ⚠️ **CRITICAL**

```bash
# Edit cli-tool/package.json
# Line 82: Change ">=14.0.0" to ">=20.0.0"
```

**Confirm:**
```bash
grep -A 1 '"engines"' cli-tool/package.json
# Should show: "node": ">=20.0.0"
```

### Step 2: Test CLI-Tool (Node 20+ Only)

```bash
# Use Node 20 or higher
node --version  # v20.x.x or v22.x.x

cd cli-tool
npm ci
npm test
npm run test:commands
npm run security-audit
```

### Step 3: Test API

```bash
cd api
npm ci
npm test
npm run test:api
npm audit --audit-level=moderate
```

### Step 4: Test Dashboard Build

```bash
cd dashboard
npm ci
npm run build
npm run preview

# Verify dist folder exists
ls -la dist/
```

### Step 5: Root Build Test

```bash
# Back to root
cd ../..
npm ci
npm run build
```

---

## Dependency Impact Summary

### Root package.json
- ✅ axios: 1.13.0 → 1.15.2 (security)
- ⚠️ uuid: 11.1.0 → 14.0.0 (breaking - Node 20+)
- ✅ Other updates: security patches only

### cli-tool/package.json
- ⚠️ uuid: 11.1.0 → 14.0.0 (breaking - **needs Node engine update**)
- ✅ js-yaml: 4.1.0 → 4.1.1 (patch)
- ✅ Other updates: security patches

### api/package.json
- ✅ axios: 1.6.2 → 1.15.2 (security)
- ✅ All other: security patches

### dashboard/package.json
- ⚠️ astro: 5.17.2 → 6.3.0 (major - **needs build test**)
- ⚠️ @astrojs/vercel: 9.0.4 → 10.0.2 (major)
- ✅ Other updates: safe patches

---

## Merge Decision Matrix

| Condition | Status | Action |
|-----------|--------|--------|
| CLI-tool Node engine updated to >=20 | ⬜ Pending | **REQUIRED** |
| Dashboard build passes | ⬜ Pending | **REQUIRED** |
| All tests passing | ⬜ Pending | **REQUIRED** |
| Security audit reviewed | ⬜ Pending | **REQUIRED** |
| CI/CD workflow added | ✅ Done | - |

---

## Sign-Off

### For Reviewers:

- [ ] Verify cli-tool/package.json engines field updated
- [ ] Confirm dashboard build test passing
- [ ] Review axios security fixes
- [ ] Check all CI tests passing
- [ ] Approve merge

### Acceptance Criteria:

```
✅ cli-tool/package.json: "engines": {"node": ">=20.0.0"}
✅ dashboard build: npm run build completes successfully
✅ All tests: npm test passes in all subdirectories
✅ Security: npm audit shows no vulnerabilities
✅ CI: GitHub Actions workflow passes all jobs
```

---

## Testing Timeline

**Recommended Order:**
1. Update cli-tool Node engine (5 min)
2. Run CLI-tool tests (10 min)
3. Run API tests (5 min)
4. Build dashboard (10 min)
5. Root build test (5 min)
6. Security review (5 min)
7. Merge approval (5 min)

**Total: ~45 minutes**

---

## Questions or Issues?

- 📖 See `DEPENDENCY_AUDIT.md` for detailed analysis
- 🔄 See `.github/workflows/test-dependencies.yml` for CI configuration
- 💬 Comment on PR #4 for discussion

---

**Last Updated:** 2026-05-07  
**Audit By:** Claude Copilot  
**Status:** Ready for Review ⏳
