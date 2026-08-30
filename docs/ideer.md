# Ideer vi IKKE bygger nu

Alt godt der dukker op undervejs ryger herned i stedet for ind i koden.
Skriv datoen på. Så mister vi ikke ideen, og vi mister ikke fokus.

---

**2026-08-30** — Årtimodus: kun 80'erne, kun 90'erne osv. (Hører til fase 5.)

**2026-08-30** — Duelmodus: to spillere gætter den samme sang samtidig.

**2026-08-30** — Bandle-mekanikken: instrumenter lagt på ét ad gangen.
Kræver stems/multitrack — teknisk og rettighedsmæssigt langt tungere.
Kun realistisk med selvproducerede spor eller AI-separation af lyd.

**2026-08-30** — Tekstmodus: gæt sangen ud fra én linje tekst.
Ingen lydrettigheder, men tekstrettigheder i stedet.

**2026-08-30** — "Hvem sang den?" som en sværere variant hvor man skal gætte
kunstneren i stedet for titlen.

**2026-08-30** — Rigtig løsning på "nogle klip starter midt i sangen":
et fast startpunkt per sang, gemt i selve datafilen (`sange.csv`/
`sange.json`), samme for alle spillere. Første forsøg var en knap i selve
spillet hvor man selv kunne justere startpunktet — men det gav mening for
Emil som "admin", ikke for et delt spil hvor alle skal opleve den samme
sang (se beslutninger.md, 2026-08-30, hvor det blev fjernet igen). Løst
2026-08-30: generator-scriptet finder nu startpunktet automatisk ved
lydanalyse (se beslutninger.md og CLAUDE.md).
