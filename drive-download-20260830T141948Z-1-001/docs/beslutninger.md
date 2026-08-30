# Beslutninger

Én linje per beslutning: **dato — hvad vi valgte — hvorfor — hvad vi fravalgte.**
Nye beslutninger nederst. Ret aldrig en gammel linje — tilføj en ny der
ophæver den.

---

**2026-08-30 — Spillet kombinerer dagens sang og et frit bibliotek.**
Dagens sang giver grunden til at komme tilbage og det der bliver delt;
biblioteket giver noget at lave når man har spillet dagens.
Fravalgt: kun dagligt (for lidt indhold), kun frit spil (ingen grund til at
vende tilbage).

**2026-08-30 — Spotify bruges ikke som lydkilde.**
Feltet `preview_url` blev lukket for nye apps i november 2024; kun apps der
allerede var godkendt før den dato har adgang. Kan stadig bruges til
metadata og "åbn i Spotify"-links senere.

**2026-08-30 — Apples iTunes Search API er førstevalg til lyd.**
Gratis, kræver ingen nøgle, giver 30-sekunders previews for stort set alt
kommercielt udgivet, kan slås op med `country=DK`.
Betingelse: Apples vilkår forventer at vi linker tilbage til Apple Music.
Reserve: Deezers åbne API for sange Apple ikke har.

**2026-08-30 — Vi hoster ikke lyd selv.**
Juridisk langt tungere position, og ikke nødvendigt.

**2026-08-30 — Koda kontaktes efter prototypen, ikke før.**
Vi beskriver præcis hvad vi gør (30-sekunders previews afspillet fra
tredjeparts API, ingen hosting af lyd) og får et konkret svar.
Status: **ikke gjort endnu.**

**2026-08-30 — Fase 1-prototypen er én statisk HTML-fil, bygget med fem
sange hentet direkte fra `data/sange.csv` (ikke via `data/sange.json`, som
stadig er tom indtil fase 2).**
Hurtigste vej til at teste kernemekanikken uden at vente på
JSON-generatoren. Fravalgt: at bygge fase 2's data-pipeline først.

**2026-08-30 — Browserbaserede opslag mod Apples iTunes Search API skal
bruge JSONP (callback-parameter + dynamisk `<script>`-tag), ikke
fetch()/XHR.**
Apples endpoint sender ikke en `Access-Control-Allow-Origin`-header, og
Apples egen dokumentation kræver JSONP til cross-site-opslag — almindeligt
fetch() blev blokeret af CORS i alle browsere under test. Fravalgt: en
backend-proxy (unødvendig kompleksitet, og strider mod "ingen backend før
vi har brug for den").
**Ophævet 2026-08-30 (se pipeline-beslutningen nedenfor): fase 2 slår ikke
længere op live i browseren, så JSONP bruges ikke længere i selve spillet.**

**2026-08-30 — Klippene i fase 1 starter fra begyndelsen af Apples
30-sekunders preview, uden manuelt sat startpunkt — og det er testet til at
virke fint.**
Emil bekræftede at 0,1-sekunds-klippet ikke ramte midt i en linje for de
fem testede sange (Barbie Girl, Kun for mig, Sleeping My Day Away, Bag
duggede ruder, Costa Kalundborg). Løser en af de åbne usikkerheder i denne
fil. Vi tilføjer ikke et manuelt startpunkt (`start_offset_ms`) medmindre
en fremtidig sang viser sig at have et preview der starter akavet.

**2026-08-30 — Gættefeltet i fase 1 er et søgefelt med live-forslag (efter
mønster fra songspot.net), der matcher på både titel og kunstner og ikke
viser noget før man begynder at skrive.**
Hurtigere end at skrive hele titlen, og forbereder til et rigtigt bibliotek
med mange sange. Fravalgt: fritekst uden hjælp, og en fuld dropdown der
viser alle sange på forhånd.

**2026-08-30 — "Dansk sang" betyder: dansk kunstner, uanset sprog.**
Bredeste og mest naturlige fortolkning for et publikum der tænker i
kunstnere og situationer, ikke i sprogstatistik. Fravalgt: kun dansksungne
sange (udelukker fx Lukas Graham), og "hits i Danmark uanset nationalitet"
(gør puljen for bred og fjerner "dansk" som identitet).

**2026-08-30 — Man får 5 gæt per sang, ét per klip-trin.**
Matcher de fem klip-trin der allerede er bygget (0,1/0,5/2/8/15 sek).
Forkert gæt eller spring over rykker automatisk til næste, længere klip.
Fravalgt: ubegrænsede gæt på samme klip (Songspot-stil) — mindre
Wordle-følelse, og fem trin er allerede en naturlig grænse.

**2026-08-30 — Delt resultat bliver et emoji-grid, Wordle/Heardle-stil.**
Ren tekst, deles hvor som helst uden billede — viser hvilket af de fem
klip-trin man gættede rigtigt på (eller at man ikke gættede den).
Fravalgt: en tekstsætning (mindre genkendeligt/delbart) og et genereret
billede (mere arbejde, ikke nødvendigt endnu).

**2026-08-30 — Arbejdstitel: Ørehænger.**
Det danske ord for en sang der sidder fast i hovedet — rammer præcis
konceptet (kendt omkvæd, glemt titel). Markeret som arbejdstitel; Emil vil
gerne kunne genoverveje det senere. Domæne er endnu ikke valgt.

**2026-08-30 — Dagens sang er bygget som ny fil `src/dagens-sang.html`,
adskilt fra fase 1-prototypen (`src/prototype.html`, som ikke er rørt).**
Så Emil stadig kan åbne den oprindelige fase 1-test, mens dagens-sang
udvikles videre for sig selv. Fravalgt: at overskrive prototype.html.

**2026-08-30 — Biblioteket bygges ikke endnu.**
CLAUDE.md beskriver filtre på årti og genre, men den data findes ikke i
`sange.csv` endnu — den "hentes automatisk i fase 2" ifølge
`data/LÆS-MIG.md`. Emil valgte at vente med Biblioteket til data findes,
frem for at bygge det uden filtre nu. Genbesøges når sange.json er beriget
i fase 2.

**2026-08-30 — Dagens sang trækker (indtil videre) fra alle ti
eksempelsange i `data/sange.csv`, ikke kun de fem fra fase 1-testen.**
Dobbelt så stor pulje uden at vente på at Emil skriver flere sange ind.
Ulempe, sagt højt: med kun ti sange gentager "dagens sang" sig hver 10.
dag — tyndt, men nok til at teste selve strukturen. Fravalgt: at vente til
`sange.csv` har flere sange (LÆS-MIG.md foreslår 30 som minimum).

**2026-08-30 — Dagens sang vælges deterministisk ud fra dagens dato i
dansk tid: dag-nummer modulo antal sange i puljen, med 2026-08-30 som
"dag 1".**
Kræver ingen backend eller database — alle browsere regner samme sang ud
for samme dato. Bruger `Intl.DateTimeFormat` med `timeZone: "Europe/
Copenhagen"`, som selv håndterer sommertid/vintertid korrekt (testet på
DST-skiftet i oktober). Fravalgt: en tilfældig, serverstyret trækning
(kræver en backend, som vi ikke har brug for endnu).

**2026-08-30 — Dagens sang kan kun spilles én gang om dagen; resultatet
gemmes i browserens `localStorage` (ikke en cookie, intet login).**
Matcher Wordle-mekanikken og reglen om at spillet skal virke uden login/
cookies. Ulempe: rydder man browserdata, kan man spille dagens sang igen.
Ikke et problem værd at løse nu. Fravalgt: ubegrænset genspil af dagens
sang.

**2026-08-30 — Dagens sang blev ændret fra én sang til fem forskellige
sange, hver med sin egen voksende klip-runde — ét samlet resultat
(emoji-grid med én linje per sang) deles efter alle fem.**
Emil kan bedre lide følelsen fra fase 1's fem sange i træk end kun én sang
om dagen. Fravalgt: at blive ved kun én sang om dagen.

**2026-08-30 — Med kun ti sange i puljen går "dagens 5 sange" igennem hele
puljen hver 2. dag.**
Samme afvejning som beslutningen om at bruge alle ti eksempelsange ovenfor,
nu bare mere udtalt fordi der bruges fem ad gangen. Emil valgte at bygge
strukturen nu alligevel. Retter sig selv når puljen vokser i fase 2.

**2026-08-30 — Hver sang linker til Spotify med et søgelink
(open.spotify.com/search/…), ikke et præcist link til den nøjagtige
optagelse.**
Et præcist Spotify-link kræver en hemmelig API-nøgle, og den nøgle må
aldrig ligge i selve spillets kode (synlig for alle via "vis kildekode" —
risiko for misbrug af Spotify-adgangen). Et præcist link ville kræve et
engangs-opslag uden for spillet, en gratis Spotify-udvikler-konto og
nøgle-deling. Emil valgte søgelinket for at undgå den kompleksitet.
Fravalgt: at bygge det præcise link nu. Kan tages op igen senere —
metoden ville være den samme fase-2-tankegang som iTunes-data: slå op
ÉN gang, gem resultatet, ikke live i browseren.

**2026-08-30 — Startpunktet for et klip kan nu justeres per sang, direkte i
spillet — det er ikke længere altid begyndelsen af Apples preview.**
Nogle af de nye sange (ud over de fem oprindeligt testede) viste sig at
have et preview der starter midt i teksten. Løsningen er selvbetjent:
knapper i spillet (−1s / −0,1s / +0,1s / +1s / Nulstil) lader Emil lytte og
finjustere, og justeringen huskes per sangtitel i `localStorage` — ingen
grund til at beskrive eller citere sangtekst i en besked for at rette det.
Dette ophæver beslutningen ovenfor om ikke at tilføje et manuelt
startpunkt — betingelsen dér ("medmindre en fremtidig sang viser sig at
starte akavet") er nu indtruffet. Fravalgt: at rette startpunkter i data-
filen i stedet (kræver at Claude ved præcis hvor hver sang skal starte,
hvilket i praksis betyder at diskutere sangtekst frem og tilbage).
**Ophævet 2026-08-30, se nedenfor.**

**2026-08-30 — Startpunkt-knapperne blev fjernet fra spillet igen; klippet
starter altid fra begyndelsen af Apples preview, som før.**
Emil påpegede at det ikke giver mening i et spil man deler med andre: en
justering gemt i den enkelte spillers browser (`localStorage`) betyder at
folk reelt ikke oplever den samme sang, og en synlig "juster klippet"-knap
hører ikke hjemme i noget spillere skal sammenligne resultater på.
Ophæver beslutningen ovenfor om startpunkt-knapper i spillet. Problemet
med sange der starter midt i teksten er stadig åbent — se `docs/ideer.md`
for den rigtige løsning (et fast startpunkt i data, samme for alle).
Fravalgt: at beholde knapperne bare for Emil selv (ingen adskillelse
mellem "admin" og "spiller" i dette spil endnu).
**Løst rigtigt 2026-08-30, se pipeline-beslutningen nedenfor —
`start_offset_ms` findes nu automatisk og er ens for alle spillere.**

**2026-08-30 — Resultatet for hver enkelt sang (grid, antal forsøg,
Apple Music/Spotify-links) vises ikke længere mellem sangene — kun det
samlede overblik for alle fem sange vises til sidst.**
Emil ville have det til kun at være ét samlet overblik i stedet for en
mellemskærm efter hver sang. Man går derfor direkte videre til næste sang
efter et gæt/spring over/afsløring, uden en "Næste sang"-knap undervejs;
det samlede overblik (som allerede fandtes) viser stadig alle fem gitre og
links, bare først når alle fem er spillet. Fravalgt: at beholde en kort
mellemskærm ("Rigtigt!"/"Ikke gættet") uden grid og links — Emil bad om
slet ingen mellemresultat.

**2026-08-30 — Fase 2's data-pipeline er bygget: `data/sange.json`
genereres nu automatisk af et script (`scripts/generer-sange-json.js`),
kørt af GitHub Actions hver gang `data/sange.csv` ændres — ikke af et
værktøj Emil selv skal åbne og klikke på.**
Emil spurgte direkte om det ville blive bøvlet i længden med mange sange
og løbende opdateringer. To løsninger blev lagt frem: (a) et lille
browser-værktøj Emil selv åbner og klikker på ved hver opdatering, eller
(b) fuld automatik via GitHub, uden noget manuelt trin nogensinde, men med
mere "maskinrum" (en rigtig CI-robot) at have stående. Emil valgte (b)
bevidst efter at have hørt begge dele. Det bryder med den tidligere
tommelfingerregel "ingen backend, ingen build-trin" — se opdateret
"Tekniske rammer" i CLAUDE.md for den præcise afgrænsning (ingen server
der kører, kun et engangs-script der starter ved en git-push).
Scriptet er indrettet til at være billigt at bruge i det lange løb: det
slår kun NYE sange op mod Apple (sange der allerede findes i
`sange.json` fra en tidligere kørsel, bliver ikke slået op igen) — så
pipelinen bliver ikke langsommere eller dyrere af at puljen vokser til
30, 100 eller flere sange. Titel/kunstner/sværhed/noter synkroniseres
altid fra CSV'en; alt det dyre (lyd, årstal, genre, startpunkt) bevares.
Fravalgt: (a), fordi Emil selv vurderede at et helt automatisk flow var
det værd, når puljen skal vokse og opdateres løbende.

**2026-08-30 — Startpunktet (`start_offset_ms`) findes nu automatisk af
generator-scriptet, ved at analysere lydklippets første sekunder og
springe stilhed i starten over (ffmpeg `silencedetect`) — ikke ved at
nogen lytter og klikker manuelt.**
Emil valgte dette fremfor at sætte alle startpunkter til 0 og tage
spørgsmålet senere. Løser det egentlige, oprindelige spørgsmål fra
fase 1 ("rammer 0,1-sekunds klippet noget genkendeligt?") for ALLE
fremtidige sange, automatisk, uden at nogen — hverken Emil eller Claude —
nogensinde skal lytte til eller omtale sangens tekst for at rette det.
Samme startpunkt gælder for alle spillere (data-drevet, ikke
browser-lokalt), i tråd med beslutningen ovenfor om at fjerne de
spiller-justerbare knapper. Fravalgt: at lade startpunktet stå på 0 og
vente på flere klager.

**2026-08-30 — `dagens-sang.html` slår ikke længere sange op live hos
Apple (JSONP) — den henter i stedet den færdige `data/sange.json` én
gang ved opstart.**
Direkte følge af pipeline-beslutningen ovenfor. Gør spillet hurtigere
(ingen "slår sangen op…"-ventetid per sang), mere robust (afhænger ikke
af at Apples søge-API svarer hver gang nogen åbner spillet), og gør at
`alternative_titler`/`alternative_kunstnere` nu bruges aktivt i både
søgeforslag og gættematch — fx matcher "Gasolin" (uden apostrof) nu
korrekt sangen af "Gasolin'". Kræver at siden åbnes via et rigtigt link
(fx GitHub Pages), ikke ved at dobbeltklikke filen lokalt — `fetch()` af
en lokal fil virker ikke over `file://`. Fravalgt: at beholde JSONP som
reserve — unødvendig kompleksitet nu hvor data findes på forhånd.

---

## Beslutninger der mangler

- [x] Hvad betyder "dansk sang"? — dansk kunstner, uanset sprog
      (2026-08-30), se beslutning ovenfor.
- [x] Projektnavn — arbejdstitel "Ørehænger" (2026-08-30), kan genoverveges
      senere. Domæne stadig ikke valgt.
- [x] Klip-trinnene: 0,1 / 0,5 / 2 / 8 / 15 sekunder — bekræftet fine i
      fase 1-testen (2026-08-30), se beslutning ovenfor.
- [x] Startpunkt per sang — findes nu automatisk af generator-scriptet
      (lydanalyse, springer stilhed i starten over), ens for alle
      spillere (2026-08-30), se pipeline-beslutningerne ovenfor.
- [x] Hvor mange gæt får man? — 5, ét per klip-trin (2026-08-30), se
      beslutning ovenfor.
- [x] Hvordan ser det delte resultat ud? — emoji-grid, nu med én linje per
      sang for dagens 5 sange, vist kun i det samlede overblik til sidst,
      ikke mellem hver sang (2026-08-30), se beslutning ovenfor.
- [x] Biblioteket — venter bevidst på at `sange.json` er beriget med
      årti/genre i fase 2 (2026-08-30). Data findes nu (genre/aarti
      genereres af pipelinen) — Biblioteket kan tages op til revurdering.
- [ ] Projektet ligger endnu ikke i et rigtigt GitHub-repo — kun herinde i
      Claude-projektet. Se `github-opsaetning.md` for hvordan automatikken
      kommer til at køre, når Emil opretter repoet.
