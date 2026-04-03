# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React Native WebView is a community-maintained WebView component for React Native, supporting iOS, Android, macOS, and Windows. It supports both Old Architecture (Paper/Bridge) and New Architecture (Fabric/TurboModules).

## Common Commands

```bash
# Install dependencies
yarn install

# Build (transpile TS → JS into lib/)
yarn build

# Type-check and lint
yarn lint                # tsc --noEmit + eslint
yarn ci                  # same as lint, with CI=true

# Run tests
jest                     # unit tests
yarn test:windows        # Windows Appium/Selenium tests

# Run example app
yarn android             # Android emulator
yarn ios                 # iOS simulator (run pod install --project-directory=example/ios first)
yarn macos               # macOS (run pod install --project-directory=macos first)
yarn windows             # Windows

# Prepare package (types + build)
yarn prepare
```

## Architecture

### TypeScript Layer (`src/`)

- **`WebViewTypes.ts`** — All shared TypeScript interfaces and types
- **`RNCWebViewNativeComponent.ts`** — Codegen spec defining native view props and commands (goBack, goForward, reload, injectJavaScript, etc.)
- **`NativeRNCWebViewModule.ts`** — TurboModule spec for native module functions
- **`WebViewShared.tsx`** — Shared logic: origin whitelist validation, URL handling, default loading/error UI
- **Platform implementations** — `WebView.ios.tsx`, `WebView.android.tsx`, `WebView.windows.tsx`, `WebView.macos.tsx` each wrap the native component with platform-specific logic
- **`WebView.tsx`** — Fallback for unsupported platforms

### Native Layer

- **`apple/`** — Shared iOS/macOS Objective-C++ implementation (WKWebView). Both `ios/` and `macos/` Xcode projects reference this directory.
  - `RNCWebViewImpl.m` — Core WebKit integration
  - `RNCWebView.mm` — RCTViewComponentView for Fabric
  - `RNCWebViewManager.mm` — View manager for legacy arch
- **`android/`** — Java implementation extending android.webkit.WebView
  - Dual source sets: `src/newarch/` and `src/oldarch/` for architecture-specific ViewManager/Module
  - `RNCWebViewClient.java` — Navigation events
  - `RNCWebChromeClient.java` — JS dialogs, file pickers
- **`windows/`** — C++/C# implementation supporting both legacy WebView and WebView2 (Chromium)

### Event Flow

Native → Codegen bridge → `WebViewShared` processing (origin whitelist, etc.) → Component callbacks (`onNavigationStateChange`, `onMessage`, etc.)

Android uses `registerCallableModule('RNCWebViewMessagingModule', ...)` for bridged messaging.

### Codegen

Configured in `package.json` under `codegenConfig` with spec name `RNCWebViewSpec`. The `RNCWebViewNativeComponent.ts` and `NativeRNCWebViewModule.ts` files are the codegen source of truth.

### Build Output

`lib/` directory (gitignored) contains compiled JS and `.d.ts` files. The package exports from `lib/` but Metro resolves to `src/` via `main-internal` field during development.

## CI

- **CircleCI**: lint → publish (semantic-release, master only)
- **GitHub Actions**: Platform-specific builds (iOS, Android, macOS, Windows) run on push to master and PRs. Android and iOS test both old and new architecture.
