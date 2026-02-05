# Quick Start Guide

## For ox Maintainers

This repository demonstrates TypeScript compilation errors in ox@0.11.3 and ox@0.12.0.

### See the Issue Immediately

1. **Check GitHub Actions** (if this is a GitHub repository)
   - Go to the "Actions" tab
   - See the failing builds with TypeScript 5.8 and 5.9
   - See the passing builds with workarounds

2. **Or test locally:**
   ```bash
   npm install
   npm run build
   ```
   
   Expected output: **7 TypeScript errors** in ox source files

### The Errors

All 7 errors are in the `ox` package (not in the consuming code):

1. `Errors.ts:27` - Invalid `override` modifier
2. `Errors.ts:89` - Wrong number of arguments to `super()`
3. `abiParameters.ts:451` - Type mismatch in array encoding
4. `abiParameters.ts:762` - Type narrowing issue
5. `Secp256k1.ts:391` - Optional `yParity` property issue
6. `Signature.ts:288` - Optional `v` property issue
7. `SignatureEnvelope.ts:549` - Missing `prehash` property

### Verify the Workaround

Test that older ox version works:

```bash
npm install viem@2.28.4
npm run build
```

Expected output: **Success** (no errors)

### Environment

- **Fails with:**
  - TypeScript 5.9.3 + ox@0.11.3 ❌
  - TypeScript 5.8.3 + ox@0.11.3 ❌
  - TypeScript 5.7.3 + ox@0.11.3 ❌

- **Works with:**
  - TypeScript 5.9.3 + ox@0.6.9 ✅
  - TypeScript 5.4.5 + ox@0.11.3 ✅

## For Users Hitting This Issue

If you're experiencing this issue in your project:

### Quick Fix

Downgrade viem to 2.28.4:

```json
{
  "dependencies": {
    "viem": "2.28.4"
  }
}
```

### Alternative Fix

Downgrade TypeScript to 5.4.x:

```json
{
  "devDependencies": {
    "typescript": "~5.4.5"
  }
}
```

### Track the Issue

Watch this repository or the ox GitHub issues for updates on when this is fixed.

## Files in This Repo

- `src/index.ts` - Minimal code that triggers the bug
- `package.json` - Dependencies (viem@2.45.1, typescript@5.9.3)
- `tsconfig.json` - Standard TypeScript config
- `.github/workflows/reproduce-bug.yml` - Automated CI that shows the issue
- `README.md` - Detailed error analysis
- `SHARE_WITH_OX_TEAM.md` - Instructions for reporting

## Questions?

See `README.md` for detailed error descriptions and analysis.
