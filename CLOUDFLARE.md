# Cloudflare Pages preview

This configuration serves only the Create React App `build/` directory. It adds
no Functions, bindings, secrets, custom domains, or paid services.

## Build

Use the existing Yarn v1 lockfile without updating dependencies. Run the build
with Node 14.21.3:

```sh
npx yarn@1.22.22 install --frozen-lockfile --ignore-scripts
node --version # v14.21.3
node node_modules/react-scripts/scripts/build.js
```

The legacy locked PostCSS version is incompatible with Node 22 package exports.
Node 14 is used only to build static files; no Node runtime is deployed. A
toolchain upgrade is separate work. The build keeps the existing browser-side
Pixabay integration. No new
environment variables are required. Do not put private API credentials into a
client build.

## Preview publication

Use an already authenticated Wrangler installation and the intended account:

```sh
CLOUDFLARE_ACCOUNT_ID=fa9f73e26e04ae80f02ac35d16eaa837 \
  npx wrangler@4.155.0 pages deploy build \
  --project-name=pixabay-coding-exercise \
  --branch=migration-cloudflare-pages
```

Keep the Pages production branch as `main`; the branch above is a preview.
Publish only the build directory. Keep the existing Vercel publication while
preview checks are pending. The `*.vercel.app` address cannot be transferred to
Cloudflare; publishing a preview does not switch existing visitors.

## Routing and verification

Cloudflare Pages automatically serves `/index.html` when a request does not
match an asset and no top-level `404.html` exists. Keep that default SPA behavior
for React Router's `/:imageid` route; no catch-all redirect should intercept JS,
CSS, or image files.

After publication, check the home page, JS/CSS content types, gallery API
response, pagination, a gallery-to-detail click, and browser back navigation.
Also check that an unknown path receives the SPA HTML rather than a hosting
404. See [Pages serving behavior](https://developers.cloudflare.com/pages/configuration/serving-pages/#single-page-application-spa-rendering).

The existing detail component expects `history` navigation state. Opening a
detail URL directly without that state can fail in the application even when
the hosting fallback works. That existing application behavior is not changed
by the hosting configuration.

## Rollback boundary

The migration starts from Vercel production source
`e0c043297f93a140be849ce923849b35b16e8a47`. Vercel, DNS, the public API
integration, and the dependency lockfile are unchanged. Review the tested
Cloudflare preview before deciding on production publication or link changes.
