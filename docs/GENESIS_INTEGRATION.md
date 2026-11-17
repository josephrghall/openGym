# Genesis integration — original OpenGym interface

This repository is a fork of https://github.com/DuarteSantos8/openGym. Genesis runs the original OpenGym React interface, API, storage, and guided workouts rather than recreating it.

## Repository and authority

- This fork owns OpenGym application source, customizations, tests, and changes to its user interface.
- Project Genesis (josephrghall/project_genesis) owns integration-level contracts and the relationship to Genesis Core. Engineering issue: https://github.com/josephrghall/project_genesis/issues/90.
- The private Second Brain vault (josephrghall/second-brain-vault) owns personal plans and its existing web command center, not this app's source code or its database.
- Genesis Core does not depend on OpenGym. Workout sets and sensor readings are not universal Core primitives.

## Deployment

Source checkout: /srv/opengym/app on VPS srv1969440. Git origin is josephrghall/openGym and upstream is DuarteSantos8/openGym. The normal Docker Compose source builds produce OpenGym's API and web containers.

The VPS has an untracked, local compose.override.yml binding the web container to 127.0.0.1:5211, with private .env configuration for BASE_PATH=/gym, ORIGIN=https://srv1969440.hstgr.cloud and RP_ID=srv1969440.hstgr.cloud. Its existing nginx server forwards HTTPS /gym/ to that local container, passing through the path prefix. During first-account bootstrap nginx also requires Basic authentication. The credentials live outside Git under /root/.config/genesis-opengym/access.txt.

The original web UI: https://srv1969440.hstgr.cloud/gym/. Both OpenGym login and the temporary external login gate must pass. The existing Genesis command-center links to this page; it does not embed or copy the application.

No wearable or nutrition syncing has been configured. Android browser/PWA uses the same HTTPS page. Native Android pairing and Health Connect require separate testing and permissions; the additional Basic-auth proxy may need adjustment for the native app.

## Upstream maintenance

Keep origin pointing to this fork and upstream pointing to the original author. Review upstream releases and merge changes to an isolated update branch, then test and raise a PR against our fork. Avoid permanently diverging from the upstream UI merely for branding. Backup OpenGym's local data directory before updates. Never commit data/, .env, access credentials, authentication tokens, personal body measurements, or logs.

## Licensing

The app source is GNU AGPLv3; preserve LICENSE and NOTICE, and provide source as required for modified network-served builds. Exercise images and animations have separate usage terms and must not be reused outside OpenGym. For details see upstream NOTICE.md.

## Future Genesis adapter

Expose an explicitly scoped adapter for OpenGym plans, workout sessions, and exercise performance (provider IDs, timestamps, provenance, corrections). Link domain identities to Genesis later rather than copying OpenGym's entire canonical store. Apple Health/HealthKit, Android Health Connect, and nutrition are independent source integrations, not preexisting OpenGym capabilities.
