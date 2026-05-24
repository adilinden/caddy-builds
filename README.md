# caddy-binaries

Builds Caddy with extended modules using xcaddy and publishes versioned
binaries as GitHub Release assets.

Used by the `caddy` deploy repo on Forgejo to install a pinned, verified
Caddy binary on the caddy LXC.

## Variables

| Variable | Purpose | Reference |
|----------|---------|-----------|
| `CADDY_VERSION` | Caddy release version to build | https://github.com/caddyserver/caddy/releases |
| `XCADDY_VERSION` | xcaddy version used to perform the build | https://github.com/caddyserver/xcaddy/releases |

## Variants

Variants are defined in `modules/*.env`. Each file specifies a `VARIANT` name
and a space-separated list of `MODULES` to include at build time.

| File | Variant | Modules |
|------|---------|---------|
| `modules/dns-cloudflare.env` | `dns-cloudflare` | caddy-dns/cloudflare |
| `modules/dns-azure.env` | `dns-azure` | caddy-dns/azure |
| `modules/full.env` | `full` | cloudflare, azure |

Naming convention for module files:

| Prefix | Category |
|--------|----------|
| `dns-` | DNS provider modules |
| `auth-` | Authentication / SSO |
| `storage-` | Storage backends |

## Module Version Pinning

By default, xcaddy fetches the latest version of each module at build time.
If a module has a minimum Caddy version requirement that exceeds the requested
`CADDY_VERSION`, or if a specific module version is required for any other
reason, pin the version directly in `MODULES` using `module@version` syntax:

```bash
# modules/dns-azure.env — pinned to a version compatible with an older Caddy
VARIANT=dns-azure
MODULES=github.com/caddy-dns/azure@v0.5.0
```

When pinning a module version, update **both** the individual variant file and
`modules/full.env` to keep them consistent.

Available module versions: check the releases page of the relevant module repo
on GitHub (e.g. https://github.com/caddy-dns/azure/releases).

## Adding a Variant

1. Create `modules/<category>-<name>.env` with `VARIANT` and `MODULES`
2. If the module requires a minimum Caddy version, pin it with `module@version`
3. Open a pull request — the validate workflow confirms versions are valid
4. Merge — the build workflow produces the new variant automatically

No workflow changes required.

## Upgrading

1. Edit `versions.env` on a branch — bump `CADDY_VERSION` and optionally
   `XCADDY_VERSION`
2. Open a pull request targeting `main`
3. The validate workflow runs automatically:
   - Confirms the Caddy version exists on caddyserver/caddy releases
   - Confirms the xcaddy version exists on caddyserver/xcaddy releases
   - Confirms no release for this version already exists here
4. Merge if all checks pass
5. The build workflow runs and publishes all variant binaries as release assets

## Download

Specific version and variant (used by deploy.sh):

    https://github.com/adilinden/caddy-binaries/releases/download/v2.11.3/caddy-linux-amd64-dns-cloudflare
    https://github.com/adilinden/caddy-binaries/releases/download/v2.11.3/caddy-linux-amd64-dns-cloudflare.sha256

## Branch Protection

main is a protected branch. Direct pushes are not permitted.
All changes go through a pull request with required status checks.

## Branch Protection Settings

Configure in GitHub → Settings → Branches → Add rule for `main`:

| Setting | Value |
|---------|-------|
| Require a pull request before merging | ✓ |
| Require status checks to pass | ✓ |
| Required status check | `validate` |
| Require branches to be up to date | ✓ |
| Do not allow bypassing the above settings | ✓ |
| Allow direct pushes | ✗ |

## Changelog

| Date | Change |
|------|--------|
| 2026-05-24 | Document module version pinning |
| 2026-05-23 | Add matrix build with variant support (dns-cloudflare, dns-azure, full) |
| 2026-05-23 | Initial release |
