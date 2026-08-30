# Sådan lægger du projektet på GitHub, så automatikken virker

Denne guide er kun nødvendig én gang. Bagefter kører opdateringen af
`data/sange.json` helt af sig selv, hver gang du tilføjer en sang til
`data/sange.csv`.

## 1. Opret et repo

Gå til [github.com](https://github.com) (opret en gratis konto hvis du ikke
har en). Klik **"New repository"**. Giv det et navn — fx `oerehaenger` —
og opret det (privat eller offentligt, som du vil — begge dele virker med
det der står i denne guide).

## 2. Læg filerne ind

Åbn det nye, tomme repo og klik **"Add file" → "Upload files"**. Træk hele
mappestrukturen ind i uploadfeltet — det er vigtigt at mappestien følger
med, ikke kun filnavnene:

```
CLAUDE.md
README.md
docs/beslutninger.md
docs/ideer.md
design/noter.md
data/LÆS-MIG.md
data/sange.csv
src/prototype.html
src/dagens-sang.html
scripts/generer-sange-json.js
.github/workflows/generer-sange-json.yml
```

GitHub's upload-felt kan godt tage hele mapper ad gangen (træk fx hele
`src`-, `data`-, `scripts`- og `.github`-mapperne ind én for én) — det
bevarer stierne automatisk. `.github` er en mappe hvis navn starter med et
punktum; nogle styresystemer skjuler den i Finder/Stifinder, men det
generer ikke noget når du trækker mappen direkte ind i browseren. Bekræft
uploaden med **"Commit changes"** nederst.

Du behøver ikke selv at oprette `data/sange.json` — den bliver oprettet af
robotten første gang, lige så snart `data/sange.csv` ligger i repoet.

## 3. Tjek at Actions er slået til

Klik på fanen **"Actions"** øverst i repoet. Er den slået fra, viser GitHub
en knap i stil med *"I understand my workflows, go ahead and enable
them"* — klik den. Er den allerede slået til (det er den som regel som
standard), sker der ikke noget synligt her, og du kan bare gå videre.

## 4. Se robotten køre

Så snart `data/sange.csv` er uploadet (trin 2 udløser det automatisk),
begynder robotten at arbejde. Under fanen **Actions** kan du følge med —
det tager typisk et par minutter for ti sange (den skal både spørge Apple
om hver sang OG lytte til klippet for at finde et godt startpunkt). Når
den er færdig, ligger et nyt, udfyldt `data/sange.json` i repoet.

Hvis en sang ikke kan findes hos Apple (fx en stavefejl), springer robotten
den sang over og skriver en advarsel i Actions-loggen — resten af sangene
kommer stadig med. Ret stavningen i `sange.csv` og læg filen ind igen, så
prøver den automatisk igen for netop den sang.

## 5. Få et link du kan åbne spillet fra

For at `dagens-sang.html` kan hente `data/sange.json` med det samme,
skal siden åbnes via et rigtigt link — ikke ved at dobbeltklikke filen
lokalt. Den nemmeste, gratis løsning er **GitHub Pages**, indbygget i
samme repo:

1. Gå til **Settings → Pages** i repoet.
2. Under "Build and deployment", vælg **"Deploy from a branch"**.
3. Vælg branchen **main** og mappen **/ (root)**. Gem.
4. Efter et minuts tid får du et link i stil med
   `https://<dit-brugernavn>.github.io/<repo-navn>/src/dagens-sang.html`
   — det er linket du åbner for at spille.

## Når du tilføjer flere sange senere

Bare ret i `data/sange.csv` direkte på GitHub (åbn filen, klik blyantikonet
"Edit", tilføj en linje, commit) — robotten opdaterer automatisk
`data/sange.json` inden for et par minutter. Du skal ikke gøre andet.
