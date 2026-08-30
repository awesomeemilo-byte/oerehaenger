# Om datafilerne

## `sange.csv` — din arbejdsliste

Det er den eneste fil du selv skal skrive i. Åbn den i Excel, Numbers eller
Google Sheets. Fire kolonner:

| Kolonne | Hvad du skriver |
|---|---|
| `titel` | Sangens titel, som den er udgivet |
| `kunstner` | Kunstnernavnet, som det er udgivet |
| `svaerhed` | 1 = alle kender den · 2 = de fleste · 3 = kun hvis du var der |
| `noter` | Til dig selv. Hvorfor er den god? Hvad er fælden? |

Alt andet — udgivelsesår, genre, cover, lyd-URL, startpunkt, alternative
stavemåder — hentes automatisk. Du skal ikke slå noget op.

**De ti rækker der ligger der nu er eksempler.** Slet dem eller behold dem,
som du vil.

### Hvad der gør en sang god til spillet

Den bedste sang er én hvor **alle kan nynne omkvædet, men ingen kan huske
hvad den hedder.** Det er ikke det samme som "mest streamede". Tænk i
situationer i stedet for i hitlister: fest, radio i bilen, gymnastiksal,
sommerhus, fredag aften i 90'erne, konfirmation.

Sigt efter 100 sange. 30 er nok til at komme i gang.

---

## `sange.json` — spillets datafil

**Skriv ikke i den i hånden.** Den genereres automatisk fra `sange.csv` —
se `scripts/generer-sange-json.js`, som kører af sig selv via GitHub
Actions hver gang du ændrer `sange.csv` (opsætning: se
`github-opsaetning.md`). Formatet:

```json
{
  "id": "gasolin-kvinde-min",
  "titel": "Kvinde min",
  "kunstner": "Gasolin'",
  "aar": 1974,
  "aarti": "70er",
  "genre": ["Rock"],
  "svaerhed": 3,
  "alternative_titler": [],
  "alternative_kunstnere": ["Gasolin"],
  "lyd_kilde": "itunes",
  "lyd_id": "0000000000",
  "lyd_url": "https://...",
  "start_offset_ms": 0,
  "cover_url": "https://...",
  "link_ud": "https://...",
  "noter": "Alle kender omkvædet, få kender titlen"
}
```

`alternative_titler` og `alternative_kunstnere` er ikke pynt. De er
forskellen på et spil der føles fair og et der føles ødelagt, fordi nogen
skrev "Gasolin" uden apostrof — spillet matcher nu selv på dem, både i
søgeforslag og når man gætter.

`start_offset_ms` findes automatisk (lydanalyse — springer stilhed i
starten af klippet over). Ingen grund til at rette i den manuelt; hvis en
sang stadig starter akavet, kan du sige det til Claude UDEN at beskrive
eller citere selve teksten — bare navnet på sangen er nok til at
undersøge det videre.

**Sange der ikke kan findes hos Apple** bliver ikke med i `sange.json` —
pipelinen skriver i stedet en advarsel i GitHub Actions-loggen. Tjek
stavningen i `sange.csv` og prøv igen.
