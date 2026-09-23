# Maintaining the custom Ghostfolio application

This repository is the application source. It is separate from
`imantzio/my-server`, which stores deployment configuration, benchmark import
tools, and operational documentation.

## Where everything lives

| Item | Location |
| --- | --- |
| Clean official Ghostfolio source | `imantzio/ghostfolio`, branch `main` |
| Personal application code | `imantzio/ghostfolio`, branch `custom` |
| Compiled custom application | `ghcr.io/imantzio/ghostfolio:<versioned-tag>` |
| Docker Compose and server scripts | `imantzio/my-server`, branch `custom` |
| Portfolio and application data | PostgreSQL, outside Git |
| Passwords and application secrets | Each server's `.env`, outside Git |

## Branch rules

### `main`

Keep this branch identical to the official `ghostfolio/ghostfolio` `main`
branch. Do not add personal code or deployment files here.

The `upstream` Git remote points to the official repository. The `origin`
remote points to `imantzio/ghostfolio`.

### `custom`

Make personal Ghostfolio source changes here. This branch currently changes the
Holdings Quantity column to display between zero and eight fractional digits,
without unnecessary trailing zeroes.

## Which repository should I update?

Update this `ghostfolio` repository when changing the application itself, for
example:

- page layout or components;
- number formatting;
- frontend or API behavior;
- Ghostfolio source code; or
- the custom-image build workflow.

Update the `my-server` repository when changing deployment or operations, for
example:

- the Docker image tag used by Compose;
- ports or environment variables;
- backup scripts;
- Afidnes or other benchmark definitions;
- benchmark import scripts; or
- server setup instructions.

Changing application source normally requires both repositories in this order:

1. commit the application change in `ghostfolio/custom`;
2. publish a new versioned Docker image;
3. update the image tag in `my-server/custom`; and
4. pull `my-server/custom` on each server and recreate the application container.

## Updating from official Ghostfolio

First update the clean branch:

```bash
git switch main
git fetch upstream
git merge --ff-only upstream/main
git push origin main
```

Then merge the clean update into the personalized branch:

```bash
git switch custom
git merge main
```

If Git reports a conflict, resolve it on `custom`. Never put conflict-resolution
or personal commits on `main`.

Review the quantity display after an upstream update because Ghostfolio may have
changed the same formatter or holdings table. Commit the merge and any required
adjustment, then push `custom`.

## Publishing a custom Docker image

The workflow `.github/workflows/publish-custom-image.yml` runs only when a Git
tag ending in `-custom.<number>` is pushed. A normal branch push does not publish
an image.

After committing and pushing `custom`, create a unique version tag on the exact
commit to publish:

```bash
git switch custom
git pull --ff-only
git tag 3.72.0-custom.1
git push origin 3.72.0-custom.1
```

The Git tag and Docker image tag are intentionally identical. The tag makes it
clear which source commit produced the image.

The resulting image is:

```text
ghcr.io/imantzio/ghostfolio:3.72.0-custom.1
```

Never move or reuse a published version tag for different code. Increment the
custom revision for another build from the same Ghostfolio version:

```text
3.72.0-custom.1
3.72.0-custom.2
```

After an official version update, use its new version number:

```text
3.73.0-custom.1
```

## Deploying an already published image

Publishing an image does not update a server. Deployment happens from the
`my-server` repository after its Compose image tag has been updated and
committed.

Before an application upgrade, back up PostgreSQL. Then, on each server, pull
the deployment repository and run the documented Compose update commands.

## Current source customization

The Holdings table uses the shared `gf-value` component. The custom branch:

- adds an optional `minimumPrecision` input to
  `libs/ui/src/lib/value/value.component.ts`; and
- configures the Quantity column in
  `libs/ui/src/lib/holdings-table/holdings-table.component.html` with a minimum
  of zero and maximum of eight fractional digits.

This changes display formatting only. It does not change PostgreSQL quantities
or require a database migration.
