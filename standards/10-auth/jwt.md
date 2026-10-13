# Authentication Standard: JWT Bearer Authentication (Tier 1)

## 1. Purpose
Defines token formats, cryptographic validation, guard middleware, public route exceptions, and request user enrichment for stateless JWT authentication.

---

## 2. Applies when
- `auth: jwt` is selected in Step 10.

---

## 3. Interaction Notes

| Preceding Decision | Impact on JWT Authentication |
|:---|:---|
| Step 2: `nestjs` | Implements NestJS `AuthGuard('jwt')` using `@nestjs/passport` + `passport-jwt`. |
| Step 2: `express` | Implements Express authentication middleware verifying JWT signatures with `jsonwebtoken`. |
| Step 9: `typed-config` | Injects `JWT_SECRET` (min 32 chars) and `JWT_EXPIRES_IN`. |

---

## 4. Contract

### Header Format & Extraction
- Protected endpoints MUST require an `Authorization` HTTP header with format: `Bearer <token>`.
- If the header is missing, malformed, expired, or cryptographically invalid, the request MUST terminate immediately with HTTP 401 Unauthorized using error code `UNAUTHORIZED` or `INVALID_TOKEN`.

### Standard Token Payload Claims
- The JWT payload MUST adhere to this minimum claim structure:
  - `sub`: Unique subject identifier (user ID string).
  - `email`: User email address.
  - `roles`: Array of assigned role strings (e.g. `['admin', 'user']`).
  - `iat`: Issued-at epoch seconds.
  - `exp`: Expiration epoch seconds.

### Public Route Bypass & Context Enrichment
- Endpoints MUST be protected by default.
- Public endpoints MUST be explicitly decorated or marked (e.g. `@Public()` decorator in NestJS or route-level bypass flag in Express).
- Upon successful verification, the guard/middleware MUST attach the authenticated payload to the request object: `req.user: AuthenticatedUser`.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// NestJS JWT guard contract with @Public bypass
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private readonly reflector: Reflector) { super(); }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
      context.getHandler(),
      context.getClass()
    ]);
    return isPublic ? true : super.canActivate(context);
  }
}
```

---

## 6. Agent Output Checklist
- [ ] JWT authentication guard or middleware implemented.
- [ ] `@Public()` decorator or explicit public route list wired.
- [ ] `req.user` strongly typed via `AuthenticatedUser` interface.
- [ ] 401 error response matches the standard error envelope.

---

## 7. Common Mistakes
- **Open by Default:** Making endpoints public by default and requiring manual protection tags.
- **Weak Secrets:** Allowing short JWT secrets (< 32 chars) in configuration schemas.
- **Leaking Password Hashes in Tokens:** Storing password hashes or internal database columns in the JWT payload.
