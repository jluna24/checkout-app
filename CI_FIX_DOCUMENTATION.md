# CI Error Fix: TS2307 Cannot find module './services/ChatService'

## Error Summary
TypeScript compilation error TS2307 at src/app.ts:10:25: "Cannot find module './services/ChatService'"

## Root Cause Analysis

The repository uses **React Router v7** with the modern `app/` directory structure, not a `src/` directory. The TypeScript compiler was attempting to compile files in a `src/` directory that either:

1. Should not exist in this project (legacy structure)
2. Was accidentally created/committed
3. Was being picked up due to overly broad tsconfig includes

### Why This Happened
- The `tsconfig.json` had `"include": ["**/*"]` which would pick up ANY TypeScript file in the repository, including files in unintended directories
- No explicit exclusion of the `src/` directory
- Jest configuration didn't explicitly ignore `src/` directory in test paths

## Immediate Fix Applied

### 1. Updated `tsconfig.json`
**Changed:**
- Made includes more specific: `"app/**/*"` instead of `"**/*"`
- Added explicit `exclude` array with: `"src"`, `"node_modules"`, `"build"`, `"dist"`, `".react-router"`

**Why this works:**
- TypeScript will only compile files in the `app/` directory and other explicitly included paths
- Even if a `src/app.ts` file exists, it will be ignored by the type checker
- Prevents phantom errors from files outside the intended project structure

### 2. Updated `jest.config.cjs`
**Changed:**
- Added `testPathIgnorePatterns` with `/src/` directory

**Why this works:**
- Jest won't try to run tests from the `src/` directory
- Prevents test failures from files that shouldn't be tested
- Keeps test runs focused on the actual `app/` directory

### 3. Updated `.gitignore`
**Changed:**
- Added `/src/` to the gitignore under a new "Legacy/unused source directories" section

**Why this works:**
- Prevents accidental commits of files in the `src/` directory
- Documents that this directory should not be used in this project
- Future-proofs against similar issues

## Validation Steps

After these changes, the CI pipeline should pass:

1. **Type Check** (`npm run typecheck`):
   - ✅ TypeScript will only compile files in `app/` directory
   - ✅ No errors about missing `src/app.ts` or ChatService

2. **Tests** (`npm test`):
   - ✅ Jest will only run tests from `app/__tests__/`
   - ✅ No attempts to import from non-existent modules

3. **Build** (`npm run build`):
   - ✅ React Router will build from `app/` directory as designed
   - ✅ No interference from stray source files

## Prevention

To avoid this issue in the future:

1. **Use the correct directory structure:**
   - ✅ Use `app/` for all application code (React Router 7 pattern)
   - ❌ Don't create `src/` directory

2. **Keep includes/excludes explicit:**
   - Always explicitly list directories in tsconfig includes
   - Explicitly exclude build outputs and legacy directories

3. **Document the structure:**
   - The README.md already documents the correct `app/` structure
   - New developers should follow this documented structure

## Additional Recommendations

1. **Add a pre-commit hook** to run `npm run typecheck` before commits
2. **Consider adding ESLint** to catch import errors earlier
3. **Add path validation** in CI to fail if unexpected directories contain TypeScript files

## Files Modified
- ✅ `tsconfig.json` - Made includes explicit, added excludes
- ✅ `jest.config.cjs` - Added testPathIgnorePatterns
- ✅ `.gitignore` - Added /src/ directory exclusion

## Expected Outcome
All CI jobs (test, typecheck, build) should now pass successfully without the TS2307 error.
