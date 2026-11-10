# SafeCircle v4 fixes

This release fixes the backend TypeScript build errors seen with Express 5 route parameter typings.

## Fixed

- Added `src/utils/routeParam.ts` to normalize single-value route parameters safely.
- Updated check-in route IDs.
- Updated contact route IDs.
- Updated safety-session route IDs.
- Updated public share-token route parameter handling.

These changes address the `string | string[]` -> `string` TypeScript errors reported during the Docker backend build.

Frontend fixes from v3 are retained.
