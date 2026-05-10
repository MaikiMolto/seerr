<p align="center">
<img src="./public/logo_full.svg" alt="Seerr" style="margin: 20px 0;">
</p>
<p align="center">
<img src="https://github.com/seerr-team/seerr/actions/workflows/release.yml/badge.svg" alt="Seerr Release" />
<img src="https://github.com/seerr-team/seerr/actions/workflows/ci.yml/badge.svg" alt="Seerr CI">
</p>
<p align="center">
<a href="https://discord.gg/seerr"><img src="https://img.shields.io/discord/783137440809746482" alt="Discord"></a>
<a href="https://hub.docker.com/r/seerr/seerr"><img src="https://img.shields.io/docker/pulls/seerr/seerr" alt="Docker pulls"></a>
<a href="https://translate.seerr.dev/engage/seerr/"><img src="https://translate.seerr.dev/widget/seerr/svg-badge.svg" alt="Translation status" /></a>
<a href="https://github.com/seerr-team/seerr/blob/develop/LICENSE"><img alt="GitHub" src="https://img.shields.io/github/license/seerr-team/seerr"></a>

**Seerr** is a free and open source software application for managing requests for your media library. It integrates with the media server of your choice: [Jellyfin](https://jellyfin.org), [Plex](https://plex.tv), and [Emby](https://emby.media/). In addition, it integrates with your existing services, such as **[Sonarr](https://sonarr.tv/)**, **[Radarr](https://radarr.video/)**.

## Custom fork note

This fork exists to keep a small but important **API request permission fix** available as a continuously updated Docker image, automatically synced with upstream Seerr releases.

### What is changed?

When a media request is created through the API with a `userId`, Seerr should evaluate auto-approve permissions against the **target user** (`requestUser`), not the **API caller** (`user`).

This fork changes the auto-approve checks in `server/entity/MediaRequest.ts` from `user.hasPermission(...)` to `requestUser.hasPermission(...)` for movie, TV, and season request approval handling.

### Why does this matter?

Without this fix, API requests created with an admin API key can be auto-approved based on the admin's permissions, **even when the target user should require manual approval**.

This matters for third-party integrations and bots that create requests on behalf of real users, for example:

- 🤖 n8n workflows
- 📱 Telegram bots
- 💬 WhatsApp bots
- 🏠 Home Assistant dashboards
- 🔧 Any automation calling the Seerr API with an admin key on behalf of regular users

### Upstream status

We submitted the fix upstream but it was rejected — the maintainers consider the current behavior intentional for admin API keys.

- Issue: <https://github.com/seerr-team/seerr/issues/2678>
- PR: <https://github.com/seerr-team/seerr/pull/2679>

This fork therefore exists as a permanent home for users who need the alternative behavior.

## How this fork stays in sync with upstream

A GitHub Actions workflow runs **daily at 04:00 UTC** and:

1. Fetches the latest changes from `seerr-team/seerr:main`
2. Rebases this fork's `develop` branch on top of upstream
3. Pushes the result back, which triggers an automatic Docker build
4. The new image is published with multiple tags (see below)
5. If a rebase conflict occurs, an issue is opened automatically and the build is skipped until manually resolved

This means: **whenever upstream Seerr releases a new version, a patched image is usually available within ~10 minutes**, with no manual intervention needed.

## Available image tags on GHCR

All tags are published to `ghcr.io/maikimolto/seerr`:

| Tag | Use case | Example |
|---|---|---|
| `latest` | Rolling — always the most recent build | `ghcr.io/maikimolto/seerr:latest` |
| `v3.2.0-patched` | Pin to a specific upstream version | `ghcr.io/maikimolto/seerr:v3.2.0-patched` |
| `sha-<short>` | Pin to an exact commit (reproducible) | `ghcr.io/maikimolto/seerr:sha-b6c9eb0` |

## Using this fork

### Quick start with docker compose

```yaml
services:
  seerr:
    image: ghcr.io/maikimolto/seerr:latest
    init: true
    container_name: seerr
    restart: unless-stopped
    environment:
      - TZ=Europe/Berlin
      - PORT=5055
    volumes:
      - ./seerr-config:/app/config
    ports:
      - "5055:5055"
    healthcheck:
      test: ["CMD-SHELL", "wget -q -O - --timeout=5 http://localhost:5055/api/v1/status >/dev/null"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 30s
```

### When to use this fork

Use this fork **only if** you create requests via the Seerr API on behalf of other users (e.g. via bots or automations) and you want auto-approve to honor the target user's permissions instead of the API caller's. For interactive use, vanilla upstream Seerr behaves identically.

### When **not** to use this fork

- You only use the Seerr UI (no API integrations) — vanilla upstream is fine
- You want vendor support — only upstream offers official support; this fork is community-maintained
- You are running a hosted/managed Seerr instance — stick with upstream

### Disclaimer

This fork is provided as-is for personal homelab use. The patch is intentionally minimal (5 lines in one file) and aimed at staying mergeable with upstream forever. Use at your own risk.

## Current Features

- Full Jellyfin/Emby/Plex integration including authentication with user import & management.
- Support for **PostgreSQL** and **SQLite** databases.
- Supports Movies, Shows and Mixed Libraries.
- Ability to change email addresses for SMTP purposes.
- Easy integration with your existing services. Currently, Seerr supports Sonarr and Radarr. More to come!
- Jellyfin/Emby/Plex library scan, to keep track of the titles which are already available.
- Customizable request system, which allows users to request individual seasons or movies in a friendly, easy-to-use interface.
- Incredibly simple request management UI. Don't dig through the app to simply approve recent requests!
- Granular permission system.
- Support for various notification agents.
- Mobile-friendly design, for when you need to approve requests on the go!
- Support for watchlisting & blocklisting media.

With more features on the way! Check out our [issue tracker](/../../issues) to see the features which have already been requested.

## Getting Started

Check out our documentation for instructions on how to install and run Seerr:

https://docs.seerr.dev/getting-started/

## Preview

<img src="./public/preview.jpg" alt="Seerr application preview" />

## Migrating from Overseerr/Jellyseerr to Seerr

Read our [release announcement](https://docs.seerr.dev/blog/seerr-release) to learn what Seerr means for Jellyseerr and Overseerr users.

Please follow our [migration guide](https://docs.seerr.dev/migration-guide) for detailed instructions on migrating from Overseerr or Jellyseerr.

## Support

- Check out the [Seerr Documentation](https://docs.seerr.dev) before asking for help. Your question might already be in the docs!
- You can get support on [Discord](https://discord.gg/seerr).
- You can ask questions in the Help category of our [GitHub Discussions](/../../discussions).
- Bug reports and feature requests can be submitted via [GitHub Issues](/../../issues).

## API Documentation

You can access the API documentation from your local Seerr install at http://localhost:5055/api-docs

## Community

You can ask questions, share ideas, and more in [GitHub Discussions](/../../discussions).

If you would like to chat with other members of our growing community, [join the Seerr Discord server](https://discord.gg/seerr)!

Our [Code of Conduct](./CODE_OF_CONDUCT.md) applies to all Seerr community channels.

## Contributing

You can help improve Seerr too! Check out our [Contribution Guide](./CONTRIBUTING.md) to get started.

## Contributors ✨

[![Contributors](https://opencollective.com/seerr/contributors.svg?width=890)](https://opencollective.com/seerr/#backers)

[![Become a Backer](https://opencollective.com/seerr/backers.svg)](https://opencollective.com/seerr/#backers)
[![Become a Sponsor](https://opencollective.com/seerr/sponsors.svg)](https://opencollective.com/seerr/#sponsors)
