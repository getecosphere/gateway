# Gateway LXS integration guide

Gateway is the estate front door. Compose it once with `eco lxs add
gateway@<version>` and never hand-edit generated gateway files. The source of
truth is each service's `access.routes` in the manifest generated/managed by
Eco.

## Required integration order

1. Add gateway, auth, then UI/domain LXS through `eco lxs add`.
2. Declare every HTTP route with `public`, `auth`, or `role:<name>`.
3. Declare `auth.roles`, including `public` and a default authenticated role.
4. Use exact routes before wildcards. Gateway chooses the longest matching
   prefix and otherwise returns 404—there is no catch-all forward.
5. Browser SSR pages protected by Auth must use `cookie: eco_token`; browser
   API calls use the bearer token.

## Auth pattern

Auth's external `/auth-api/*` browser alias must strip `/auth-api` and rewrite
to `/api`; each public credential path is declared explicitly, while the rest
requires auth. A profile alias should similarly use `/api/profile/*` → `/api`.
This keeps LXS internal paths stable and estate paths readable.

Gateway passes the bearer header and injects trusted `X-Eco-User` only after
validation. Never treat a client-provided identity header as trusted. TLS ends
at the tunnel; the gateway remains localhost-only.

## Release discipline

If route semantics or generated-config behavior changes, update source docs
and this guide, build/publish a versioned binary, and push source plus registry
changes. Test public, unauthenticated, and authenticated route behavior.
