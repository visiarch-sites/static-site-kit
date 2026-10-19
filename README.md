# static-site-kit

How every static client site of `visiarch-sites` is built and served: GitHub Actions builds the site, packs
it into a small Nginx image on GHCR, and Coolify runs one container per site (a few MB of RAM each).

- `base/`: the shared Nginx image, `ghcr.io/visiarch-sites/static-site-base` (gzip, a year of cache for
  hashed files, security headers, `/healthz`, `/404.html`). Rebuilt by `.github/workflows/base-image.yml`.
- `.github/workflows/deploy.yml`: the reusable workflow each site calls.
- `template/`: the one file a site's repository needs.

Works for any build that writes a folder of static files: Astro (`dist`), Vite (`dist`), plain HTML with
a build step, Next.js static export (`out`).

## Once, for the organization

1. Organization **Settings → Actions → General → Workflow permissions**: Read and write.
2. Organization **Settings → Secrets and variables → Actions**, secrets for all repositories:
   - `COOLIFY_BASE_URL`: the Coolify dashboard address.
   - `COOLIFY_TOKEN`: a Coolify API token with deploy permission.
3. This repository is public (nothing secret in it), and so is the `static-site-base` package
   (**Packages → static-site-base → Package settings → Change visibility → Public**), so every site's
   build can pull it.
4. The Coolify server is logged in to GHCR (`docker login ghcr.io`) with a token that can read the
   organization's packages: the site images themselves stay private.

## Adding a site

1. **Repository** in `visiarch-sites`, with the site's code and `package-lock.json`.
2. Copy `template/.github/workflows/deploy.yml` into it. Push: the image
   `ghcr.io/visiarch-sites/<repo>:latest` appears under the organization's Packages.
3. **Coolify** → project *Client Sites* → New resource → **Docker Image**:
   - Image `ghcr.io/visiarch-sites/<repo>:latest`, port **80**.
   - Domains: `https://example.com,https://www.example.com`.
   - Health check path `/healthz`.
   - Deploy, then copy the resource's UUID from its URL.
4. Repository **Settings → Secrets and variables → Actions → Variables**: `COOLIFY_RESOURCE_UUID` = that
   UUID. From now on every push to `main` builds and redeploys the site.
5. **DNS** (Cloudflare, proxied): `@` and `www` A records to the server. SSL/TLS mode **Full (strict)**.

## Rolling back

Every build is also tagged `sha-<commit>`. In Coolify, set the resource's image tag to that commit and
redeploy; set it back to `latest` afterwards.

## Changing the shared Nginx setup

Edit `base/`, push to `main`: the base image is rebuilt. Sites pick it up on their next deploy (or run
their Deploy workflow by hand).
