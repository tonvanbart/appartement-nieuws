## Why

The apartment building needs a small news site that one non-technical editor can update almost daily with text and the occasional photo, hosted cheaply, resilient to hacking, and requiring minimum ongoing technical support. We want to prove the workflow with a demo (editor included) before committing to production hosting and real content.

## What Changes

- Scaffold a Hugo site using the existing PaperMod theme, configured for short text-plus-image bulletins on a responsive layout.
- Define a single `posts` content collection using Hugo page bundles (`content/posts/<slug>/index.md` plus an optional image alongside), so phone photos can be resized by Hugo's image processing.
- Embed Sveltia CMS at `/admin/` (served from `static/admin/`) as the editor's surface. The CMS schema makes images optional, but enforces alt text whenever an image is present.
- Wire GitHub OAuth via a self-deployed Cloudflare Worker (Sveltia's `sveltia-cms-auth`), since Sveltia does not provide a hosted OAuth proxy. The editor logs in with a GitHub account that has write access to the repo; the worker brokers the OAuth handshake.
- Configure GitHub Pages with a Hugo build workflow (GitHub Actions) so a Sveltia commit publishes the site within roughly thirty seconds.
- Demo the flow with the repo owner (`josokw`) acting as the editor; collaborator access for the actual editor (Kitty, with her own GitHub account) is added after the demo without site changes.
- Document the deferred decisions and known risks (production hosting, custom domain, content discovery channel) so they are not lost between demo and rollout.

## Capabilities

### New Capabilities

- `content-authoring`: How posts are structured, what fields the editor sees, how images are handled, and how the editor publishes through Sveltia CMS without touching git.
- `site-publishing`: How the Hugo site is built and hosted (GitHub Pages via GitHub Actions), how the editor authenticates, and what guarantees the demo setup makes versus production.

### Modified Capabilities

<!-- None — no prior specs exist in this repo. -->

## Impact

- New repository scaffold under `josokw` on GitHub: Hugo config, PaperMod theme as a submodule or module, page-bundle content layout, Sveltia config under `static/admin/`, GitHub Actions workflow for Hugo build and Pages deploy.
- New GitHub OAuth App registered against the deployed Cloudflare Worker (callback URL: `<worker-url>/callback`).
- Cloudflare account required to deploy the OAuth worker (free tier sufficient).
- Public repo for the demo only; production rollout is gated on a private-repo hosting decision (Cloudflare Pages, or GitHub Pro) so resident names and photos never enter public git history.
- No production domain, no email/RSS distribution, no custom theme, and no editorial review workflow are introduced by this change. They are explicitly deferred.
- Demo URL during this change: `https://josokw.github.io/<repo>/`. The Hugo `baseURL` must match the path prefix or links break.
