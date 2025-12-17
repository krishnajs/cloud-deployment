# Sample Web UI - Code Walkthrough

## Overview
The Sample Web UI is a Single Page Application (SPA) built with Angular (and previously React components). It provides a reference implementation for managing devices, profiles, and domains via the Open AMT APIs.

## Key Files & Entry Points

### 1. Entry Point: `src/main.ts`
- **Function**: `bootstrapApplication(AppComponent)`
- **Responsibilities**:
  - Bootstraps the Angular application.
  - Configures global providers:
    - `provideRouter(routes)`: Sets up client-side routing.
    - `provideHttpClient`: Configures the HTTP client with interceptors.
    - `OAuthService`: Configures OIDC authentication.
    - `TranslateService`: Sets up i18n.

### 2. Routing: `src/app/routes.ts`
- Defines the navigation paths (e.g., `/devices`, `/profiles`, `/domains`).
- Maps paths to standalone Angular components.
- Applies `AuthGuard` to protected routes.

### 3. API Integration: `src/app/core/services/` (implied)
- Services interact with the RPS and MPS APIs via the Kong Gateway.
- `provideHttpClient` in `main.ts` includes `authorizationInterceptor` to attach JWT tokens to requests.

### 4. KVM & Redirection
- The UI includes components (likely using `ui-toolkit` libraries) to handle the KVM (Keyboard, Video, Mouse) redirection via WebSockets.
- Connects to the MPS WebSocket endpoint for the video stream.

## Startup Flow
1.  **Bootstrap**: Angular loads `main.ts`.
2.  **Auth Check**: `AuthGuard` checks if the user is logged in via OIDC. If not, redirects to the login provider.
3.  **Render**: Loads the `AppComponent` and the router outlet.
4.  **Data Fetch**: Components (like Device List) make HTTP calls to RPS/MPS to populate the UI.

## Key Technologies
- **Angular**: Core framework (Standalone Components).
- **RxJS**: Reactive programming for HTTP and event handling.
- **Angular Material**: UI component library.
- **angular-oauth2-oidc**: Authentication library.
