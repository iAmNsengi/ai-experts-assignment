## What was the bug?

`HttpClient.request` only refreshed the OAuth2 token when `oauth2Token` was falsy or an `OAuth2Token` instance that had expired. If `oauth2Token` was a plain object (e.g. `{ accessToken, expiresAt }`), it was truthy but not an `OAuth2Token`, so the refresh condition was skipped and no `Authorization` header was set.

## Why did it happen?

The `TokenState` type allows `OAuth2Token | Record<string, unknown> | null`, but the runtime logic assumed that any truthy token was either valid or an `OAuth2Token` instance worth checking for expiry. This mismatch between the wider union type and the narrow `instanceof` check meant that plain-object tokens were silently treated as “valid” even though the client could not use them.

## Why does your fix solve it?

The refresh condition now explicitly treats anything that is _not_ an `OAuth2Token` instance, or any `OAuth2Token` whose `expired` getter returns `true`, as invalid and triggers `refreshOAuth2`. After that, `oauth2Token` is guaranteed to be a fresh `OAuth2Token`, so the code reliably sets the `Authorization` header for API calls.

## One realistic edge case not covered by tests

The tests do not cover boundary timing cases where the token is about to expire (e.g. clock skew or a token that expires in the same second the request is made). In a real system, you might want a small safety window (refresh slightly before expiry) and tests that assert correct behavior around that threshold.

