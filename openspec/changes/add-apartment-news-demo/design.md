## Context

The apartment building wants a small news site maintained almost daily by a single non-technical editor (Kitty), with photos of building events and announcements about maintenance, deliveries, and shared facilities. The team is already familiar with Hugo. The repo will live under the GitHub user `josokw`. A demo is needed first — with the eventual editor present — to prove the publishing workflow before any production decision is made.

The hard constraints from exploration: Markdown-based content, static-only output (cheap and resilient), responsive presentation, and minimum ongoing technical support. The editor must be able to publish without using git or a terminal. Production hosting and custom-domain wiring are explicitly deferred until after the demo.

## Goals / Non-Goals

**Goals:**

- Stand up a Hugo site using PaperMod that renders cleanly on phone, tablet, and desktop.
- Provide a `/admin/` URL where the editor logs in with a GitHub account and publishes posts via a form.
- Make image uploading optional but enforce alt text whenever an image is attached.
- Resize phone-camera photos automatically at build time so the live site stays light.
- Build and deploy via GitHub Actions and GitHub Pages with no human in the loop after a Sveltia commit.
- Make collaborator onboarding (Kitty's GitHub account) a post-demo configuration step that requires no site changes.
- Capture deferred decisions (production host, custom domain, content discovery channel) so they survive the gap between demo and rollout.

**Non-Goals:**

- Production hosting on a private repo (Cloudflare Pages or GitHub Pro) — flagged but not implemented.
- Custom domain setup; the demo runs on `josokw.github.io/<repo>/`.
- Custom theme; PaperMod with minimal overrides only.
- Email digest, RSS-driven newsletter, or push notifications.
- Categories, tags, or any taxonomy in the content model.
- Editorial review, draft preview branches, or multi-editor workflows.
- Self-hosted OAuth worker (deferred to production hardening).
- i18n of the editor's form labels beyond what Sveltia ships out of the box.

## Decisions

### Hugo + PaperMod over a custom theme or alternate generator

PaperMod is widely used, actively maintained, responsive by default, and has good built-in image handling. A custom theme would be unnecessary support debt for a demo. Other generators (Astro, Eleventy) were considered but Hugo is what the team already runs, and "minimum support" rewards familiarity over novelty. PaperMod's defaults will be lightly overridden so text-only posts (no cover image) render cleanly in list views.

### Sveltia CMS over Decap, TinaCMS, or raw markdown editing

Sveltia is the most actively maintained Decap-compatible CMS, ships as a single static page (no server), commits markdown straight to the repo, and supports per-collection media folders so uploaded images land inside the post's page bundle. Decap is functionally equivalent but slower-moving. Tina requires its own backend or a hosted account, which violates "minimum support". Raw markdown editing was rejected because daily git operations are exactly the friction the editor cannot absorb.

### Page bundles over flat content + flat uploads

Each post is a folder: `content/posts/<slug>/index.md` plus the (optional) image. This co-locates the photo with its post, makes Hugo's image processing pipeline available (resize, srcset, format conversion), and makes deletion atomic. The flat alternative (`content/posts/<slug>.md` + `static/uploads/<file>.jpg`) is simpler but gives up image resizing and orphans uploads when posts are deleted. Phone photos are typically 4–6 MB; serving those un-processed is the single fastest way to break the site within a month.

### Image as an optional object widget, alt text required only when image is present

Sveltia's `object` widget groups fields into a unit that can itself be marked optional. By placing `file` and `alt` inside an optional `image` object, the editor sees no image fields when she doesn't need one, and is forced to write alt text the moment she does. A flat `image?` plus `alt?` would let her upload without alt — accessibility regression we will not accept. This shape also keeps text-only posts ergonomic for daily quick announcements.

### GitHub Pages for the demo over Cloudflare Pages

GitHub Pages keeps everything on github.com — repo, build, hosting, auth, dashboard. One vendor, one account, fewer things to keep alive. Hugo isn't built natively but the official Hugo docs publish a copy-paste GitHub Actions workflow that handles it. Cloudflare Pages is technically nicer (native Hugo build, faster edge) but adds a second vendor for marginal demo value. The GitHub Pages free tier requires the repo to be **public**, which is acceptable for the demo (fake content) and explicitly *not* acceptable for production — see migration plan.

### Sveltia hosted OAuth proxy for the demo over PAT or self-hosted worker

Sveltia is a static SPA and cannot keep an OAuth client secret. The demo will register a GitHub OAuth App and use Sveltia's hosted proxy to broker the login. PAT login was rejected because tokens expire and management leaks to the editor. A self-hosted Cloudflare Worker is a better long-term answer (no third-party dependency in the auth path) but is out of scope for the demo and lands as a production hardening step.

### Editor identity model: collaborator with a separate GitHub account

For the demo, Jos (`josokw`) logs in as the editor. Post-demo, Kitty creates her own free GitHub account and is added to the repo as a collaborator with write access. Sveltia checks repo write access; it does not care about identity. A separate account gives Jos a clean audit trail (commits show Kitty), one-click revocation, smaller blast radius, and least-privilege isolation from his other repositories.

## Risks / Trade-offs

- **Phone-photo weight on the live site** → Wire Hugo image processing into PaperMod's post layout (or a small layout override) so the rendered HTML uses resized, format-converted variants with `srcset`. Verify with a real phone photo before declaring the demo done.
- **PaperMod list view shows placeholder thumbnails for imageless posts** → Override the relevant partial or set theme params to suppress the cover slot when no image is present. Visual check in list view, not just on the post page.
- **Sveltia hosted OAuth proxy is a third-party dependency on demo day** → Test the full login flow at least once the day before the demo. If the proxy is unreachable, document the PAT fallback as the break-glass path; do not present it as the recommended workflow.
- **Public repo exposes resident data if used for production** → Treat the public-repo demo as throwaway. Production rollout requires a deliberate host decision (Cloudflare Pages + private repo, or GitHub Pro) before any real names or photos are committed. The migration plan below makes this explicit.
- **Hugo `baseURL` mismatch breaks links on `josokw.github.io/<repo>/`** → Configure `baseURL` to include the repo path prefix, and verify internal links and the Sveltia bundle paths after first deploy.
- **GitHub Actions build minutes on a public repo are unmetered, but workflow churn from many daily commits is still noisy** → Acceptable for the use case; flag again only if it changes.
- **Discovery problem (tenants don't visit the site daily)** → Out of scope for this change. Capture as an open question so it doesn't get lost between the demo and rollout.

## Migration Plan

This is a greenfield change — no existing site, no existing content, no rollback needed. The relevant "migration" is the path from demo to production:

1. Demo runs on a **public** GitHub repo with fake or sample content. Kitty is added as a collaborator (her own account) only after the demo proves out.
2. Before any real content with resident names or photos is published, decide and execute the production host:
   - **Path A**: Convert the repo to private and upgrade `josokw` to GitHub Pro (small monthly cost) so GitHub Pages can serve from a private repo.
   - **Path B**: Keep the repo private on the free plan and switch hosting to Cloudflare Pages (free, native Hugo build). This adds a Cloudflare account but no per-month cost.
3. Custom domain wiring (third-party DNS) happens at the same time as the production host switch. Hugo `baseURL` updates accordingly.
4. Replace Sveltia's hosted OAuth proxy with a self-hosted Cloudflare Worker once the production host is stable, removing the third-party dependency from the auth path.

If the demo fails (OAuth flow won't work, or stakeholders reject the editor UX), the rollback is to delete the GitHub OAuth App, archive the repo, and revisit the editor-surface decision (text editor + GitHub web UI, or a different CMS). Nothing is in production, so no data is at risk.

## Open Questions

- **Form-label language** — RESOLVED: Dutch. Sveltia UI locale set to `nl`, all field labels and helper text written in Dutch.
- **Repo name** — RESOLVED: `appartement-nieuws`. Demo URL: `https://josokw.github.io/appartement-nieuws/`. Hugo `baseURL` set accordingly.
- **Demo URL acceptable as-is?**: stakeholders may want to see the eventual domain hooked up before they sign off, even though the production-host decision is deferred. Verify before the demo whether `josokw.github.io/appartement-nieuws/` is a credible-looking URL for the audience.
- **Discovery channel**: how do tenants learn there is a new post? RSS, email digest seeded from RSS, a WhatsApp link share, or a building-bulletin-board QR code? Not a blocker for the demo, but the home-page shape (recent-posts list vs. single most-recent vs. dated index) should be informed by the answer once it lands.
- **Content seeding for the demo**: 3–5 sample posts mixing text-only and text-plus-image. Who writes the sample content — Jos, or the user (Ton)?
