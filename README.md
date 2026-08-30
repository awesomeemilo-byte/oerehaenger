# Ørehænger (arbejdstitel)

Et webspil hvor man gætter danske sange ud fra et lydklip der bliver længere
for hvert forkert gæt.

- **Dagens sang** — én sang om dagen, samme for alle, delbart resultat.
- **Biblioteket** — fri leg, ubegrænsede runder, filtre på årti og genre.

## Status

Fase 1: den grimme prototype virker (`src/prototype.html`) — fem sange,
voksende klip, søgefelt med forslag. Dagens-sang/bibliotek-strukturen er
endnu ikke bygget.

## Sådan er repoet skruet sammen

```
CLAUDE.md          rammerne for projektet — læses automatisk af Claude
START-HER.md       sådan starter du en arbejdssession
/docs
  beslutninger.md  hver beslutning + hvorfor + hvornår
  ideer.md         alt vi IKKE bygger nu
/data
  LÆS-MIG.md       hvordan sanglisten fungerer
  sange.csv        arbejdslisten — den eneste fil du skriver i i hånden
  sange.json       spillets datafil — genereres fra CSV'en
/design
  noter.md         designbeslutninger
/src               koden — fase 1-prototypen ligger her
```

## Lyd og rettigheder

Spillet hoster ikke musik. Lyd afspilles som 30-sekunders previews direkte
fra Apples iTunes Search API, og hver sang linker tilbage til kilden.

Er du rettighedshaver og vil have en sang fjernet: [KONTAKTADRESSE].
