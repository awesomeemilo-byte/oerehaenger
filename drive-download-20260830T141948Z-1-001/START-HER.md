# Sådan starter du en arbejdssession

Du skal **ikke** fodre mig drejebogen. Den er skrevet til dig, ikke til mig —
og en lang tekst i en besked gør mig dårligere, ikke bedre, fordi jeg ikke
kan se hvad der er instruks og hvad der er baggrund.

Det du skal fodre mig er to ting:

1. **Filerne i denne mappe** (læg dem i repoet, eller vedhæft dem i chatten).
2. **En kort opstartsbesked** der siger hvad dagens mål er.

`CLAUDE.md` gør resten. Den bliver læst automatisk hver gang, og den
indeholder alt det du ellers skulle gentage.

---

## Opstartsbesked — kopiér denne

```
Nyt projekt: dansk sang-gættespil. Filerne er vedhæftet / ligger i repoet
— læs CLAUDE.md, docs/beslutninger.md og data/LÆS-MIG.md først.

Jeg er designer, ikke udvikler. Du skriver koden.

Dagens mål: byg fase 1 — den grimme prototype.
Én HTML-fil, fem sange fra data/sange.csv, klip der vokser
(0,1 / 0,5 / 2 / 8 / 15 sekunder), gættefelt, spring over-knap.
Intet design, intet navn, ingen deling. Formålet er ét spørgsmål:
er det sjovt?

Vigtigt at afklare undervejs: rammer 0,1-sekunds klippet noget
genkendeligt, eller starter previewet midt i sangen så trinene
skal laves om?

Læg en plan først. Byg ikke noget før jeg har sagt god for planen.
```

---

## Til senere sessioner

Når projektet er i gang, er opstartsbeskeden kortere:

```
Vi fortsætter på sangspillet. Læs CLAUDE.md og docs/beslutninger.md.
Dagens mål: [én ting].
Læg en plan først.
```

## Til at afslutte en session — kør altid denne

```
Vi stopper her.

Opdatér CLAUDE.md og docs/beslutninger.md med det vi har besluttet i dag.
Skriv derefter en kort status: hvad virker nu, hvad er halvfærdigt, og
hvad er det næste jeg skal tage stilling til.
```

Det er den besked der gør næste session hurtig i stedet for at starte forfra.

---

## Rækkefølgen på det du skal gøre

1. Udfyld `[PROJEKTNAVN]` og `[UDFYLDES]` i `CLAUDE.md`. Fem minutter.
2. Skriv 30 sange i `data/sange.csv`. Se `data/LÆS-MIG.md`.
3. Læg mappen i et GitHub-repo (eller vedhæft filerne i chatten).
4. Send opstartsbeskeden ovenfor.

**Genvej hvis du hellere vil se noget virke end at planlægge:**
Spring 1 og 2 over. Send opstartsbeskeden med det samme — de ti
eksempelsange i CSV-filen er nok til en prototype. Så finder vi ud af
resten bagefter.
