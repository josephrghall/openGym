# Genesis integration — original OpenGym interface

This repository is a fork of https://github.com/DuarteSantos8/openGym. Genesis runs the original OpenGym React interface, API, storage, and guided workouts rather than recreating it.

## Repository and authority

- This fork owns OpenGym application source, customizations, tests, and changes to its user interface.
- Project Genesis (josephrghall/project_genesis) owns integration-level contracts and the relationship to Genesis Core. Engineering issue: https://github.com/josephrghall/project_genesis/issues/90.
- The private Second Brain vault (josephrghall/second-brain-vault) owns personal plans and its existing web command center, not this app's source code or its database.
- Genesis Core does not depend on OpenGym. Workout sets and sensor readings are not universal Core primitives.

## Deployment

Source checkout: /srv/opengym/app on VPS srv1969440. Git origin is josephrghall/openGym and upstream is DuarteSantos8/openGym. The normal Docker Compose source builds produce OpenGym's API and web containers.

The VPS has an untracked, local compose.override.yml binding the web container to 127.0.0.1:5211, with private .env configuration for BASE_PATH=/gym, ORIGIN=https://srv1969440.hstgr.cloud and RP_ID=srv1969440.hstgr.cloud. Its existing nginx server forwards HTTPS /gym/ to that local container, passing through the path prefix. The temporary additional nginx Basic authentication was removed when enabling native pairing and MCP OAuth; OpenGym now requires only its own session, passkey/password, OAuth consent or scoped API-token checks.

The original web UI: https://srv1969440.hstgr.cloud/gym/. OpenGym authentication remains required; there is no extra nginx login. The existing Genesis command-center links to this page; it does not embed or copy the application.

No wearable or nutrition syncing has been configured. Android browser/PWA uses the same HTTPS page. Native Android pairing and Health Connect require separate testing and permissions; the additional Basic-auth proxy may need adjustment for the native app.

## Upstream maintenance

Keep origin pointing to this fork and upstream pointing to the original author. Review upstream releases and merge changes to an isolated update branch, then test and raise a PR against our fork. Avoid permanently diverging from the upstream UI merely for branding. Backup OpenGym's local data directory before updates. Never commit data/, .env, access credentials, authentication tokens, personal body measurements, or logs.

## Licensing

The app source is GNU AGPLv3; preserve LICENSE and NOTICE, and provide source as required for modified network-served builds. Exercise images and animations have separate usage terms and must not be reused outside OpenGym. For details see upstream NOTICE.md.

## Future Genesis adapter

Expose an explicitly scoped adapter for OpenGym plans, workout sessions, and exercise performance (provider IDs, timestamps, provenance, corrections). Link domain identities to Genesis later rather than copying OpenGym's entire canonical store. Apple Health/HealthKit, Android Health Connect, and nutrition are independent source integrations, not preexisting OpenGym capabilities.

## Active MCP connection (November 16, 2026)

The optional OpenGym MCP service is now running, built from the fork source, under the standard Compose profile `mcp`; `.env` contains `MCP=on` and `COMPOSE_PROFILES=mcp`. The existing `/gym/` proxy already forwards `/gym/mcp` to the built-in OpenGym proxy; no extra MCP service port is published. OAuth metadata routes under `/gym/.well-known/` and the exact RFC 8414 paths `/.well-known/oauth-authorization-server/gym` and `/.well-known/oauth-protected-resource/gym/mcp` are proxied by existing host nginx.

Connection URL: `https://srv1969440.hstgr.cloud/gym/mcp`.

Verified from HTTPS: browser app 200, app health 200, OAuth metadata 200, unauthorized personal data 401, unauthenticated MCP tool request 401. Web/API/MCP Docker containers healthy. **No account consent or scoped token has been granted to any agent yet**, and no training data was fetched or changed.

To authorize an agent, use OpenGym Settings → Use your subscription (OAuth connector) or create one agent-specific API token in Settings → Integrations → API tokens; never share raw credentials in GitHub or the vault. Restrict scope to the required domain and write operation. Use distinct, revocable scopes per agent; keep deterministic automation and UI access through the original HTTP API as appropriate.

No device health permissions, mobile-pairing, or agent write operations have been tested yet.


## ChatGPT OAuth compatibility fix — November 16, 2026

Claude's OAuth MCP connection worked; ChatGPT's failed at /gym/api/oauth/authorize with HTTP 400. Reproduced from ChatGPT's published CIMD metadata (https://chatgpt.com/oauth/client.json): legacy token_endpoint_auth_method prefers private_key_jwt, but plural token_endpoint_auth_methods_supported includes the public none method supported by OpenGym. Earlier api/oauth.js validated the singular preference only.

Fixed in fork PR #1 (https://github.com/josephrghall/openGym/pull/1): use plural supported-methods when present; accept only clients including none; preserve rejection of clients offering only unsupported private methods. Existing PKCE, OAuth consent, scoped token requirements and login remain unchanged.

After API-only source build/deploy, all 23 targeted unit/server OAuth tests passed. A synthetic ChatGPT published-identity authorize request now returns HTTP 302 to OpenGym consent (formerly HTTP 400). Web/API/MCP containers are healthy; unauthorized data and MCP tool access still return HTTP 401. The previous API image is preserved locally as genesis-opengym-api:pre-chatgpt-cimd-20261116 for rollback. The existing Claude authorization was not removed or edited. Final ChatGPT user consent, token exchange, and tool read still require a retry from ChatGPT.


## Connector field verification after the compatibility fix — November 16, 2026 Pacific

After fork PR #1 was merged and the API was rebuilt, the owner reported that ChatGPT's custom MCP connector connected successfully. Public HTTPS request status logs showed the ChatGPT MCP client completing protocol requests with HTTP 200/202 on November 17 UTC. Claude OAuth/MCP requests had already been verified successful. This supersedes the previous checkpoint saying ChatGPT authorization alone was tested. No secret token, account identifier, personal measurements, or raw access logs are recorded here.

**Verified scope:** OAuth authorization and MCP protocol exchange. A separate private vault investigation found no persisted per-profile saved training state after initial account login; a real user-approved workout, routine or body-weight edit still needs to sync before an agent can read meaningful workout records. Do not fabricate sample workouts in the owner's production profile.

Cross-system chronology and open acceptance criteria:
- https://github.com/josephrghall/project_genesis/blob/docs/genesis-body-vertical-slice/docs/03-development/GENESIS-BODY-OPENGYM-PROTOTYPE-RECORD.md (architecture PR review branch until merged)
- https://github.com/josephrghall/project_genesis/issues/90
