# TypeScript Compilation Errors in ox@0.11.3 and ox@0.12.0

## Issue Summary

The `ox` package versions 0.11.3 and 0.12.0 have TypeScript compilation errors when compiled with TypeScript 5.7+, 5.8, and 5.9. These errors prevent projects using `viem` (which depends on `ox`) from building successfully.

## Quick Reproduction

### Local Testing

```bash
# Install dependencies
npm install

# Run TypeScript compilation (will fail with 7 errors)
npm run build
```

### Strict Mode (viem-recommended config)

[viem's TypeScript docs](https://viem.sh/docs/typescript) state: *"To ensure everything works correctly, make sure that your tsconfig.json has strict mode set to true"*.

With strict mode enabled, the **same 7 ox errors** occur:

```bash
npm run build:strict   # uses tsconfig.strict.json with "strict": true
```

This shows the issue is not related to strict mode—it's the ox package itself.

### GitHub Actions

This repository includes a GitHub Actions workflow that automatically demonstrates the issue across multiple TypeScript and viem versions:

- ❌ **TypeScript 5.9.3 + viem 2.45.1** (ox@0.11.3) - FAILS with 7 errors
- ❌ **Strict mode + viem 2.45.1** (ox@0.11.3) - FAILS with same 7 errors ([viem recommends strict](https://viem.sh/docs/typescript))
- ✅ **TypeScript 5.9.3 + viem 2.28.4** (ox@0.6.9) - WORKS
- ❌ **TypeScript 5.8.3 + viem 2.45.1** - FAILS
- ⚠️ **TypeScript 5.4.5 + viem 2.45.1** - Has errors (ox still buggy)

The workflow runs automatically on push and can be triggered manually. Check the "Actions" tab after pushing to GitHub to see the results.

## Environment

- **TypeScript**: 5.9.3 (also fails with 5.8.3 and 5.7.3)
- **viem**: 2.45.1 (depends on ox@0.11.3)
- **ox**: 0.11.3 (via viem dependency)
- **Node**: 20.x or 22.x

## Compilation Errors

Running `npm run build` produces 7 TypeScript errors in the `ox` package:

### Error 1: TS4113 in `Errors.ts:27`
```
error TS4113: This member cannot have an 'override' modifier because it is not declared in the base class 'Error'.

27   override cause: cause
              ~~~~~
```

**Issue**: The `override` keyword is used on a property that doesn't exist in the base `Error` class.

### Error 2: TS2554 in `Errors.ts:89`
```
error TS2554: Expected 0-1 arguments, but got 2.

89     super(message, options.cause ? { cause: options.cause } : undefined)
                      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

**Issue**: The `Error` constructor in TypeScript's lib.es2022.error.d.ts only accepts 0-1 arguments, but 2 are being passed.

### Error 3: TS2345 in `abiParameters.ts:451`
```
error TS2345: Argument of type 'string | number | bigint | boolean | ...' is not assignable to parameter of type 'readonly (readonly ...)[]'.
Type 'string' is not assignable to type 'readonly (readonly ...)[]'.

451     return encodeArray(value, {
                           ~~~~~
```

**Issue**: Type mismatch in ABI parameter encoding - primitive types not assignable to nested readonly array types.

### Error 4: TS2322 in `abiParameters.ts:762`
```
error TS2322: Type 'readonly unknown[]' is not assignable to type 'readonly (readonly ...)[]'.
Type '{}' is missing the following properties from type 'readonly (readonly ...)[]': length, concat, join, slice, and 19 more.

762       value: (value as any)[index!] as readonly unknown[],
          ~~~~~
```

**Issue**: Type narrowing issue with nested array types in ABI parameter handling.

### Error 5: TS2322 in `Secp256k1.ts:391`
```
error TS2322: Type '{ r: bigint; s: bigint; yParity: number; } | { r: bigint; s: bigint; yParity?: number; }' is not assignable to type '{ r: bigint; s: bigint; yParity: number; }'.
Property 'yParity' is optional in type '{ r: bigint; s: bigint; yParity?: number; }' but required in type '{ r: bigint; s: bigint; yParity: number; }'.

391     return Address.isEqual(address, recoverAddress({ payload, signature }))
                                                                  ~~~~~~~~~
```

**Issue**: Union type with optional `yParity` property not assignable to type requiring `yParity`.

### Error 6: TS2345 in `Signature.ts:288`
```
error TS2345: Argument of type '{ r: bigint; s: bigint; yParity: number; v?: undefined; } | ...' is not assignable to parameter of type 'Legacy<bigint, number>'.
Property 'v' is optional in type '{ r: bigint; s: bigint; yParity: number; v?: undefined; }' but required in type 'Legacy<bigint, number>'.

288     if (signature.v) return fromLegacy(signature)
                                           ~~~~~~~~~
```

**Issue**: Signature type with optional `v` property not assignable to `Legacy` type requiring `v`.

### Error 7: TS2339 in `SignatureEnvelope.ts:549`
```
error TS2339: Property 'prehash' does not exist on type 'Secp256k1Flat<bigint, number> | UnionPartialBy<SignatureEnvelope<bigint, number>, "type" | "prehash">'.
Property 'prehash' does not exist on type 'Secp256k1Flat<bigint, number>'.

549     ...(type === 'p256' ? { prehash: value.prehash } : {}),
                                               ~~~~~~~
```

**Issue**: Property `prehash` doesn't exist on all union members.

## Why `skipLibCheck` Doesn't Help

The `tsconfig.json` has `skipLibCheck: true`, but this only skips checking `.d.ts` declaration files. TypeScript is compiling the source `.ts` files from the `ox` package, which bypasses `skipLibCheck`.

## Testing with Different Versions

### ox Versions
- ❌ **ox@0.11.3** (viem@2.45.1): 7 compilation errors
- ❌ **ox@0.12.0**: Same 7 compilation errors
- ✅ **ox@0.6.9** (viem@2.28.4): Compiles successfully

### TypeScript Versions
- ❌ **TypeScript 5.9.3**: 7 errors
- ❌ **TypeScript 5.8.3**: 7 errors  
- ❌ **TypeScript 5.7.3**: 9 errors (includes "Type instantiation is excessively deep")
- ❌ **TypeScript 5.6.3**: Debug Failure (internal TypeScript error)
- ❌ **TypeScript 5.5.4**: Debug Failure (internal TypeScript error)
- ✅ **TypeScript 5.4.5**: Compiles successfully

## Workaround

**The only reliable solution** is to downgrade `viem` to version 2.28.4 or earlier, which uses `ox@0.6.9`:

```json
{
  "dependencies": {
    "viem": "2.28.4"
  }
}
```

This works with **all TypeScript versions** including 5.9.3.

**Note:** Downgrading TypeScript to 5.4.x does NOT fully fix the issue - ox@0.11.3 still has compilation errors even with TypeScript 5.4.5.

## Impact

This issue affects any project using:
- `viem` >= 2.30.0 (which depends on ox >= 0.6.9 with breaking changes in 0.11.3)
- TypeScript >= 5.7.0

Projects cannot upgrade to newer versions of `viem` without:
- Staying on viem@2.28.4 or earlier (recommended - works with all TypeScript versions)

## Expected Fix

The `ox` package should:
1. Fix the `override` modifier usage in `Errors.ts`
2. Correct the `super()` call to match TypeScript's Error constructor signature
3. Fix type narrowing issues in ABI parameter encoding
4. Resolve optional property type mismatches in signature types
5. Ensure all properties exist on union type members

## Additional Context

- The `ox` package claims to support TypeScript >= 5.4.0 in its peer dependencies
- The errors occur in the `ox` source code itself, not in consuming code
- This is blocking TypeScript upgrades for projects using `viem`
- The issue appears to be related to stricter type checking in TypeScript 5.7+

## Related Links

- ox repository: https://github.com/wevm/ox
- viem repository: https://github.com/wevm/viem
- TypeScript 5.7 release notes: https://devblogs.microsoft.com/typescript/announcing-typescript-5-7/
