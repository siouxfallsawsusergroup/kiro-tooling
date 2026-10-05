---
inclusion: manual
name: auth-cognito-authguard
description: Optional advanced pattern for gating a React SPA with Amazon Cognito and react-oidc-context. Include with #auth-cognito-authguard when building that UI.
tags:
  - type/steering
  - aws/security
  - domain/authentication
---

# Optional pattern: Cognito AuthGuard for a React SPA

This is an optional, advanced pattern. Use it when a React single-page app should sign in with an Amazon Cognito user pool through OIDC. It is not a default for every project, and it is not a substitute for API authorization.

The browser guard stops protected UI from rendering without a token. APIs must validate the JWT themselves and enforce access. Anything the browser can see, including this guard, can be skipped by a client that calls the API directly.

## Shape

1. **AuthProvider** wraps the app and points at the user pool.
2. **AuthGuard** wraps protected routes and checks authentication, then group membership if you use groups.
3. **User context** (optional) exposes email, display name, and groups so the rest of the tree does not parse the token again.

## Dependencies

```json
{
  "oidc-client-ts": "^3.2.0",
  "react-oidc-context": "^3.2.0"
}
```

## App client

Create a separate Cognito app client per application. Do not reuse a client id across apps. Public SPAs use a client without a secret, with the authorization code grant and PKCE.

| Setting | Notes | Example |
| --- | --- | --- |
| `authority` | User pool issuer | `https://cognito-idp.{region}.amazonaws.com/{userPoolId}` |
| `client_id` | That app's client id | `{appClientId}` |
| `redirect_uri` | Must match an allowed callback URL | `window.location.origin + "/"` |
| `response_type` | Authorization code | `"code"` |
| `scope` | OIDC scopes the app needs | `"email openid phone profile"` |
| `identity_provider` | Optional. Skips the hosted UI chooser | `"{YourIdPName}"` |

In the Cognito console: User pool, App integration, App clients. Set callback and sign-out URLs to the real origins, grant type **Authorization code**, and the scopes above. Enable a federated IdP on the app client only if this app uses one.

## AuthProvider

```tsx
import { AuthProvider } from "react-oidc-context";

const cognitoAuthConfig = {
  authority: "https://cognito-idp.{region}.amazonaws.com/{userPoolId}",
  client_id: "{appClientId}",
  redirect_uri: window.location.origin + "/",
  response_type: "code",
  scope: "email openid phone profile",
  loadUserInfo: true,
  // Optional: skip the hosted UI and send the browser to one federated IdP.
  extraQueryParams: {
    identity_provider: "{YourIdPName}",
  },
  onSigninCallback: () => {
    window.history.replaceState({}, document.title, window.location.pathname);
  },
};

ReactDOM.createRoot(document.getElementById("root")!).render(
  <AuthProvider {...cognitoAuthConfig}>
    <App />
  </AuthProvider>
);
```

Leave `extraQueryParams` out when users should see the hosted UI.

## AuthGuard

The guard covers loading, error, signed-out (redirect), and signed-in (optional group check).

```tsx
import { useEffect } from "react";
import { useAuth } from "react-oidc-context";

const ALLOWED_GROUPS = ["app-admins", "app-users"];

/**
 * Cognito groups usually arrive as an array on `cognito:groups`.
 * A SAML mapping sometimes delivers `custom:groups` as a string.
 */
function getGroups(profile: Record<string, unknown>): string[] {
  const raw = profile["custom:groups"] ?? profile["cognito:groups"];
  if (!raw) return [];
  if (Array.isArray(raw)) return raw.map(String);
  if (typeof raw === "string") {
    const cleaned = raw.replace(/^\[|\]$/g, "");
    return cleaned.split(",").map((g) => g.trim()).filter(Boolean);
  }
  return [];
}

export function AuthGuard({ children }: { children: React.ReactNode }) {
  const auth = useAuth();

  useEffect(() => {
    if (
      !auth.isAuthenticated &&
      !auth.isLoading &&
      !auth.activeNavigator &&
      !auth.error
    ) {
      auth.signinRedirect();
    }
  }, [auth.isAuthenticated, auth.isLoading, auth.activeNavigator, auth.error]);

  if (auth.isLoading || !auth.isAuthenticated) {
    return auth.error ? (
      <AuthError message={auth.error.message} onRetry={() => auth.signinRedirect()} />
    ) : (
      <LoadingSpinner />
    );
  }

  const groups = getGroups(auth.user?.profile ?? {});
  const hasAccess = groups.some((g) => ALLOWED_GROUPS.includes(g));
  if (!hasAccess) {
    return <AccessDenied groups={groups} onSignOut={() => auth.removeUser()} />;
  }

  return <>{children}</>;
}
```

If the app does not use groups, skip the group check and render children once `auth.isAuthenticated` is true. An empty allow-list that denies everyone is a confusing default, so decide explicitly.

## Routes

Put protected routes inside the guard. Public routes stay outside it.

```tsx
import { BrowserRouter, Routes, Route } from "react-router";
import { AuthGuard } from "./components/AuthGuard";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route
          element={
            <AuthGuard>
              <YourLayout />
            </AuthGuard>
          }
        >
          <Route index element={<Dashboard />} />
          <Route path="settings" element={<Settings />} />
        </Route>
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

## User context

```tsx
import { createContext, useContext } from "react";
import { useAuth } from "react-oidc-context";

interface UserContextType {
  email: string;
  displayName: string;
  groups: string[];
  isAdmin: boolean;
}

const UserContext = createContext<UserContextType>({
  email: "",
  displayName: "",
  groups: [],
  isAdmin: false,
});

export function UserProvider({ children }: { children: React.ReactNode }) {
  const auth = useAuth();
  const profile = auth.user?.profile ?? {};
  const groups = getGroups(profile);

  const value: UserContextType = {
    email: (profile.email as string) ?? "",
    displayName:
      (profile.name as string) ?? (profile.preferred_username as string) ?? "",
    groups,
    isAdmin: groups.includes("app-admins"),
  };

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

export const useUser = () => useContext(UserContext);
```

## Checklist for a new app

- New app client, unique `client_id`, no client secret for a public SPA.
- Callback and sign-out URLs match the deployed origins, including localhost only if you need it.
- `ALLOWED_GROUPS` matches groups that actually exist in this pool, or the check is removed.
- `redirect_uri` matches one allowed callback.
- Federated IdP name is set only when you intend to skip the hosted UI.
- The API validates the JWT (issuer, audience, expiry, signature) and does its own authorization.
- If the app is hosted behind CloudFront as a single-page app, the distribution returns `index.html` for client-side routes.

## Security notes

1. The guard is client-side. It does not protect APIs.
2. The client id, user pool id, and group names ship in the JavaScript bundle. That is normal for a public OIDC client. Security is PKCE, redirect URI checks, and server-side token validation.
3. Read groups from `cognito:groups` or, when you map a federated IdP, from the claim you configured. Handle both a real array and a stringified list.
4. `oidc-client-ts` stores tokens in session storage by default. Know that XSS can read them. Keep dependencies patched and avoid injecting untrusted HTML.
5. Do not put the client secret of a confidential client into a SPA.

## Calling an API

```tsx
const auth = useAuth();
const accessToken = auth.user?.access_token;

fetch("/api/resource", {
  headers: { Authorization: `Bearer ${accessToken}` },
});
```

The API still checks the token. The header is not authorization by itself.

## Sign-out

```tsx
const auth = useAuth();

auth.removeUser();       // local session only
auth.signoutRedirect();  // Cognito logout endpoint as well
```

## Related

- [AWS security standards](../cloudformation/cfn-security-standards.md)
- [Network security baseline](../networking/net-security-baseline.md)
