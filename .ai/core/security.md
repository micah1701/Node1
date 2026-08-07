# Core Security

## JWT auth (`src/core/middlewares/auth.middleware.ts`)

```ts
authenticate(req, res, next)      // requires `Authorization: Bearer <token>`,
                                   // verifies via verifyToken(), attaches req.user
authorize(roles: string[])        // must run AFTER authenticate — checks req.user.role
```

`req.user: UserPayload = { id: string; email: string; role: string }`
(declared via Express namespace augmentation in `auth.middleware.ts`).

`authenticate` throws `401 Authentication required` if there's no bearer
token, and `401 Invalid authentication token` if verification fails.
`authorize` throws `401` if `req.user` is missing (i.e. `authenticate`
wasn't run first) and `403 Insufficient permissions` if the role doesn't
match.

## Encryption (`src/core/utils/encryption.ts`)

Three distinct modes — pick based on who needs to be able to decrypt:

| Mode | Functions | Who can decrypt |
|---|---|---|
| Master-key | `encryptWithMasterKey` / `decryptWithMasterKey` | This server (uses `config.encryption.masterKey`) |
| Passphrase | `encryptWithPassphrase` / `decryptWithPassphrase` | Only the holder of the passphrase — server cannot decrypt without it |
| Public-key | `encryptWithPublicKey` / `decryptWithPrivateKey` | Only the private-key holder — true E2EE |

All three use AES-256-CBC with a random 16-byte IV and PBKDF2 key
derivation (10,000 iterations, SHA-1) — IV is prepended to the ciphertext
and base64-encoded.

Public-key mode auto-detects the key format (`detectKeyType`): RSA PEM
(`-----BEGIN PUBLIC KEY-----`), SSH RSA (`ssh-rsa ...`, converted to PEM
internally), or X25519 (`x25519:<base64>`, NaCl box). **Ed25519 keys are
explicitly rejected for encryption** — it's a signature-only algorithm;
`encryptWithPublicKey` throws a clear error directing you to RSA or X25519
instead.

## SSH / asymmetric key generation

`generateSSHKeyPair(keyType: 'RSA2048' | 'RSA4096' | 'Ed25519' | 'X25519')`
→ `{ publicKey, privateKey, keyType, fingerprint, pemPublicKey? }`. RSA
keys get both SSH-wire-format and PEM public keys; fingerprint is
SHA-256/base64 of the SSH-format key.

## Request/audit logging (`src/core/middlewares/requestLogger.middleware.ts`)

Mounted globally at `config.endpointPrefix` in `src/index.ts` — runs on
every API request. For each request it:

1. Assigns a `requestUuid` (`req.requestUuid`), captures `startTime`.
2. Best-effort extracts a user id from the bearer token (swallows errors —
   never blocks the request on an invalid/missing token).
3. Redacts sensitive fields from headers and body (see `SENSITIVE_FIELDS`
   below), truncates any string value over 100 chars to `first20...last20`
   if the total payload exceeds 1MB.
4. Encrypts the (redacted) headers/request body/response body with
   `encryptWithMasterKey` and inserts a row into `api_request_logs`
   (MySQL) or calls `insertApiRequestLog` (Supabase) — readable back via
   `/api/api-logs`.
5. Injects `requestId: requestUuid` into every JSON response body
   automatically (by monkey-patching `res.json`/`res.send`/`res.end`) —
   you don't need to add it yourself.

**`SENSITIVE_FIELDS`** (exact list, matched case-insensitively as a
substring of the key name): `password`, `api_password`, `passphrase`,
`account_secret`, `private_key`, `privatekey`, `apisecret`, `api_secret`,
`secret`, `token`, `authorization`, `cookie`, `x-api-key`. Add new
sensitive field names here when you introduce them.

## Transport-level

- Helmet CSP (`src/index.ts`) allows `'unsafe-eval'`/`'unsafe-inline'` for
  `script-src`/`style-src` — needed by the served `public/` static pages.
- JSON/urlencoded body parsing has no explicit `limit` set in `src/index.ts`,
  so it falls back to Express's default (~100kb). Raise it with
  `express.json({ limit: '...' })` if an endpoint needs larger payloads
  (e.g. base64-encoded files).
