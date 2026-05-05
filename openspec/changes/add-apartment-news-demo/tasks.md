## 1. Repo and toolchain setup

- [x] 1.1 Decide and lock the repo name (e.g. `apartment-news`); confirm it owns the demo URL `https://josokw.github.io/<repo>/`
- [x] 1.2 Confirm whether Sveltia form labels need to be Dutch; pin the answer in `design.md` open questions
- [ ] 1.3 Initialise the Hugo site locally (`hugo new site`), commit the scaffold, push to a public repo under `josokw`
- [x] 1.4 Add the PaperMod theme (Hugo Module preferred over git submodule for easier upgrades); commit the lock file
- [x] 1.5 Configure `hugo.toml` (or `config.toml`): site title, language, `baseURL` set to the path-prefixed Pages URL, PaperMod theme params for clean defaults
- [x] 1.6 Add a `.gitignore` covering `public/`, `resources/`, and Hugo build artefacts

## 2. Content model and theme adjustments

- [x] 2.1 Create `archetypes/posts.md` so `hugo new posts/<slug>/index.md` produces the expected frontmatter shape (title, date, draft=false, no image by default)
- [ ] 2.2 Author 3–5 sample posts as page bundles under `content/posts/<slug>/`, mixing text-only with text-plus-image; commit a real (or stand-in) phone photo for at least one post
- [ ] 2.3 Verify Hugo image processing: render one post locally, inspect the HTML, confirm `srcset` variants and that the original full-resolution file is not the served asset
- [x] 2.4 Override PaperMod's list-view partial (or use theme params) so posts without a cover image render cleanly with no placeholder thumbnail
- [ ] 2.5 Visual check on phone, tablet, and desktop viewports for both text-only and text-plus-image posts

## 3. Sveltia CMS

- [x] 3.1 Create `static/admin/index.html` that loads Sveltia's bundled JS and renders the CMS shell
- [x] 3.2 Create `static/admin/config.yml` defining a single `posts` collection with: `title` (required), `date` (auto, hidden or read-only), `summary` (optional), `body` (markdown), `draft` (boolean, default false), and an optional `image` object widget containing `file` (required-within-object) and `alt` (required-within-object)
- [x] 3.3 Set the collection's `media_folder` and `public_folder` so uploaded images land inside the post's page bundle directory, not in a flat `static/uploads/`
- [ ] 3.4 Smoke-test the form locally (`hugo server`): confirm the image block is collapsible, alt is required only when a file is attached, and the generated frontmatter matches the schema
- [x] 3.5 If form-label language was decided as Dutch in 1.2, set Sveltia's UI locale and translate field labels in `config.yml`

## 4. GitHub OAuth and editor identity

- [ ] 4.1a Deploy `sveltia/sveltia-cms-auth` as a Cloudflare Worker; record its `*.workers.dev` URL
- [ ] 4.1b Register a GitHub OAuth App with the Worker's `/callback` URL as the Authorization callback URL; copy Client ID and generate Client Secret
- [ ] 4.1c Add `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, and `ALLOWED_DOMAINS` (set to the Pages hostname) as Worker environment variables
- [ ] 4.2 Set `backend.base_url` in `static/admin/config.yml` to the Worker URL and push
- [ ] 4.3 Verify Jos can log in to `/admin/` and that the login token does not appear in any committed file
- [x] 4.4 Document (in repo `README.md`, brief) the steps to add Kitty post-demo: create her GitHub account, accept the collaborator invite, log in to `/admin/` — no site changes
- [x] 4.5 Capture a PAT-login fallback path in the same README as a break-glass option only, clearly labelled as not the recommended workflow

## 5. GitHub Pages build pipeline

- [x] 5.1 Add `.github/workflows/hugo.yml` using Hugo's officially documented GitHub Pages workflow (checkout, setup Hugo, build, deploy to Pages)
- [ ] 5.2 Enable GitHub Pages in repo settings, source set to "GitHub Actions"
- [ ] 5.3 Push a commit and confirm the workflow runs, deploys, and the live URL serves the home page over HTTPS
- [ ] 5.4 Verify internal links and the `/admin/` bundle resolve correctly under the `/<repo>/` path prefix
- [ ] 5.5 Measure end-to-end latency from a Sveltia commit to the post being visible; confirm it is in the ~30-second range

## 6. Demo dry run and handoff

- [ ] 6.1 Walk through the full editor flow end-to-end: open `/admin/`, log in, create a text-only post, publish, watch it go live
- [ ] 6.2 Repeat with a phone-photo post; confirm alt-text validation blocks publish until alt is filled
- [ ] 6.3 Repeat the OAuth login flow on a second device or browser profile to catch device-specific issues before demo day
- [ ] 6.4 Re-test the full flow the day before the demo on the actual demo machine and network, so a Worker misconfiguration or Cloudflare outage is detected early
- [x] 6.5 Prepare a one-page handoff (in repo `README.md`) covering: how the editor logs in, how to add a post, how to delete a post, who to contact when something looks wrong

## 7. Capture deferred decisions before they are forgotten

- [x] 7.1 Record the production-hosting options (Cloudflare Pages + private repo, or GitHub Pro + private repo) in the README's "Before going live" section, with a check-box gate on private repo before any real content
- [x] 7.2 Record the discovery-channel question (RSS, email digest, WhatsApp link share) as an unresolved item in the same section
- [x] 7.3 Record the custom-domain handover plan with the third-party DNS provider, including the `baseURL` change Hugo will need at switchover
- [x] 7.4 Record OAuth-Worker production-hardening notes (pin Worker code revision, restrict `ALLOWED_DOMAINS` to production host, rotate Client Secret if Worker is shared between demo and production) — the Worker itself is no longer deferred and is set up under task 4.1
