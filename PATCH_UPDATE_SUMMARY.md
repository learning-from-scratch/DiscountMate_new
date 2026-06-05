# Patch Update Summary

Date: 2026-06-04  
Branch context: `fix/update-packages`

## Overview

This patch updates dependency versions across the application, upgrades the Expo frontend to SDK 56, resolves upgrade-related runtime and build issues, and documents the final Tailwind/NativeWind compatibility decision.

The main app now passes Expo dependency checks and web export verification after the upgrade work.

## Frontend App

### Expo SDK and React Native stack

The main frontend in `Frontend/` was upgraded from the older Expo SDK 51-era stack to Expo SDK 56-compatible versions.

Key updated packages include:

- `expo` to `^56.0.8`
- `expo-router` to `~56.2.8`
- `react` to `19.2.3`
- `react-dom` to `19.2.3`
- `react-native` to `0.85.3`
- Expo modules such as `expo-asset`, `expo-constants`, `expo-font`, `expo-image-picker`, `expo-linking`, `expo-notifications`, `expo-splash-screen`, `expo-status-bar`, `expo-system-ui`, and `expo-web-browser`
- React Native packages such as `react-native-gesture-handler`, `react-native-reanimated`, `react-native-safe-area-context`, `react-native-screens`, `react-native-svg`, `react-native-web`, `react-native-webview`, and `react-native-worklets`

### Expo config changes

`Frontend/app.json` was updated for the new Expo SDK expectations.

The old top-level splash configuration was moved into the `expo-splash-screen` plugin configuration, which matches the newer Expo setup.

### React Navigation import migration

React Navigation imports that were previously pulled directly from React Navigation packages were updated to use Expo Router's React Navigation compatibility export where needed:

- `expo-router/react-navigation`

This avoids package/version mismatches after the Expo Router upgrade.

### NativeWind and Tailwind decision

Tailwind was tested on version 4, but this app's stable NativeWind runtime is not compatible with it.

Final supported styling stack:

- `nativewind` `^4.2.4`
- `react-native-css-interop` `^0.2.4`
- `tailwindcss` `^3.4.19`

Why Tailwind 4 was not kept:

- `react-native-css-interop@0.2.4` expects `tailwindcss ~3`.
- NativeWind 4 translates React Native `className` values into React Native styles through that interop package.
- With Tailwind 4, the app rendered with broken partial styling, including stretched bars and unstyled layout.
- NativeWind 5 preview is the Tailwind 4 path, but it caused Expo web static rendering failures in this project.

Final CSS config was restored to the stable NativeWind/Tailwind 3 setup:

- `Frontend/global.css` uses `@tailwind base`, `@tailwind components`, and `@tailwind utilities`.
- `Frontend/postcss.config.js` uses the `tailwindcss` PostCSS plugin.
- `Frontend/tailwind.config.js` includes the app/component content paths and `nativewind/preset`.

### Route warning cleanup

Several context/provider files were moved out of `Frontend/app/(tabs)/` because Expo Router treated them as routes and warned that they were missing default route component exports.

Moved into `Frontend/context/`:

- `AuthContext.tsx`
- `BasketContext.tsx`
- `CartContext.tsx`
- `ImageSearchContext.tsx`
- `ShoppingListsContext.tsx`
- `WishlistContext.tsx`

Imports were updated across the frontend to point to the new context folder.

### Profile form rendering fixes

`Frontend/components/profile/ProfileBasicInfoSection.tsx` was updated to address React Native Web text-node errors after the React/Expo upgrade.

The error was:

```text
Unexpected text node: . A text node cannot be a child of a <View>.
```

The affected JSX around the profile fields was adjusted so that stray text nodes are not rendered inside React Native `View` components.

### Styling warning cleanup

`Frontend/app/(tabs)/RecipeBot.tsx` was updated to replace deprecated web `shadow*` style props with `boxShadow` where applicable, while keeping native-friendly style properties such as `elevation`.

### Auth/session handling

Expired-token handling was tightened to reduce noisy console errors and clear invalid sessions more consistently.

Updated areas:

- `Frontend/utils/authSession.ts`
- `Frontend/context/ShoppingListsContext.tsx`
- `Frontend/context/UserProfileContext.tsx`

Expected expired-session errors such as `Token Has Expired` are now normalized and suppressed where appropriate. Users still need to log in again when their token expires.

## Backend

Backend Node dependencies in `Backend/package.json` and `Backend/package-lock.json` were updated.

Key packages now include:

- `express` `^5.2.1`
- `mongoose` `^9.6.3`
- `mongodb` `^7.2.0`
- `firebase` `^12.14.0`
- `firebase-admin` `^13.10.0`
- `helmet` `^8.2.0`
- `multer` `^2.1.1`
- `pg` `^8.21.0`
- `eslint` `^10.4.1`

An ESLint 10 flat config file was added:

- `Backend/eslint.config.mjs`

Backend CI checks were run successfully. Backend linting runs under the new config, but the codebase still contains many pre-existing lint/style findings that were not part of this dependency update patch.

## Python and Data Service Dependencies

Python dependency manifests were updated across backend services, ML services, scraping tools, and data engineering pipelines.

Updated files include:

- `Backend/analytics-service/requirements.txt`
- `Backend/ml-service/requirements.txt`
- `Catalogue_Scraping_2025/requirements.txt`
- `Scrapping/Australia_GroceriesScraper/requirements.txt`
- `ML/ReverseImageSearch/requirements.txt`
- `DE/etl-pipeline/pyproject.toml`
- `DE/ingestion-pipeline/pyproject.toml`
- `Archived/DE/db_init/requirements.txt`
- `Archived/DE/db_sample_create/requirements.txt`

## Docs Site

The Docusaurus docs site dependencies were updated in:

- `docs-site/package.json`
- `docs-site/package-lock.json`

Key packages now include:

- `@docusaurus/core` `^3.10.1`
- `@docusaurus/preset-classic` `^3.10.1`
- `@mdx-js/react` `^3.1.1`
- `react` `^19.2.7`
- `react-dom` `^19.2.7`

Docs image paths were also fixed after the build surfaced path issues. Windows-style paths such as `img\...` were changed to web-compatible `img/...` paths.

Updated docs include:

- `docs-site/docs/Deep Learning & AI/Catalogue YOLO OCR Pipeline.md`
- `docs-site/docs/Deep Learning & AI/LLM RAG Walkthrough.md`
- `docs-site/docs/Templates & Examples/How_to_Docusaurus.md`
- `docs-site/docs/Templates & Examples/documarkdown_copy_paste_template.md`

## Demo Product Page

The demo product page dependencies were updated in:

- `Frontend/demo-product page/package.json`
- `Frontend/demo-product page/package-lock.json`

The React entrypoint was migrated from the deprecated `ReactDOM.render` API to the React 18+ `createRoot` API:

- `Frontend/demo-product page/src/index.tsx`

A CSS module declaration file was added so TypeScript accepts CSS imports:

- `Frontend/demo-product page/src/global.d.ts`

## Verification Performed

The following checks passed during the patch:

- `Frontend`: `npx expo install --check`
- `Frontend`: `npx expo-doctor`
- `Frontend`: `npx expo export --platform web --clear`
- `Backend`: CI check command completed successfully
- `docs-site`: production build completed successfully
- `Frontend/demo-product page`: production build completed successfully

Additional package checks:

- `Backend`: `npm outdated --json` returned no outdated packages.
- `docs-site`: `npm outdated --json` returned no outdated packages.
- `Frontend/demo-product page`: `npm outdated --json` returned no outdated packages.
- `Frontend`: Expo-managed packages are compatible with SDK 56 according to Expo checks.

## Remaining Notes and Caveats

### Tailwind 4 is intentionally not used

Although Tailwind 4 is the latest major version, the main frontend currently stays on Tailwind 3.4.x because the stable NativeWind setup depends on Tailwind 3 behavior.

Do not update the main frontend to Tailwind 4 unless the app is also migrated to a stable NativeWind version that officially supports Tailwind 4.

### Some npm packages have newer registry versions

`npm outdated` still reports newer versions for some frontend packages, but Expo SDK compatibility is the higher-priority constraint for this React Native app.

Notable packages that should not be blindly updated:

- `tailwindcss`
- `react`
- `react-dom`
- React Native packages managed by Expo SDK 56

### Expired auth tokens still require login

The patch reduces noisy expired-token logs and normalizes session-expired handling, but it does not bypass authentication. Users with expired tokens still need to log in again.

