## ADDED Requirements

### Requirement: Posts are stored as Hugo page bundles

The system SHALL represent every post as a Hugo page bundle: a directory under `content/posts/<slug>/` containing an `index.md` file, with any associated image stored as a sibling file inside the same directory.

#### Scenario: New post with an image

- **WHEN** the editor publishes a post titled "Liftonderhoud" with a photo attached
- **THEN** the repository contains a directory `content/posts/<slug>/` with `index.md` and the uploaded image file inside it
- **AND** the image path referenced in `index.md` is relative to the bundle, not an absolute `/static/...` path

#### Scenario: New post without an image

- **WHEN** the editor publishes a post with no image attached
- **THEN** the repository contains a directory `content/posts/<slug>/` with `index.md` only
- **AND** no orphaned image file is created elsewhere in the repository

### Requirement: Frontmatter schema for posts

The system SHALL store post frontmatter with the following fields and constraints: `title` (string, required), `date` (timestamp, set automatically to publish time), `summary` (string, optional), `body` (markdown, the post content), `draft` (boolean, default false), and `cover` (optional object containing `image` and `alt`). The `cover` object name aligns with PaperMod's built-in cover convention so the theme's responsive image rendering applies automatically.

#### Scenario: Cover field is omitted

- **WHEN** the editor publishes a post without attaching an image
- **THEN** the post frontmatter MUST NOT contain a `cover` key, or the `cover` value MUST be empty
- **AND** the post still publishes successfully

#### Scenario: Cover field is provided without alt text

- **WHEN** the editor attaches an image but leaves the alt-text field blank
- **THEN** the CMS MUST prevent publication and display a validation error on the alt-text field
- **AND** no commit is made to the repository

#### Scenario: Cover field is provided with alt text

- **WHEN** the editor attaches an image and provides alt text
- **THEN** the post frontmatter contains a `cover` object with `image` (the bundle-relative file path) and `alt` (the editor's text), and the post publishes successfully

### Requirement: Phone-camera images are resized at build time

The system SHALL resize images larger than the page's content width and SHALL emit responsive variants (e.g. `srcset`) using Hugo's image processing, so that a 4–6 MB phone photo is never served unmodified to a reader.

#### Scenario: A 5 MB phone photo is uploaded

- **WHEN** the editor publishes a post with a 5 MB JPEG from a phone camera
- **THEN** the rendered HTML for that post references resized image variants whose largest dimension is at most the layout's content width
- **AND** the original full-resolution file is not the image actually requested by a typical viewport

### Requirement: Editor surface at /admin/

The system SHALL expose a Sveltia CMS interface at the path `/admin/` on the live site, served as static files from `static/admin/` in the repository.

#### Scenario: Editor opens the CMS

- **WHEN** an authorized user navigates to `<site>/admin/`
- **THEN** the Sveltia CMS loads in the browser and presents a login affordance backed by GitHub
- **AND** no separate hostname or service is required

### Requirement: CMS form mirrors the post schema

The system SHALL configure Sveltia so the "New Post" form exposes exactly the fields defined by the frontmatter schema, with `cover` rendered as an optional object widget grouping `image` and `alt`, where `alt` is required only when `image` is set.

#### Scenario: Editor opens the New Post form

- **WHEN** the editor clicks "New Post" in the CMS
- **THEN** the form shows fields for title, summary, body, draft toggle, and a collapsible image block
- **AND** the date is filled automatically and not presented as a primary editing field

#### Scenario: Editor expands the cover block

- **WHEN** the editor expands the cover block and selects a file
- **THEN** an alt-text field is shown and marked required
- **AND** the editor cannot submit the form until alt text is provided

### Requirement: Editor publishes without using git

The system SHALL allow the editor to create, edit, and publish posts entirely through the Sveltia CMS in a browser, without invoking any git command, opening a terminal, or learning git concepts (branches, merges, conflicts).

#### Scenario: Editor publishes a post

- **WHEN** the editor fills the New Post form and clicks "Publish"
- **THEN** Sveltia commits the post (markdown plus any image) directly to the default branch via the GitHub API
- **AND** the editor receives a confirmation in the CMS UI without seeing any git terminology
