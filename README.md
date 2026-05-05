# Appartement Nieuws

Een kleine statische nieuwssite voor de bewoners van een appartementencomplex. Posts worden geschreven in het Nederlands door één redacteur via een browser-formulier; de site wordt gebouwd met Hugo en gehost op GitHub Pages.

Demo-URL: `https://josokw.github.io/appartement-nieuws/`

## Stack

- **Hugo** met de **PaperMod**-thema (geïmporteerd als Hugo Module)
- **Sveltia CMS** op `/admin/` als redactie-interface
- **GitHub Pages** + **GitHub Actions** voor build en hosting
- **GitHub OAuth** voor de login van de redacteur

## Lokale ontwikkeling

```sh
hugo server -D            # bouwt en serveert lokaal op http://localhost:1313/
hugo --gc --minify        # productie-build, output in ./public/
```

## Architectuur

```
content/posts/<slug>/index.md      # bericht (TOML-frontmatter)
content/posts/<slug>/foto.jpg      # eventuele foto, naast de index.md
static/admin/index.html            # Sveltia-shell, geserveerd op /admin/
static/admin/config.yml            # Sveltia-configuratie (collecties, velden)
.github/workflows/hugo.yml         # build + deploy naar GitHub Pages
hugo.toml                          # Hugo-configuratie inclusief PaperMod-params
```

Posts zijn **page bundles**: een map per bericht met `index.md` en eventueel een afbeelding ernaast. Sveltia plaatst geüploade afbeeldingen automatisch in de juiste map. Hugo schaalt grote foto's bij build via de PaperMod cover-pipeline en levert responsive `srcset`-varianten.

## Frontmatter-schema (zie `static/admin/config.yml`)

| Veld      | Type            | Verplicht? | Toelichting                                                  |
| --------- | --------------- | ---------- | ------------------------------------------------------------ |
| `title`   | string          | ja         | Titel van het bericht                                        |
| `date`    | datetime        | auto       | Wordt automatisch op publicatiemoment gezet                  |
| `summary` | tekst           | nee        | Eén zin, zichtbaar in de lijst                               |
| `cover`   | object          | nee        | Optionele foto. Indien aanwezig: `image` én `alt` verplicht. |
| `cover.image` | bestand     | binnen object | Geüpload bestand, opgeslagen in de page-bundle map        |
| `cover.alt`   | string      | binnen object | Verplichte alt-tekst, minstens 3 tekens                   |
| `body`    | markdown        | ja         | De inhoud van het bericht                                    |
| `draft`   | boolean         | nee        | Standaard `false`. Concepten worden niet gepubliceerd.       |

## Voor Jos: setup-checklist (eenmalig)

Volg deze stappen in volgorde. Aanvinken na elke stap zodat niets wordt overgeslagen.

### 1. Repository op GitHub

- [ ] Maak een **publieke** repository onder `josokw` aan met de naam `appartement-nieuws`.
- [ ] Push de inhoud van deze map naar `main`:
  ```sh
  git add .
  git commit -m "initial scaffold: hugo + papermod + sveltia"
  git remote add origin git@github.com:josokw/appartement-nieuws.git
  git push -u origin main
  ```

### 2. Cloudflare Worker voor OAuth deployen

Sveltia is een statische SPA en kan zelf geen OAuth-secret bewaren. **Sveltia heeft géén gehoste OAuth-proxy** — iedere Sveltia-site draait een eigen Cloudflare Worker, zowel voor de demo als voor productie. De Worker is een single-file deploy via Cloudflare's gratis tier.

- [ ] Maak een gratis Cloudflare-account aan als je er nog geen hebt.
- [ ] Open `https://github.com/sveltia/sveltia-cms-auth` en klik op de **Deploy with Workers**-knop in de README. Dit fork't de auth-Worker en deployt hem.
- [ ] Noteer de Worker-URL na de deploy, formaat: `https://sveltia-cms-auth.<SUBDOMAIN>.workers.dev`.

### 3. GitHub OAuth App registreren

- [ ] Ga naar `https://github.com/settings/developers` → **OAuth Apps** → **New OAuth App**.
- [ ] Vul in:
  - **Application name**: `Appartement Nieuws CMS`
  - **Homepage URL**: `https://josokw.github.io/appartement-nieuws/`
  - **Authorization callback URL**: `<Worker-URL>/callback` (let op: het pad `/callback` is verplicht)
- [ ] Sla de **Client ID** op en genereer een **Client Secret**; kopieer beide direct want het secret wordt maar één keer getoond.

### 4. Worker-secrets configureren

- [ ] Ga in het Cloudflare-dashboard naar de zojuist gedeployde Worker → **Settings → Variables**.
- [ ] Voeg toe (klik **Add variable**):
  - `GITHUB_CLIENT_ID` — Client ID uit stap 3 (plain text)
  - `GITHUB_CLIENT_SECRET` — Client Secret uit stap 3 (vink **Encrypt** aan)
  - `ALLOWED_DOMAINS` — `josokw.github.io` (begrenst welke sites deze Worker mogen gebruiken)
- [ ] **Save and deploy** in Cloudflare.

### 5. Sveltia-config bijwerken en pushen

- [ ] Open `static/admin/config.yml`.
- [ ] Vervang de placeholder in `base_url` door de Worker-URL uit stap 2 (zonder pad-suffix).
- [ ] Commit en push naar `main`. De GitHub Actions-workflow herbouwt en deployt de site.

### 6. GitHub Pages aanzetten

- [ ] Ga in de repo naar **Settings → Pages**.
- [ ] Bij **Source** kies **GitHub Actions**.
- [ ] Push een commit naar `main`. De workflow `.github/workflows/hugo.yml` draait automatisch en publiceert de site.
- [ ] Verifieer dat de site opent op `https://josokw.github.io/appartement-nieuws/`.

### 7. Verificatie

- [ ] Open `https://josokw.github.io/appartement-nieuws/admin/` in een browser.
- [ ] Klik op **Login with GitHub** en doorloop de OAuth-flow.
- [ ] Maak een testbericht aan, druk op **Publish**, en wacht tot de site is herbouwd (~30 seconden).
- [ ] Controleer dat het bericht zichtbaar is op de homepage.

## Voor Jos: Kitty toevoegen na de demo

Eenmalige stappen, geen wijzigingen aan de site nodig:

- [ ] Vraag Kitty een eigen GitHub-account aan te maken (gratis).
- [ ] Ga in de repo naar **Settings → Collaborators** en stuur een uitnodiging naar haar account.
- [ ] Kitty accepteert de uitnodiging via de e-mail.
- [ ] Kitty opent `/admin/` en logt in met haar eigen GitHub-account.
- [ ] Indien gewenst: stuur haar het korte handleiding-document hieronder.

Een aparte account voor Kitty is bewust gekozen: commits zijn herleidbaar tot haar, toegang kan met één klik worden ingetrokken, en bij compromittering blijft de schade beperkt tot deze ene repo.

## Voor Kitty: berichten plaatsen, bewerken, verwijderen

1. Ga naar `https://josokw.github.io/appartement-nieuws/admin/`.
2. Klik op **Login with GitHub** en log in.
3. **Nieuw bericht**: klik op **Berichten → New Bericht**, vul het formulier in, klik op **Publish**. Het bericht staat na ongeveer 30 seconden op de site.
4. **Bestaand bericht aanpassen**: kies het bericht in de lijst, pas aan, klik op **Publish**.
5. **Bericht verwijderen**: kies het bericht en klik op **Delete**.
6. **Concept opslaan zonder publiceren**: vink het vakje **Concept** aan en klik op **Save**.

Bij vragen of als er iets mis lijkt: neem contact op met Jos.

## Break-glass: PAT-login (alleen als OAuth-proxy uitvalt)

Als de OAuth-proxy onbereikbaar is en er nu *echt* gepubliceerd moet worden, kan Sveltia tijdelijk via een **Personal Access Token** worden gebruikt. **Dit is geen aanbevolen werkwijze** — tokens verlopen, moeten beheerd worden, en de redacteur ziet techniek die ze niet zou hoeven zien. Alleen als noodscenario.

- [ ] Maak in GitHub een PAT aan met `repo`-scope.
- [ ] Plak het token in Sveltia's loginscherm wanneer daarom wordt gevraagd.
- [ ] Verwijder het token zodra de OAuth-proxy weer beschikbaar is.

## Demo-content: vóór de demo

De repo bevat vier sample-berichten zonder foto. Voor de demo is het waardevol om:

- [ ] Eén bestaand bericht uit te breiden met een echte foto (bijvoorbeeld via Sveltia tijdens de dry-run), zodat het cover-image-pad daadwerkelijk wordt getest.
- [ ] De `srcset`-output in de gebouwde HTML te controleren: bekijk `public/posts/<slug>/index.html` na een lokale `hugo`-build en bevestig dat er meerdere afbeeldingsvarianten worden gerefereerd, niet één origineel groot bestand.

## Vóór de site live gaat met echte content — productie-checklist

De demo draait op een **publieke** repository. Echte content (bewonersnamen, foto's, interne logistiek) mag pas worden gepubliceerd nadat de productie-host is gekozen en ingeregeld.

### Hosting

Kies één van:

- [ ] **Optie A**: upgrade `josokw` naar GitHub Pro (~€4/maand) en zet de repo op private. GitHub Pages werkt op een private repo binnen Pro.
- [ ] **Optie B**: zet de repo op private (gratis) en verhuis de hosting naar **Cloudflare Pages**. Cloudflare bouwt Hugo native, gratis, en haalt de bron uit een private GitHub-repo via OAuth-koppeling.

### OAuth-hardening

De Cloudflare Worker is vanaf dag één al self-hosted (Sveltia heeft geen gehoste alternatief). Productie-hardening hier beperkt zich tot:

- [ ] Pin de Worker-code aan een vaste revisie van `sveltia/sveltia-cms-auth` zodat ongeplande upstream-wijzigingen niet doorkomen.
- [ ] Scherm `ALLOWED_DOMAINS` af tot het productiedomein zodra dat is ingesteld; verwijder de demo-host als die niet meer in gebruik is.
- [ ] Roteer het GitHub OAuth App Client Secret als demo en productie dezelfde Worker delen.

### Custom domein

Het domein wordt beheerd door een derde partij.

- [ ] Bepaal het gewenste domein (bijvoorbeeld `nieuws.<gebouw>.nl`).
- [ ] Stel een CNAME of A/AAAA-record in bij de DNS-provider, wijzend naar de gekozen host (GitHub Pages of Cloudflare Pages).
- [ ] Voeg het domein toe in de host-instellingen en wacht op HTTPS-certificering.
- [ ] Pas `baseURL` in `hugo.toml` aan naar het nieuwe domein (zonder pad-prefix).
- [ ] Pas `backend.repo`, `site_url` en `display_url` in `static/admin/config.yml` aan indien nodig.

### Discovery — hoe weten bewoners dat er nieuws is?

Open vraag, niet blokkerend voor de demo, maar moet vóór livegang beantwoord worden. Opties:

- [ ] **RSS-feed**: aanwezig op `/index.xml` (Hugo doet dit automatisch). Werkt voor tech-savvy bewoners; de meesten gebruiken geen RSS.
- [ ] **E-mail-digest**: een dienst zoals Buttondown of Mailchimp die de RSS-feed leest en automatisch een mail verstuurt. Geringe maandkost.
- [ ] **WhatsApp- of Telegram-link delen**: handmatig de URL van een nieuw bericht in de bestaande bewonersgroep plaatsen. Lage drempel, geen extra dienst.
- [ ] **QR-code op het mededelingenbord**: minder geschikt voor dagelijks nieuws.

Aanbevolen onderzoek: vraag Kitty wat haar voorkeur heeft. Het home-page-ontwerp (lijst recent / één laatste / gedateerd archief) volgt uit deze keuze.

## Status

Wat in deze repo al is opgeleverd:

- Hugo-site met PaperMod (Hugo Module).
- Sveltia-configuratie met Nederlandse labels en een optionele cover-foto met verplichte alt-tekst.
- Vier text-only sample-berichten als page bundles.
- GitHub Actions-workflow die op iedere push naar `main` bouwt en naar Pages deployt.
- Deze README met de stappen die nog door Jos op github.com en Cloudflare moeten worden uitgevoerd.

Wat nog door Jos gedaan wordt:

- Repo aanmaken op GitHub onder `josokw` en push uitvoeren.
- Cloudflare Worker (`sveltia/sveltia-cms-auth`) deployen voor OAuth.
- GitHub OAuth App registreren en de Worker-secrets configureren.
- `base_url` in `static/admin/config.yml` bijwerken naar de Worker-URL en pushen.
- GitHub Pages aanzetten in de repo-instellingen.
- Demo-dry-run, inclusief publicatie van een bericht met echte foto.
- Kitty toevoegen na de demo.
