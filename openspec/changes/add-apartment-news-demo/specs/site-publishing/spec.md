## ADDED Requirements

### Requirement: Hugo site builds via GitHub Actions on every push to the default branch

The system SHALL run a GitHub Actions workflow that builds the Hugo site on every push to the repository's default branch and deploys the build output to GitHub Pages.

#### Scenario: A commit lands on the default branch

- **WHEN** a commit is pushed to the default branch (whether from a developer or from Sveltia)
- **THEN** the GitHub Actions workflow runs Hugo and publishes the resulting `public/` directory to GitHub Pages
- **AND** the workflow uses an officially documented Hugo + GitHub Pages action recipe, not a bespoke script

#### Scenario: A commit lands on a non-default branch

- **WHEN** a commit is pushed to any branch other than the default
- **THEN** the production deploy workflow does not publish that commit to the live site

### Requirement: Build-and-publish completes in roughly thirty seconds

The system SHALL complete the full build-and-publish cycle (Sveltia commit → Actions run → Pages serves) quickly enough to support a live demo, with a target of roughly thirty seconds from commit to visible on the site.

#### Scenario: Editor publishes during a demo

- **WHEN** the editor clicks "Publish" in Sveltia
- **THEN** the new post is reachable on the live URL within approximately thirty seconds
- **AND** no manual step (no clicking in the GitHub UI, no rerunning the workflow) is required for it to appear

### Requirement: Site is served from GitHub Pages on the default project URL

The system SHALL host the demo site on GitHub Pages at the default project URL `https://josokw.github.io/<repo>/`, with HTTPS enabled automatically.

#### Scenario: Visitor opens the site

- **WHEN** a visitor opens `https://josokw.github.io/<repo>/`
- **THEN** the home page loads over HTTPS
- **AND** internal links and asset URLs resolve correctly under the `/<repo>/` path prefix

#### Scenario: Hugo baseURL is misconfigured

- **WHEN** the Hugo `baseURL` is set without the repo path prefix
- **THEN** a build-time or visible-link check during initial setup MUST surface the mismatch before the demo
- **AND** the site is not declared demo-ready until internal links resolve under the path prefix

### Requirement: Editor authenticates via GitHub OAuth

The system SHALL authenticate editors through a GitHub OAuth App registered for this site, brokered by Sveltia's hosted authentication proxy, so that no OAuth client secret is embedded in the static site.

#### Scenario: Editor logs in to the CMS

- **WHEN** the editor clicks "Login with GitHub" in the CMS
- **THEN** a GitHub OAuth flow runs via Sveltia's hosted auth proxy and returns a token scoped to the configured GitHub OAuth App
- **AND** the token is stored in the editor's browser only, not committed to the repository

#### Scenario: A user without repo write access tries to log in

- **WHEN** a logged-in GitHub user without write access to the repository attempts to publish via the CMS
- **THEN** the GitHub API rejects the commit and Sveltia surfaces the failure to the user
- **AND** no partial state is committed

### Requirement: Editor access is controlled by GitHub repository collaborators

The system SHALL grant editing rights based on GitHub repository write access, so that adding or removing an editor is a GitHub collaborator change with no site code or configuration changes.

#### Scenario: Adding the actual editor after the demo

- **WHEN** the repo owner invites Kitty's GitHub account as a collaborator with write access
- **AND** Kitty accepts the invitation
- **THEN** Kitty can log in to `/admin/` and publish posts using her own GitHub identity
- **AND** no Sveltia config, OAuth App, or workflow file is modified

#### Scenario: Removing an editor

- **WHEN** the repo owner removes a collaborator's write access
- **THEN** that user can no longer publish via the CMS, even if they retain a previously issued OAuth token
- **AND** no rotation of the repo owner's credentials is required

### Requirement: Demo runs on a public repository; production rollout is gated on a private-repo decision

The system SHALL operate the demo on a public GitHub repository for cost and simplicity, and the project SHALL NOT publish real apartment content (resident names, photos of identifiable residents, building-internal logistics) to that public repository. A production-host decision (Cloudflare Pages plus a private repo, or GitHub Pro plus a private repo) MUST precede any switch to real content.

#### Scenario: Demo content is published

- **WHEN** content is committed during the demo phase
- **THEN** the content consists only of sample or fictional posts with no identifying resident information
- **AND** the repository remains publicly readable

#### Scenario: Real content is proposed before the host decision is made

- **WHEN** any contributor attempts to commit a post containing real resident names, real photos of residents, or building-internal logistics while the repository is public
- **THEN** that commit MUST be blocked or reverted, and the production-host decision MUST be resolved first

### Requirement: Deferred concerns are documented, not implemented

The system SHALL leave the following concerns explicitly out of scope for this change while documenting them so they survive the gap to production: production hosting (private repo, paid plan or alternate host), custom-domain wiring via the third-party DNS provider, content-discovery channel for tenants (RSS, email digest, link-sharing), and self-hosted OAuth proxy.

#### Scenario: Stakeholder asks about production readiness

- **WHEN** a stakeholder asks during or after the demo whether the site is production-ready
- **THEN** the deferred concerns above are pointed to as unresolved prerequisites
- **AND** none of them is silently implemented as part of this change
