# Projekt: Ørehænger (arbejdstitel) — dansk sang-gættespil

> Denne fil læses automatisk hver gang Claude arbejder i projektet.
> Alt hvad Claude skal vide uden at blive spurgt, står her.
> Ret i den. Opdatér den når noget ændrer sig.
> Ting i [KANTEDE PARENTESER] mangler stadig at blive besluttet.

## Hvad det er

Et webspil hvor man gætter danske sange ud fra et lydklip der bliver
længere for hvert forkert gæt.

To tilstande:
- **DAGENS SANG** — fem forskellige sange om dagen, samme for alle, ét
  samlet delbart resultat efter alle fem. Bygget (`src/dagens-sang.html`,
  2026-08-30).
- **BIBLIOTEKET** — fri leg, ubegrænsede runder, filtre på årti og genre.
  Ikke bygget endnu. Data findes nu (årti/genre genereres automatisk af
  fase 2-pipelinen, se "Tekniske rammer") — kan tages op til revurdering.

Forbilleder: songspot.net (klip der vokser) og bandle.app (dagligt format).
Vi kopierer ikke deres design.

Navnet **Ørehænger** er en arbejdstitel (valgt 2026-08-30) — Emil vil gerne
kunne genoverveje det senere. Domæne er endnu ikke valgt.

**Status:** Projektet ligger endnu kun her i Claude-projektet, ikke i et
rigtigt GitHub-repo. Se `github-opsaetning.md` for hvordan Emil lægger det
op, så pipelinen (se nedenfor) begynder at køre automatisk.

## Hvem det er til

Danskere i alle aldre der kender dansk musik fra radioen, fester og
barndommen. Mobil først — det bliver spillet i sofaen og sendt videre i en
gruppechat.

## Spilregler

- **Dagens sang er fem sange, ikke én.** Hver sang spilles for sig med sin
  egen voksende klip-runde (se nedenfor), i træk. Efter alle fem får man
  ét samlet, delbart resultat.
- **Gæt per sang:** 5 forsøg, ét per klip-trin (0,1 / 0,5 / 2 / 8 /
  15 sekunder). Forkert gæt eller "spring over" rykker automatisk til
  næste, længere klip. Efter det femte forkerte gæt afsløres sangen.
  Gæt matcher også kendte alternative stavemåder af titlen (fx uden
  apostrof), ikke kun den officielle titel — se `alternative_titler` i
  "Tekniske rammer".
- **Ingen mellemresultat mellem sangene.** Man ser IKKE hvor mange forsøg
  en sang tog, eller dens grid/links, lige efter den er slut — man går
  direkte videre til næste sang. Kun det samlede overblik for alle fem
  sange (grid + links per sang) vises til sidst. Emil valgte dette
  fremfor en mellemskærm per sang (2026-08-30) — se beslutninger.md.
- **Klippets startpunkt er fast og ens for alle spillere** — ikke altid
  sekund 0. Generator-pipelinen (se "Tekniske rammer") finder automatisk
  hvor stilheden i starten af Apples preview slutter, for hver sang, og
  gemmer det i data. Et tidligere forsøg med spiller-justerbare knapper i
  selve spillet blev bygget og fjernet igen (2026-08-30): det gav mening
  for Emil alene, men ikke i et spil man deler, hvor alle skal opleve
  samme klip. Se beslutninger.md, 2026-08-30.
- **Dagens sange — hvornår de skifter:** deterministisk ud fra dagens dato
  i dansk tid (`Intl.DateTimeFormat` med `timeZone: "Europe/Copenhagen"`,
  håndterer selv sommertid). 2026-08-30 = "dag 1". Ingen backend
  nødvendig — se beslutninger.md, 2026-08-30.
- **Dagens sange — kun én gang:** det samlede resultat gemmes i
  `localStorage` (ikke en cookie). Prøver man igen samme dag, vises det
  gemte resultat i stedet for at man kan spille om.
- **Delt resultat:** emoji-grid i Wordle/Heardle-stil — én linje per sang,
  🟩 for det klip-trin man gættede rigtigt på, 🟥 for brugte forkerte trin,
  ⬜ for trin der ikke blev brugt, plus en facit-linje ("Gættet: 3/5
  sange"). Ren tekst, ingen billede. Vises kun i det samlede overblik.
- **Links per sang:** Apple Music (den faktiske lydkilde) og et
  Spotify-søgelink (bekvemmelighed, ikke lydkilde — se "Tekniske rammer").
  Vises kun i det samlede overblik, ikke undervejs.

## Hvad "dansk sang" betyder i dette projekt

**Dansk kunstner, uanset sprog.** Fx tæller Lukas Graham (engelsk tekst) og
Aqua (dansk gruppe) med, fordi kunstneren/gruppen er dansk. Se
beslutninger.md, 2026-08-30.

## Claudes rolle

- Emil er designer, ikke udvikler. Han læser ikke kode.
- Forklar altid HVAD en ændring gør for spilleren — ikke hvordan den er
  implementeret. Hold det på 3–5 linjer.
- Foreslå aldrig at han "lige kører en kommando lokalt". Alt skal kunne
  gøres fra Cowork eller direkte på GitHub.
- Når et valg har konsekvenser (koster penge, låser os til en leverandør,
  rører rettigheder) — STOP og spørg først.
- Er du i tvivl om noget der handler om smag: spørg.
  Er du i tvivl om noget teknisk: vælg det simpleste og sig hvad du valgte.
- Svar på dansk.
- Bed aldrig Emil om at beskrive eller citere sangtekst (rettighedshensyn).
  Skal noget rettes ud fra hvad en sang faktisk lyder som (fx et
  startpunkt), skal løsningen gælde alle spillere ens og helst automatisk
  (lydanalyse) — ikke en samtale om teksten, og ikke en knap kun Emil
  bruger i det delte spil (se beslutninger.md, 2026-08-30).

## Tekniske rammer

- **Stack:** Statisk HTML/CSS/JavaScript i én fil per spiltilstand (se
  "Vigtige filer"). Ingen backend der kører/svarer på requests, ingen
  database, ingen framework. Bekræftet i fase 1-prototypen (2026-08-30).
- **Datapipeline (fase 2, bygget 2026-08-30):** `data/sange.json`
  genereres af `scripts/generer-sange-json.js`, kørt automatisk af
  GitHub Actions (`.github/workflows/generer-sange-json.yml`) hver gang
  `data/sange.csv` ændres. Dette ER et build-trin/en CI-robot — en
  bevidst, velovervejet undtagelse fra "ingen backend, ingen build-trin"
  (se beslutninger.md, 2026-08-30): der kører ingen server, kun et
  engangs-script der starter ved en git-push og stopper igen. Emil valgte
  dette fremfor et manuelt browser-værktøj, efter selv at have spurgt om
  det ville blive bøvligt med mange sange og løbende opdateringer.
  Scriptet slår kun NYE sange op — sange der allerede findes i
  `sange.json`, slås ikke op igen — så det forbliver hurtigt uanset hvor
  stor puljen bliver. `data/sange.csv` er stadig kilden Emil selv
  redigerer; `data/sange.json` er et genereret resultat og skal ALDRIG
  rettes i hånden (håndrettelser bliver overskrevet ved næste kørsel, men
  bevares hvis sangen allerede er i cachen — se scriptets kommentarer).
- **Startpunkt (`start_offset_ms`):** findes automatisk af pipelinen ved
  at analysere lydklippet med ffmpeg (`silencedetect`) og springe evt.
  stilhed i starten over. Ingen sangtekst involveret, ingen manuel
  lytning nødvendig af hverken Emil eller Claude.
- **Lydkilde:** Apples iTunes Search API (gratis, ingen nøgle, `country=DK`),
  slået op af pipelinen — ikke længere live i browseren. Deezer som
  reserve for sange Apple ikke har.
- **Spotify:** IKKE en lydkilde (deres `preview_url` er lukket for nye apps
  siden november 2024). Bruges kun som et simpelt søgelink
  (`open.spotify.com/search/…`) ved siden af Apple Music — ingen konto,
  ingen nøgler. Et præcist Spotify-link til den nøjagtige optagelse ville
  kræve en hemmelig API-nøgle, og den nøgle må ALDRIG ligge i spillets
  egen kode. Se beslutninger.md, 2026-08-30.
- Ingen nye dependencies uden at spørge først.
- Spillet skal virke uden login og uden cookies.
- `src/dagens-sang.html` skal åbnes via et rigtigt link (fx GitHub Pages),
  ikke ved at dobbeltklikke filen lokalt — den henter `data/sange.json`
  med `fetch()`, som ikke virker over `file://`. Se `github-opsaetning.md`.
- Al UI-tekst på dansk. Æ, Ø og Å skal virke overalt — også i søgefeltet,
  i gættematchning og i delte resultater.
- Mobil først. Test altid layoutet på 390px bredde.
- Dagens sang skifter ved midnat **dansk tid**, ikke UTC. Husk sommertid.

## Rettigheder — ikke til forhandling

- Vi hoster ALDRIG musikfiler selv.
- Lyd afspilles kun fra lydkildens officielle preview-URL'er.
- Hver sang linker tilbage til kilden.
- Vi cacher eller downloader ikke lydfiler (den eneste undtagelse:
  generator-scriptet downloader et klip MIDLERTIDIGT under kørsel, kun
  for at måle startpunktet, og sletter filen igen med det samme).
- Der skal være en synlig kontaktadresse så en rettighedshaver kan bede om
  at få en sang fjernet.
- Hvis en opgave kræver at bryde en af disse regler: sig det, gør det ikke.

## Sådan arbejder vi

- Én ting ad gangen. Én branch, én ændring, én pull request.
- Læg en plan først ved alt der er større end en knap. Byg først når planen
  er godkendt.
- Vis altid resultatet som noget Emil kan ÅBNE — en side — ikke en
  beskrivelse af hvad du har gjort.
- Skriv beslutninger ind i `docs/beslutninger.md` når de er truffet.
- Ideer vi ikke bygger nu ryger i `docs/ideer.md`, ikke ind i koden.
- Opdatér denne fil når rammerne ændrer sig.

## Sådan må du IKKE gøre

- Ingen placeholder-indhold ("Sang 1", "Kunstner A"). Brug rigtige sange fra
  `data/sange.json`.
- Byg ikke funktioner der ikke er bedt om, heller ikke "mens du er i gang".
- Skriv ikke om til et andet framework fordi det er pænere.
- Slet ikke sange fra datafilen uden at spørge.
- Brug ikke engelske ord i UI'et hvor der findes et dansk.
- Ret aldrig `data/sange.json` i hånden — ret i `data/sange.csv` og lad
  pipelinen generere resten.

## Vigtige filer

- `data/sange.json` — selve spillets datafil. Genereres automatisk, IKKE
  redigeret i hånden. Indeholder lyd-link, årstal, genre, startpunkt og
  alternative stavemåder for hver sang.
- `data/sange.csv` — Emils arbejdsliste, den eneste fil han selv
  redigerer. Kilden pipelinen læser fra. Ti eksempelsange lige nu.
- `scripts/generer-sange-json.js` — genererer `sange.json` fra `sange.csv`.
  Slår op hos Apple, finder startpunkt med ffmpeg. Kører normalt via
  GitHub Actions, kan også køres manuelt.
- `.github/workflows/generer-sange-json.yml` — kører scriptet ovenfor
  automatisk hver gang `data/sange.csv` ændres, og gemmer resultatet.
- `docs/beslutninger.md` — hvorfor tingene er som de er.
- `docs/ideer.md` — ting vi bevidst ikke bygger endnu.
- `design/noter.md` — Emils designbeslutninger.
- `github-opsaetning.md` — engangsguide til at lægge projektet på GitHub,
  så pipelinen begynder at køre.
- `src/prototype.html` — fase 1-prototypen. Fem sange i træk, ingen
  dagens-sang-struktur. Rørt ikke længere — historisk reference.
- `src/dagens-sang.html` — dagens sang-tilstanden. Henter `data/sange.json`
  ved opstart (ikke længere live iTunes-opslag), deterministisk dagligt
  valg af fem sange, ét spilleforsøg om dagen, ingen mellemresultat mellem
  sangene, samlet emoji-grid til deling, Apple Music + Spotify-søgelink
  per sang. Biblioteket findes ikke i denne fil endnu.
