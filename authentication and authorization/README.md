## Authentication vs Authorization

- Authentication: proving who the caller is (identity).
- Authorization: deciding what an authenticated caller can do (permissions/roles/policies).
- Both often ride on the same transport: the client presents proof (cookie, header, token), the server verifies and then enforces access rules.

---

## Sessions (stateful)

Use a server-side store to remember user state and send back only a session identifier.

Flow:

```
Login form -> Server verifies credentials -> Server creates session row {sessionId, userId, expires, data}
            -> Server sets cookie: Set-Cookie: sid=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
Subsequent requests -> Client sends Cookie: sid=abc123 -> Server looks up session -> applies authz -> responds
```

Pros:

- Easy to revoke (delete session server-side) and to implement per-device logout.
- Session data can hold arbitrary server-side state without enlarging cookies.

Cons:

- Requires shared/session store across servers (DB, Redis) for scaling.
- Store lookups add latency if not kept local.

Good practices:

- Short idle timeout (e.g., 15-60 minutes) plus absolute max age.
- Regenerate session ID after login or privilege elevation.
- HttpOnly + Secure + SameSite=Lax/Strict cookies; Path=/ to scope.
- Store server-side only the minimal data needed (user id, roles); avoid putting secrets in the client cookie.

---

## JWT (JSON Web Token, stateless)

Self-contained, signed token the server can verify without a lookup. Structure:

```
<base64url header>.<base64url payload>.<base64url signature>

Header: { "alg": "RS256", "typ": "JWT" }
Payload (claims): { "sub": "user123", "exp": 1737052800, "iat": 1737049200, "scope": "read:orders" }
Signature: sign(header + "." + payload)
```

Flow:

```
Client authenticates -> Server issues JWT (short-lived access token)
Client sends Authorization: Bearer <token>
Server verifies signature, exp, audience/issuer/scope -> enforces authz
```

Pros:

- No per-request DB/session lookup; good for distributed services.
- Claims travel with the token (who, when, scopes).

Cons:

- Harder to revoke until expiration; must rely on short TTLs or a blocklist.
- Larger than a session id; must protect from leakage.

Safety tips:

- Keep access tokens short-lived (e.g., 5-15 minutes). Pair with refresh tokens that rotate on every use.
- Use asymmetric signing (RS256/ES256) so verification keys can be public.
- Validate issuer, audience, expiration, not-before, and required scopes/roles.
- Do not store secrets or PII in claims; anyone with the token can read the payload.
- Avoid localStorage for browser tokens; prefer HttpOnly, Secure, SameSite cookies or an in-memory store with a backend session.

---

## Cookies

Browser mechanism to persist and send key/value pairs automatically.

Setting:

```
Set-Cookie: sid=abc123; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=1800
```

Key attributes:

- HttpOnly: hides from JavaScript; mitigates XSS token theft.
- Secure: only sent over HTTPS.
- SameSite: Strict (no cross-site), Lax (sends on top-level GET), None (cross-site; requires Secure).
- Path/Domain: scope the cookie to reduce exposure.
- Max-Age/Expires: lifetime control; set short for sensitive cookies.

Usage guidance:

- For browser auth, prefer HttpOnly Secure cookies over localStorage tokens.
- Avoid putting large data in cookies; send only opaque identifiers or very small tokens.

---

## Types and When to Use Them

1. Stateful sessions (cookie + server store)
   - Best for web apps where logout/revocation matters and a shared store is fine.
2. Stateless tokens (JWT access tokens)
   - Best when many services need to verify auth without a central session lookup; pair with short TTL and refresh tokens.
3. API Key
   - Simple per-client secret for service-to-service or low-risk integrations; rotate regularly; scope via key metadata.
4. OAuth 2.0 (delegated auth)
   - Use Authorization Code + PKCE for browser/mobile clients; Client Credentials for service-to-service; relies on an authorization server/IdP.

Selection hints:

- Start with server sessions for most web apps; move to JWT only if you need stateless verification across services.
- For APIs consumed by third parties, issue scoped API keys or OAuth clients, not user passwords.
- Always layer authorization checks (roles, permissions, ABAC/RBAC) after authentication, regardless of mechanism.
